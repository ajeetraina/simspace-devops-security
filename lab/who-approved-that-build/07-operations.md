# Lab 5 - Operate the Governed Service

<svg viewBox="0 0 900 150" width="100%" role="img" aria-label="The ops loop: the signed service runs in production; an ops agent in an sbx sandbox reads Grafana (logs, metrics, traces) read-only to diagnose a slow request; the fix it proposes goes back through the CI gate before it can ship.">
<g font-family="ui-sans-serif, system-ui, sans-serif">
  <rect x="2" y="10" width="300" height="130" rx="10" fill="#e6f4ea" stroke="#1a7f37" stroke-width="1.3" stroke-dasharray="6 4"/>
  <text x="14" y="30" font-size="10.5" font-weight="800" fill="#14532d">PRODUCTION</text>
  <text x="14" y="46" font-size="9" fill="#14532d">read_only · cap_drop ALL · non-root</text>
  <rect x="18" y="58" width="120" height="44" rx="9" fill="#0b1533" stroke="#1a7f37" stroke-width="2.5"/><text x="78" y="85" text-anchor="middle" font-size="13" font-weight="700" fill="#ffffff">SERVICE</text>
  <text x="30" y="122" font-size="9.5" fill="#9a3412">p99 40ms → 2.3s since 02:47</text>
  <rect x="330" y="10" width="290" height="130" rx="10" fill="#eef4ff" stroke="#2563eb" stroke-width="1.3" stroke-dasharray="6 4"/>
  <text x="342" y="30" font-size="10.5" font-weight="800" fill="#1e3a8a">OPS AGENT · sbx sandbox</text>
  <text x="342" y="46" font-size="9" fill="#3730a3">read-only · one gateway · audited</text>
  <rect x="346" y="58" width="120" height="44" rx="9" fill="#0b1533" stroke="#2563eb" stroke-width="3"/><text x="406" y="79" text-anchor="middle" font-size="12" font-weight="700" fill="#ffffff">AGENT</text><text x="406" y="94" text-anchor="middle" font-size="9" fill="#9aa6c2">diagnose</text>
  <rect x="486" y="58" width="120" height="44" rx="9" fill="#12325e" stroke="#1E9BF0" stroke-width="2"/><text x="546" y="79" text-anchor="middle" font-size="11" font-weight="700" fill="#ffffff">Grafana MCP</text><text x="546" y="94" text-anchor="middle" font-size="8.5" fill="#7fb2e6">logs·metrics·traces</text>
  <rect x="648" y="46" width="70" height="60" rx="10" fill="#fff3e0" stroke="#9a3412" stroke-width="2.5"/><text x="683" y="72" text-anchor="middle" font-size="12" font-weight="800" fill="#9a3412">GATE</text><text x="683" y="90" text-anchor="middle" font-size="8.5" fill="#9a3412">fail closed</text>
  <rect x="740" y="58" width="150" height="44" rx="9" fill="#0b1533" stroke="#1a7f37" stroke-width="2.5"/><text x="815" y="79" text-anchor="middle" font-size="12" font-weight="700" fill="#ffffff">SHIP THE FIX</text><text x="815" y="94" text-anchor="middle" font-size="8.5" fill="#9aa6c2">via the same road</text>
  <g stroke="#9aa6c2" stroke-width="1.6" fill="#9aa6c2">
    <line x1="138" y1="80" x2="344" y2="80"/><polygon points="344,77 349,80 344,83"/>
    <line x1="466" y1="80" x2="484" y2="80"/><polygon points="484,77 489,80 484,83"/>
    <line x1="606" y1="80" x2="646" y2="80"/><polygon points="646,77 651,80 646,83"/>
    <line x1="718" y1="76" x2="738" y2="76"/><polygon points="738,73 743,76 738,79"/>
  </g>
</g></svg>

**10 minutes · hands-on**

> **Where you are on the road:** past the gate, in **production**. The signed, attested
> image from Move 3 is deployed and running. It is 3am and it is **slow**. The build is over;
> the *operating* has begun - and the same discipline applies. The agent that once reached for
> a base image now reaches for your logs, metrics and traces. Box it exactly the same way.

Labs 1–4 governed the **build**: prove what is in it, start it from a hardened base, gate it,
and sandbox the agent that authors it. But a build that passed the gate still has to *run* -
and at some point it misbehaves. In Lab 4 the agent got the **DHI MCP** to choose a base
before it wrote `FROM`. Its ops-time counterpart is an **observability MCP**: an agent that
can read the running system - logs, metrics, traces - to tell you *why* it is slow.

We use the official [Grafana MCP server](https://github.com/grafana/mcp-grafana)
(`mcp-grafana`), shipped as a Docker Sandboxes **kit**:
[`ajeetraina/sbx-kits-grafana`](https://github.com/ajeetraina/sbx-kits-grafana). Same
boundary as the build agent - a microVM, one gateway, **read-only** tools, every call
audited. It can *diagnose*. It cannot *act*. To ship a fix, it takes the same road as any
change.

---

## The symptom

The catalog service has been fine for weeks. Since the 02:47 deploy, p99 latency on
`/api/products/:id` has climbed from ~40ms to ~2.3s. Nothing is *down*; it is just slow, and
the logs are a wall of noise. Wading through them by hand is exactly the toil we want the
agent to take.

First, bring up a Grafana for the agent to read. The kit ships a bundled stack - Grafana plus
Prometheus - so there is nothing to sign up for:

```bash terminal-id=main
docker compose -f compose/docker-compose.yml up -d
```

That is Grafana on `http://localhost:3000` (anonymous admin), with a Prometheus datasource
already wired. The sandbox will reach it over `host.docker.internal:3000` - **no account, no
token**.

---

## Wire in the ops kit

The build agent got the DHI MCP. The ops agent gets the **Grafana kit**. Because the kit is
published to a registry, you run it straight from there - `--kit` pulls it, and it registers
`mcp-grafana` with the agent for you. Hand it the incident as a prompt, and tell it to look
but not touch:

```bash terminal-id=main
sbx run --kit docker.io/ajeetraina777/sbx-grafana-kits:latest claude -p "catalog-service p99 latency jumped after the 02:47 deploy. Use Grafana - metrics, logs, and traces - to find the cause. Do not change anything."
```

Read the transcript top to bottom. Before it guesses, the agent searches the dashboard, pulls
the p99 histogram from Prometheus, greps the Loki logs for slow queries, and checks the
sequential-scan rate - then lands the root cause: the 02:47 dependency bump **dropped the
index behind product lookups**, so every `sku` lookup became a full table scan under load. It
writes the proposed fix to `db/migrations/` and stops. It changed nothing in production,
because it *cannot*.

> [!NOTE]
> `--kit` takes a registry tag, a `git+https://…` URL, or a local path, and pins by digest -
> the same reproducibility you gave the build in Lab 4. The trailing `claude` is just the
> agent; swap in `codex`, `gemini`, or any supported agent and the tooling is identical.

Confirm the kit really registered the server with the agent:

```bash terminal-id=build
claude mcp list
```

And prove the sandbox can actually reach Grafana end to end - the kit ships a runbook that
reads back instance health, datasources and dashboards through `grafana-client`:

```bash terminal-id=build
python3 ~/runbooks/grafana_report.py
```

---

## Govern the tools: read-only

Diagnosing is all **queries**. An ops agent must never silence an alert, edit a dashboard, or
open an incident on your behalf - so scope its tools the same way you scoped the DHI tools in
Lab 4, with a **Cedar** policy that permits only the read-only `mcp-grafana` queries:

```cedar save-as=grafana-readonly.cedar
permit (principal, action == MCP::Action::"register", resource);
permit (principal, action == MCP::Action::"invokePrimordial", resource);

permit (principal, action == MCP::Action::"invokeTool", resource)
when {
  resource.server == "grafana" &&
  ["search_dashboards","get_dashboard_by_uid","list_datasources",
   "query_prometheus","query_loki_logs","list_prometheus_metric_names",
   "list_loki_label_names","list_alert_rules"].contains(resource.tool)
};
```

Everything else - `create_incident`, `add_activity_to_incident`, dashboard writes - is denied
by **default-deny**: it is simply not on the list.

---

## Audit: the connection, not the payload

The sandbox logs every decision. Look at what the ops agent reached for, and what the policy
stopped:

```bash terminal-id=build
sbx audit log --since 02:00
```

You will see the Grafana queries it ran (allowed), the `create_incident` it tried (**denied**
by the read-only policy), and a paste site it reached for (**denied** by the network
allowlist).

> [!IMPORTANT]
> The audit records the **connection, not the payload**. It proves `query_loki_logs` was
> *called and allowed* - it does **not** capture the log lines that came back. This is a
> governance and forensics trail - *who reached what, allowed or denied* - **not** DLP or
> content inspection. If you need request and response bodies, that is app-level
> observability, a different layer. Do not oversell the audit log as something it is not.

---

## The fix goes back through the gate

The agent found the cause and proposed a migration. It cannot deploy it. To ship, the fix
takes the **same road as any change** - and hits the Move 3 gate on the way:

```bash terminal-id=build
git add db/migrations/002_products_sku_index.sql
```

```bash terminal-id=build
git commit -m "fix: restore index on products.sku (p99 regression from 02:47)"
```

```bash terminal-id=build
git push origin main
```

Watch the **CI Pipeline** tab. The migration rides through the exact `secure-build` gate you
wired in Move 3 - scanned, policy-checked, attested - before anything reaches production. A
human authored it or an agent proposed it; the bar is the same.

---

## One file, the whole ops kit

You ran the kit from its published tag. Like the build sandbox in Lab 4, the whole thing is a
declarative artifact - the [`sbx-kits-grafana`](https://github.com/ajeetraina/sbx-kits-grafana)
`spec.yaml` is a `kind: mixin` that installs `mcp-grafana`, sets `GRAFANA_URL`, and pins the
allowed domains. That is what makes it reproducible across a team:

| Target | Runs where | Credential |
|---|---|---|
| **local** (default) | Host `host.docker.internal:3000` | none - anonymous |
| **cloud** | Grafana Cloud `*.grafana.net` | `sbx secret set -g grafana` (proxy-injected) |
| **oss** | Any self-hosted Grafana | `sbx secret set -g grafana` |

The token is **never baked into the kit** - the sbx proxy injects it into outbound requests at
runtime, so raw credentials never enter the sandbox. Same rule as every secret in this
workshop.

---

## Checkpoint

- [ ] Grafana is up on `:3000`; the sandbox reaches it at `host.docker.internal:3000`
- [ ] The ops agent launched with the Grafana kit and registered `mcp-grafana`
- [ ] It found the root cause from metrics, logs and traces - not by guessing
- [ ] The Cedar policy permits only read-only queries; `create_incident` is denied
- [ ] `sbx audit log` shows connections (allowed/denied) - and you know it is **not** payloads
- [ ] The proposed fix cleared the same CI gate as any other change

## What you should be thinking

The build agent and the ops agent are the **same shape**: boxed in a microVM, wired to one
gateway, scoped to read-only tools, audited on every call. One prevents a bad image *before*
`FROM`; the other diagnoses a live incident *without touching prod*. Neither can act on its
own - action always routes back through the gate.

That is the whole workshop in one sentence, now true at both ends of the day: **the agent that
builds runs in a box, and the agent that operates runs in a box too.**

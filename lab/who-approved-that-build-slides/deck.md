<!--
layout: title
chrome: false
eyebrow: "Docker · Agentic DevOps"
byline: "Ajeet Singh Raina — Developer Advocate, Docker"
-->

# Who Approved That Build?

## Docker AI Governance for the Agentic DevOps Pipeline

Note: The title is a real question. By the end you'll have a way to answer it for your own pipeline. We spent a decade making delivery trustworthy — versioned pipelines, peer review, auditable deploys. Then we gave a set of keys to AI agents. Let's start with something that actually happened.

---

<!--
layout: section
eyebrow: "Meet your host"
-->

# Ajeet Singh Raina

Developer Advocate at **Docker** · former Docker Captain

Co-author of **_Operational AI with Docker_** (Packt) · runs the 17,000-member Docker Bengaluru meetup

Note: Quick hello before we dive in — I'm Ajeet Singh Raina, Developer Advocate at Docker. Twenty-plus years across system integration testing, consulting and developer relations; former Docker Captain, and I run the 17,000-member Docker Bengaluru meetup. I co-authored Operational AI with Docker with Harsh Manvar — on deploying, scaling and operating agentic AI services with Docker and Kubernetes, which is exactly the world this talk lives in. Treat this as hands-on and interrupt me with questions. Now, the cold open.

---

<!-- layout: section -->

# What we'll walk through

1. **02:47 AM** — an agent shipped a build nobody approved
2. **Evidence** — SBOM · VEX · SLSA: what's in it, where it's from
3. **Docker Hardened Images (DHI)** — a governed base, by default
4. **CI pipeline** — sign + policy gate that fails closed
5. **Sandboxing** — box the agent's own build environment
6. **DHI MCP** — the tools the agent calls, hardened too

Built for developers **and** the platform / SRE / ops teams who answer for what ships.

Note: Here's the road. We open with the 2:47 AM incident — why this is a problem now. Then the evidence layer: SBOM, VEX, SLSA. Then Docker Hardened Images as a governed baseline. Then the CI pipeline that turns policy into a gate. Then we push the fix left — sandboxing the agent's build environment, and hardening the MCP tools the agent calls at runtime. And a note on audience: the first half feels like a developer talk, but the gate, the baseline and the audit trail are owned by platform, SRE and ops — I'll call out who owns what as we go.

---

# 02:47 AM

```console
$ git log -1 --stat
commit a9d0e42  (HEAD -> main, origin/main)
Author: svc-build-agent <ci@dockerlabs.xyz>
Date:   Tue 02:47:11

    chore: bump base image, regenerate Dockerfile

 Dockerfile        |  34 +++++++------
 package-lock.json | 812 +++++++++++++++++++++++++++++++
```

The author isn't a person. **`svc-build-agent`.** It changed *how the app is built* — and 812 lines of dependencies.

Note: 2:47 in the morning, a commit lands on main. The author is a service account — an agent. It bumped the base image, rewrote the Dockerfile, and pulled in 812 lines of new lockfile. This is the highest-leverage change you can make to an artifact: what it's built on, and what's inside it. Nobody was awake.

---

# Who reviewed it?

```console
$ gh pr checks 4127
build      pass    2m14s
scan       pass    0m48s
publish    pass    1m03s
✓ all checks passed → merged to main

$ gh pr view 4127 --json reviews -q '.reviews'
[]
```

CI said yes. **The reviewers array is empty.**

Note: Who reviewed it? CI did — build, scan, publish, all green, merged. Human reviewers? Empty array. And nothing was broken; every check we had passed. The problem isn't a failing check. It's that the checks we had were never designed to answer what a reviewer would ask: what's in this, and where did it come from.

---

<!-- layout: section -->

# The review step is gone

A human used to stand at three decisions. Now an agent does — at machine speed, on every repo.

| Then | Now |
|---|---|
| A human **picks** the base image | An agent picks it, autonomously |
| Dependencies **reviewed** in a PR | Packages resolved, no review |
| CI runs config **a human wrote** | The agent wrote the Dockerfile |

Note: The supply chain didn't change. The review step did. Every row where trust used to pass through a person now passes through an agent. And this scales with capability — the better the agent, the more it does unsupervised. The job isn't to stop agents. It's to govern them without killing the speed that made them worth adopting.

---

<!-- layout: section -->

# So: govern the build.

Prove what's in it → start it from a good base → gate it in CI.
No new process. Same pipeline the change already runs through.

Note: Three moves, all inside the pipeline the change already goes through. One: prove what's in the artifact and where it came from. Two: give the agent a base image that's already good. Three: gate it in CI so nothing unverified passes. Let's do each with a terminal open.

---

# Move 1 — what's actually in it?

```console
$ npm ls --all | wc -l
431

$ docker scout quickview catalog-service:baseline
    ✓ Indexed 431 packages
  Target      catalog-service:baseline   0C   8H   41M   93L
  Base image  node:20 → updatable        0C   6H   30M   54L
```

431 packages the agent pulled in blind. **8 high, 41 medium** — before any of your code.

Note: Move one — measure it. The agent's image carries 431 packages. Scout indexes them: 8 highs, 41 mediums, 93 lows, and that's the base alone, before your application code. You can't govern what you can't see. This is the evidence layer, and everything else is built on it.

---

# Four questions, four attestations

| Question | Answer |
|---|---|
| What's in it? | **SBOM** |
| Where did it come from? | **SLSA provenance** |
| Can you verify that? | **Signature** (keyless / cosign) |
| Which CVEs actually affect me? | **VEX** |

```console
$ docker scout attest get --predicate-type openvex ...
  "vulnerability": "CVE-2023-45853",
  "status": "not_affected",
  "justification": "vulnerable_code_not_in_execute_path"
```

Note: Four questions, four machine-readable answers. SBOM is the ingredient list. SLSA provenance is a signed record of how and where it was built. A signature proves those belong to this exact digest. And VEX — the one people skip — lets you say "that CVE is present but not exploitable here," so your gate triages instead of drowning in noise. Evidence, and the ability to act on it.

---

<!-- layout: section -->

# Move 2 — give the agent a good base

The agent picks a base image on every build. Fix the default once, fix every build.

Note: Move one told us how bad the baseline is. Move two fixes the baseline itself. Here's the leverage: an agent makes the base-image decision constantly, on every repo, at machine speed. Fix what it reaches for by default and you've fixed every future build at once — no review queue.

---

# Same app, hardened base

```console
$ docker scout quickview catalog-service:dhi
    ✓ Indexed 78 packages
  Target      catalog-service:dhi                   0C   0H   1M   4L
  Base image  dhi.io/node:24-debian13 (distroless)  0C   0H   0M   0L

$ docker images catalog-service
REPOSITORY        TAG        SIZE
catalog-service   baseline   1.1GB
catalog-service   dhi        248MB
```

431 → **78 packages**. 1.1GB → **248MB**. 8 highs → **0**. Distroless, non-root, SBOM built in.

Note: Same application, Docker Hardened Image base. Packages drop from 431 to 78. Image from 1.1 gig to 248 meg. Eight highs to zero. It ships distroless — no shell in the final image — non-root, with its own SBOM and provenance attached. The attack surface you never shipped is the CVE you never triage. This is the single highest-leverage change in the pipeline.

---

<!-- layout: section -->

# Move 3 — make it a gate, not a suggestion

Evidence and a good base do nothing if nothing enforces them. Turn CI into the control point.

Note: Move three, and this is where governance stops being a wiki page. A good baseline you don't enforce erodes on the next merge. So we encode the rules as a policy and run it in CI — the one thing every change already passes through.

---

# The policy, run on both images

```console
$ docker scout policy catalog-service:baseline
  ✗ no-critical-cves     FAILED   8 high, review required
  ✗ require-sbom         FAILED   no SBOM attestation present
  ✗ require-provenance   FAILED   no SLSA provenance
  Policy status: FAILED (0 of 3 met)

$ docker scout policy catalog-service:dhi
  ✓ no-critical-cves     PASSED
  ✓ require-sbom         PASSED
  ✓ require-provenance   PASSED
  Policy status: PASSED (3 of 3 met)
```

Note: Same policy, both images. The 2:47 AM artifact fails all three — no clean scan, no SBOM, no provenance. The hardened, attested build passes all three. This is the bar. It's the same bar whether a human or an agent authored the change.

---

# Verify the claim, don't trust it

```console
$ docker scout attest get \
    --predicate-type https://slsa.dev/provenance/v0.2 --verify \
    catalog-service:dhi
    ✓ Signature verified (keyless, Fulcio root)
  "builder":  "https://docker.com/dhi/builder",
  "configSource":
    "git+github.com/docker-hardened-images/node@refs/tags/24-debian13"
```

Signed, keyless, and **traceable to a source commit you can open and read.**

Note: Provenance only matters if it's verifiable. This pulls the SLSA attestation and checks the signature — keyless, rooted in Fulcio, no long-lived keys to leak. And it traces back to a specific source commit you can actually open. That's the difference between "trust me" and "here's the receipt."

---

# The gate fires on the agent's push

```console
CI · secure-build #218                                  ✗ failed
  ✓ checkout
  ✗ verify-attestations
      ✗ No attestations found on this digest
      ✗ tag rebuilt without --sbom / --provenance
      Error: process completed with exit code 1
  ⊘ policy-gate     skipped
  ⊘ push            skipped
```

The agent rebuilt without attestations. **Blocked before it reached the registry.**

Note: Now replay 2:47 AM with the gate in place. The agent pushes, the pipeline runs the same verification a human change triggers. The rebuilt tag has nothing signed on it, so verify-attestations fails, and everything after — policy, push — is skipped. It never reaches the registry. The agent gets held to the human standard, automatically, with no human in the loop. That's the whole point.

---

<!-- layout: split -->

# One gate, two audiences

<!-- region -->

### Developers
- Fast pass / fail right in the PR
- The base is already good — nothing to remember
- Fix at authoring time, not after a rejection

<!-- region -->

### Platform · SRE · Ops
- **Own** the policy and hardened base — fleet-wide
- Signed attestations **are** the audit trail
- 2:47 AM incident → a query, not a forensics project

Note: This is the slide for the ops folks in the room. The same gate serves two audiences. Developers get fast pass/fail in the PR and a base image that's already good, so they fix things at authoring time instead of after a rejection. Platform, SRE and ops own the other side: they set the policy and the hardened base once and it applies fleet-wide, and the signed attestations become the audit trail. When the next 2:47 AM commit happens, answering "what shipped and where did it come from" is a query against attestations, not a week of forensics. Governance is a platform capability, not a developer chore.

---

<!-- layout: section -->

# Move 4 — push the fix left

Stop cleaning up after the agent. Govern where it *works* — and what it can *reach*.

Note: Everything so far catches the bad artifact at the gate. Good — but that's still cleaning up after the agent ships. Move four pushes the fix left, into two places: the environment the agent builds in, and the tools it can call.

---

# Box the agent's build environment

```console
$ sbx daemon start
  microVM booted: own daemon, own network, host mounted read-only.

$ sbx run codex -p "containerize this service for production"
  → wrote Dockerfile
    FROM dhi.io/node:24-debian13 (distroless)  ·  non-root  ·  multi-stage
```

The agent authors inside a **microVM it can't escape**, and reaches for the hardened base on its own. The 2:47 AM build never happens.

Note: First, the sandbox. The agent runs inside a microVM — its own daemon, its own network, the host mounted read-only. It can pull, build and experiment, but it can't touch your machine or your network. And because the sandbox is wired to the hardened base and the DHI MCP, the agent reaches for the good base image on its own. Notice the Dockerfile it wrote: distroless DHI, non-root, multi-stage — with no correction from you. This is the goal: the red 2:47 AM build never happens, because the environment made the good path the default path.

---

# Harden the tools the agent calls

```console
$ docker mcp catalog pull docker/mcp-catalog-dhi
  ✓ pulled 11 hardened MCP servers
  ✓ signatures verified (Docker, Inc.)

$ docker mcp gateway run --catalog docker/mcp-catalog-dhi
  filesystem (Hardened) → verifying provenance... ✓ signed, SBOM present
  read_only rootfs · cap_drop ALL · secrets mounted into target only
  Gateway ready. An unsigned or tampered server would be refused here.
```

An agent is only as governed as the tools it can reach. **DHI MCP** servers are signed, SBOM'd and sandboxed at runtime.

Note: Second, the tools. An agent calls MCP servers — filesystem, git, github, databases — and each one is code running with access to your stuff. If those are unsigned images off the internet, you've governed the build and left the runtime wide open. DHI MCP is a catalog of hardened MCP servers: built on a DHI base, signed, with SBOMs, and the gateway verifies the signature before it runs one, then sandboxes it — read-only rootfs, all capabilities dropped, secrets scoped to the target. Same three moves — prove, harden, enforce — applied to the agent's tools, not just its artifacts.

---

<!-- layout: default -->

# Or watch it live

```bash terminal-id=demo
docker scout policy catalog-service:baseline
```

::terminal{id=demo height=300}

Note: This is the simulator from the hands-on lab — the exact commands you just saw. Run the policy on the 2:47 AM image and watch it fail three of three. In the lab you take that same image and walk it all the way to green: SBOM, hardened base, signature, and the CI gate. Let me point you there.

---

<!-- layout: stats -->

# One frame

:::stat{value="Prove"}
SBOM · VEX · SLSA — evidence on every agent change
:::

:::stat{value="Default"}
Hardened base — good by default, at machine scale
:::

:::stat{value="Enforce"}
Sign + policy in CI — fails closed, one bar for all
:::

Note: Three moves, one model. Prove, default, enforce. None of them slow the agent down — they run in the pipeline the change already goes through. That's the goal: keep the agents fast, keep yourself in control. Those aren't in tension.

---

# Takeaways

1. A governance model that **fits your existing CI/CD** — not a parallel process.
2. **SBOM / VEX / SLSA** as the audit trail for agent-driven changes.
3. **Sign + policy gate**: agents clear the same bar as humans, automatically.

Note: Three things to leave with. A model that fits the pipeline you already run. Supply-chain evidence as the audit trail. And a policy gate that applies one standard whether a human or an agent authored the change.

---

<!--
layout: title
byline: "agentic.dockerworkshop.com"
-->

# Do the lab

Take the 2:47 AM image and drive it to a signed, attested, policy-gated build. In your browser, nothing to install.

```console
$ open agentic.dockerworkshop.com
```

Note: Right next to this deck is a hands-on lab that runs entirely in your browser — no install. You take the exact image the agent shipped and walk it through every move here: SBOM, hardened base, signature, CI gate. Bookmark it.

---

<!--
layout: section
chrome: false
-->

# Who approved that build?

Now you can answer — and prove it.

**Thank you. Questions?**

Note: Who approved that build? With evidence, a governed base, and an enforceable gate, the answer is: the same policy that approves every build, applied to the agent exactly as to a human, with a trail to prove it. Thank you — questions?

<!--
layout: title
chrome: false
eyebrow: "Docker · Agentic DevOps"
byline: "Ajeet Singh Raina - Developer Advocate, Docker"
-->

# Who Approved That Build?

## Docker AI Governance for the Agentic DevOps Pipeline

Note: The question in the title is a real one - who approved that build? By the end you'll have a way to answer it for your own pipeline. We spent a decade making delivery trustworthy - versioned pipelines, peer review, auditable deploys. Then we handed a set of keys to AI agents. Let's get into it.

---

<!--
layout: section
eyebrow: "Meet the speaker"
-->

# Ajeet Singh Raina

Developer Advocate at **Docker** · former Docker Captain

Co-author of **_Operational AI with Docker_** (Packt) · runs the 17,000-member Docker Bengaluru meetup

Note: Quick hello - I'm Ajeet Singh Raina, Developer Advocate at Docker. Twenty-plus years across system integration testing, consulting and developer relations; former Docker Captain, and I run the 17,000-member Docker Bengaluru meetup. I co-authored Operational AI with Docker with Harsh Manvar - on deploying, scaling and operating agentic AI services with Docker and Kubernetes, which is exactly the world this talk lives in. Treat this as hands-on and interrupt me with questions.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="Agenda" style="position:absolute;inset:0;width:100%;height:100%;object-fit:contain;background:#ffffff" font-family="Arial, Helvetica, sans-serif">
  <rect x="0" y="0" width="1600" height="900" fill="#ffffff"/>
  <text x="130" y="152" font-size="82" font-weight="800" fill="#0B1533">Agenda</text>
  <rect x="134" y="180" width="156" height="9" rx="4" fill="#1E9BF0"/>
  <text x="134" y="246" font-size="27" fill="#5B6B8C">From an unapproved build to a governed one.</text>
  <text x="134" y="828" font-size="24" fill="#5B6B8C">Built for developers <tspan font-weight="800" fill="#0B1533">and</tspan> platform / SRE / ops.</text>
  <g>
    <g transform="translate(470,300)"><rect width="1010" height="82" rx="41" fill="#1E9BF0"/><circle cx="46" cy="41" r="33" fill="#ffffff"/><text x="46" y="41" text-anchor="middle" dominant-baseline="central" font-size="25" font-weight="800" fill="#1E9BF0">01</text><text x="102" y="41" dominant-baseline="central" font-size="30" font-weight="600" fill="#ffffff">The 02:47 AM incident</text></g>
    <g transform="translate(470,392)"><rect width="1010" height="82" rx="41" fill="#1A82DA"/><circle cx="46" cy="41" r="33" fill="#ffffff"/><text x="46" y="41" text-anchor="middle" dominant-baseline="central" font-size="25" font-weight="800" fill="#1A82DA">02</text><text x="102" y="41" dominant-baseline="central" font-size="30" font-weight="600" fill="#ffffff">Evidence: SBOM · VEX · SLSA</text></g>
    <g transform="translate(470,484)"><rect width="1010" height="82" rx="41" fill="#1668C0"/><circle cx="46" cy="41" r="33" fill="#ffffff"/><text x="46" y="41" text-anchor="middle" dominant-baseline="central" font-size="25" font-weight="800" fill="#1668C0">03</text><text x="102" y="41" dominant-baseline="central" font-size="30" font-weight="600" fill="#ffffff">Docker Hardened Images (DHI)</text></g>
    <g transform="translate(470,576)"><rect width="1010" height="82" rx="41" fill="#114EA2"/><circle cx="46" cy="41" r="33" fill="#ffffff"/><text x="46" y="41" text-anchor="middle" dominant-baseline="central" font-size="25" font-weight="800" fill="#114EA2">04</text><text x="102" y="41" dominant-baseline="central" font-size="30" font-weight="600" fill="#ffffff">CI pipeline: sign + policy gate</text></g>
    <g transform="translate(470,668)"><rect width="1010" height="82" rx="41" fill="#0D3B7E"/><circle cx="46" cy="41" r="33" fill="#ffffff"/><text x="46" y="41" text-anchor="middle" dominant-baseline="central" font-size="25" font-weight="800" fill="#0D3B7E">05</text><text x="102" y="41" dominant-baseline="central" font-size="30" font-weight="600" fill="#ffffff">Sandbox the agent's build</text></g>
    <g transform="translate(470,760)"><rect width="1010" height="82" rx="41" fill="#0B1533"/><circle cx="46" cy="41" r="33" fill="#ffffff"/><text x="46" y="41" text-anchor="middle" dominant-baseline="central" font-size="25" font-weight="800" fill="#0B1533">06</text><text x="102" y="41" dominant-baseline="central" font-size="30" font-weight="600" fill="#ffffff">DHI MCP: hardened tools</text></g>
  </g>
</svg>

Note: Here's the road. We open with the evidence layer - SBOM, VEX, SLSA. Then Docker Hardened Images as a governed baseline. Then the CI pipeline that turns policy into a gate. Then we push the fix left - sandboxing the agent's build environment, and hardening the MCP tools it calls. And a note on audience: the first half feels like a developer talk, but the gate, the baseline and the audit trail are owned by platform, SRE and ops - I'll call out who owns what as we go. But first, the thing that makes all of this urgent - something that actually happened.

---

# 02:47 AM

## A commit landed.

```console
$ git log -1 --stat
commit a9d0e42  (HEAD -> main, origin/main)
Author: svc-build-agent <ci@dockerlabs.xyz>
Date:   Tue 02:47:11

    chore: bump base image, regenerate Dockerfile

 Dockerfile        |  34 +++++++------
 package-lock.json | 812 +++++++++++++++++++++++++++++++
```

The author isn't a person. **`svc-build-agent`.** It changed *how the app is built* - and 812 lines of dependencies.

Note: 2:47 in the morning, a commit lands on main. The author is a service account - an agent. It bumped the base image, rewrote the Dockerfile, and pulled in 812 lines of new lockfile. This is the highest-leverage change you can make to an artifact: what it's built on, and what's inside it. Nobody was awake.

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

Note: Who reviewed it? CI did - build, scan, publish, all green, merged. Human reviewers? Empty array. And nothing was broken; every check we had passed. The problem isn't a failing check. It's that the checks we had were never designed to answer what a reviewer would ask: what's in this, and where did it come from.

---

<!-- layout: section -->

# The review step is gone

A human used to stand at three decisions. Now an agent does - at machine speed, on every repo.

| Then | Now |
|---|---|
| A human **picks** the base image | An agent picks it, autonomously |
| Dependencies **reviewed** in a PR | Packages resolved, no review |
| CI runs config **a human wrote** | The agent wrote the Dockerfile |

Note: The supply chain didn't change. The review step did. Every row where trust used to pass through a person now passes through an agent. And this scales with capability - the better the agent, the more it does unsupervised. The job isn't to stop agents. It's to govern them without killing the speed that made them worth adopting.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="You can't inherit trust, you manufacture it" style="position:absolute;inset:0;width:100%;height:100%;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="100" y="120" font-size="56" font-weight="800" fill="#ffffff">You can't inherit trust. <tspan fill="#1E9BF0">Manufacture it.</tspan></text>
  <rect x="100" y="210" width="600" height="290" rx="18" fill="#12203F" stroke="#26365C" stroke-width="2"/>
  <text x="132" y="262" font-size="22" font-weight="800" fill="#9AA6C2" letter-spacing="3">INHERITED TRUST</text>
  <text x="132" y="322" font-size="32" font-weight="700" fill="#ffffff">Code review · CI · ownership</text>
  <text x="132" y="364" font-size="24" fill="#9AA6C2">A reviewer. A pipeline. A named owner.</text>
  <rect x="132" y="410" width="288" height="60" rx="30" fill="#0B1533" stroke="#5A4A2A" stroke-width="2"/>
  <circle cx="166" cy="440" r="8" fill="#F0A84A"/>
  <text x="188" y="440" font-size="25" font-weight="700" fill="#F0A84A" dominant-baseline="central">runs at HUMAN speed</text>
  <rect x="900" y="210" width="600" height="290" rx="18" fill="#12203F" stroke="#26365C" stroke-width="2"/>
  <text x="932" y="262" font-size="22" font-weight="800" fill="#9AA6C2" letter-spacing="3">AGENTS</text>
  <text x="932" y="322" font-size="32" font-weight="700" fill="#ffffff">Pull deps · write Dockerfiles</text>
  <text x="932" y="362" font-size="32" font-weight="700" fill="#ffffff">edit infra · trigger builds</text>
  <rect x="932" y="410" width="300" height="60" rx="30" fill="#0B1533" stroke="#5A2A2A" stroke-width="2"/>
  <circle cx="966" cy="440" r="8" fill="#F0533F"/>
  <text x="988" y="440" font-size="25" font-weight="700" fill="#F0533F" dominant-baseline="central">run at MACHINE speed</text>
  <circle cx="800" cy="355" r="46" fill="#1E9BF0"/>
  <text x="800" y="355" text-anchor="middle" dominant-baseline="central" font-size="30" font-weight="800" fill="#0B1533">vs</text>
  <rect x="100" y="558" width="1400" height="152" rx="18" fill="#0E1B3A" stroke="#1E9BF0" stroke-width="2"/>
  <text x="132" y="602" font-size="22" font-weight="800" fill="#1E9BF0" letter-spacing="2">MANUFACTURE IT</text>
  <g font-size="25" font-weight="700" fill="#ffffff">
    <rect x="132" y="622" width="316" height="62" rx="12" fill="#12325E"/><text x="290" y="653" text-anchor="middle" dominant-baseline="central">Evidence</text>
    <rect x="472" y="622" width="316" height="62" rx="12" fill="#12325E"/><text x="630" y="653" text-anchor="middle" dominant-baseline="central">Hardened base</text>
    <rect x="812" y="622" width="316" height="62" rx="12" fill="#12325E"/><text x="970" y="653" text-anchor="middle" dominant-baseline="central">Sign + gate</text>
    <rect x="1152" y="622" width="316" height="62" rx="12" fill="#12325E"/><text x="1310" y="653" text-anchor="middle" dominant-baseline="central">Runtime boundary</text>
  </g>
  <text x="100" y="788" font-size="30" font-weight="700" fill="#ffffff">Don't trust the agent. <tspan fill="#34D399" font-weight="800">Trust the system</tspan> - enough to close your laptop while the work runs.</text>
</svg>

Note: This is the hinge of the talk. For decades we inherited trust from people and process - a reviewer, a CI job, a named owner - and every bit of it ran at human speed. Agents run faster than any of it, so you can't inherit that trust anymore. You manufacture it. And notice the goal was never to trust the agent - it's to trust the system around it enough that you can close your laptop and the work keeps going. Everything after this slide is how you manufacture that trust.

---

<!-- layout: section -->

# So: govern the build.

Prove what's in it → start it from a good base → gate it in CI.
No new process. Same pipeline the change already runs through.

Note: Three moves, all inside the pipeline the change already goes through. One: prove what's in the artifact and where it came from. Two: give the agent a base image that's already good. Three: gate it in CI so nothing unverified passes. Let's do each with a terminal open.

---

# Move 1 - what's actually in it?

```console
$ npm ls --all | wc -l
431

$ docker scout quickview catalog-service:baseline
    ✓ Indexed 431 packages
  Target      catalog-service:baseline   0C   8H   41M   93L
  Base image  node:20 → updatable        0C   6H   30M   54L
```

431 packages the agent pulled in blind. **8 high, 41 medium** - before any of your code.

Note: Move one - measure it. The agent's image carries 431 packages. Scout indexes them: 8 highs, 41 mediums, 93 lows, and that's the base alone, before your application code. You can't govern what you can't see. This is the evidence layer, and everything else is built on it.

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

Note: Four questions, four machine-readable answers. SBOM is the ingredient list. SLSA provenance is a signed record of how and where it was built. A signature proves those belong to this exact digest. And VEX - the one people skip - lets you say "that CVE is present but not exploitable here," so your gate triages instead of drowning in noise. Evidence, and the ability to act on it.

---

<!-- layout: section -->

# Move 2 - give the agent a good base

The agent picks a base image on every build. Fix the default once, fix every build.

Note: Move one told us how bad the baseline is. Move two fixes the baseline itself. Here's the leverage: an agent makes the base-image decision constantly, on every repo, at machine speed. Fix what it reaches for by default and you've fixed every future build at once - no review queue.

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

Note: Same application, Docker Hardened Image base. Packages drop from 431 to 78. Image from 1.1 gig to 248 meg. Eight highs to zero. It ships distroless - no shell in the final image - non-root, with its own SBOM and provenance attached. The attack surface you never shipped is the CVE you never triage. This is the single highest-leverage change in the pipeline.

---

<!-- layout: section -->

# Move 3 - make it a gate, not a suggestion

Evidence and a good base do nothing if nothing enforces them. Turn CI into the control point.

Note: Move three, and this is where governance stops being a wiki page. A good baseline you don't enforce erodes on the next merge. So we encode the rules as a policy and run it in CI - the one thing every change already passes through.

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

Note: Same policy, both images. The 2:47 AM artifact fails all three - no clean scan, no SBOM, no provenance. The hardened, attested build passes all three. This is the bar. It's the same bar whether a human or an agent authored the change.

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

Note: Provenance only matters if it's verifiable. This pulls the SLSA attestation and checks the signature - keyless, rooted in Fulcio, no long-lived keys to leak. And it traces back to a specific source commit you can actually open. That's the difference between "trust me" and "here's the receipt."

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

Note: Now replay 2:47 AM with the gate in place. The agent pushes, the pipeline runs the same verification a human change triggers. The rebuilt tag has nothing signed on it, so verify-attestations fails, and everything after - policy, push - is skipped. It never reaches the registry. The agent gets held to the human standard, automatically, with no human in the loop. That's the whole point.

---

<!-- layout: split -->

# One gate, two audiences

<!-- region -->

### Developers
- Fast pass / fail right in the PR
- The base is already good - nothing to remember
- Fix at authoring time, not after a rejection

<!-- region -->

### Platform · SRE · Ops
- **Own** the policy and hardened base - fleet-wide
- Signed attestations **are** the audit trail
- 2:47 AM incident → a query, not a forensics project

Note: This is the slide for the ops folks in the room. The same gate serves two audiences. Developers get fast pass/fail in the PR and a base image that's already good, so they fix things at authoring time instead of after a rejection. Platform, SRE and ops own the other side: they set the policy and the hardened base once and it applies fleet-wide, and the signed attestations become the audit trail. When the next 2:47 AM commit happens, answering "what shipped and where did it come from" is a query against attestations, not a week of forensics. Governance is a platform capability, not a developer chore.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="Govern the runtime: one boundary around the agent" style="position:absolute;inset:0;width:100%;height:100%;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="100" y="110" font-size="52" font-weight="800" fill="#ffffff">Move 4 - govern the <tspan fill="#1E9BF0">runtime</tspan>, not just the artifact</text>
  <text x="100" y="162" font-size="26" fill="#9AA6C2">One boundary around the agent: execution, tool calls, credentials.</text>
  <rect x="70" y="380" width="230" height="180" rx="14" fill="#12203F" stroke="#5A2A2A" stroke-width="2"/>
  <text x="102" y="420" font-size="20" font-weight="800" fill="#F0533F" letter-spacing="2">UNTRUSTED IN</text>
  <text x="102" y="460" font-size="23" fill="#ffffff">tickets · web</text>
  <text x="102" y="492" font-size="23" fill="#ffffff">code · docs</text>
  <text x="102" y="524" font-size="23" fill="#ffffff">tool output</text>
  <rect x="360" y="230" width="880" height="470" rx="28" fill="#0E1B3A" stroke="#1E9BF0" stroke-width="3" stroke-dasharray="11 8"/>
  <text x="400" y="284" font-size="24" font-weight="800" fill="#1E9BF0" letter-spacing="3">ONE RUNTIME BOUNDARY</text>
  <rect x="400" y="312" width="800" height="52" rx="12" fill="#12325E"/>
  <text x="424" y="338" font-size="23" font-weight="700" fill="#ffffff" dominant-baseline="central">Sandbox (microVM): isolated execution · host mounted read-only</text>
  <rect x="560" y="400" width="480" height="150" rx="16" fill="#173a6b"/>
  <text x="800" y="446" text-anchor="middle" font-size="30" font-weight="800" fill="#ffffff">THE AGENT</text>
  <text x="800" y="490" text-anchor="middle" font-size="21" fill="#C8D3F5">reads untrusted input,</text>
  <text x="800" y="518" text-anchor="middle" font-size="21" fill="#C8D3F5">acts with real credentials</text>
  <rect x="400" y="586" width="800" height="80" rx="12" fill="#12325E"/>
  <text x="424" y="616" font-size="23" font-weight="700" fill="#ffffff" dominant-baseline="central">MCP Gateway: signature-verified tools · scoped secrets</text>
  <text x="424" y="646" font-size="21" fill="#9AA6C2" dominant-baseline="central">cap_drop ALL · read_only rootfs · refuse anything unsigned</text>
  <rect x="1300" y="380" width="230" height="180" rx="14" fill="#0E2A1E" stroke="#34D399" stroke-width="2"/>
  <text x="1332" y="420" font-size="20" font-weight="800" fill="#34D399" letter-spacing="2">GOVERNED OUT</text>
  <text x="1332" y="462" font-size="23" fill="#ffffff">build · push</text>
  <text x="1332" y="494" font-size="23" fill="#ffffff">deploy</text>
  <text x="1332" y="526" font-size="20" fill="#9AA6C2">only what</text>
  <text x="1332" y="550" font-size="20" fill="#9AA6C2">policy allows</text>
  <g stroke="#5B8CFF" stroke-width="4" fill="#5B8CFF"><line x1="300" y1="470" x2="352" y2="470"/><polygon points="352,462 368,470 352,478"/></g>
  <g stroke="#34D399" stroke-width="4" fill="#34D399"><line x1="1240" y1="470" x2="1292" y2="470"/><polygon points="1292,462 1308,470 1292,478"/></g>
</svg>

Note: Reframe. Everything so far governs the artifact - the thing the agent built. This move governs the runtime: where the agent executes, and the tools and credentials it can reach. It's the same shift the industry is converging on - put a control point beneath the agent rather than hoping for a smarter agent. We do it in two places: the sandbox the agent builds in, and the gateway in front of the tools it calls.

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

Note: First, the sandbox. The agent runs inside a microVM - its own daemon, its own network, the host mounted read-only. It can pull, build and experiment, but it can't touch your machine or your network. And because the sandbox is wired to the hardened base and the DHI MCP, the agent reaches for the good base image on its own. Notice the Dockerfile it wrote: distroless DHI, non-root, multi-stage - with no correction from you. This is the goal: the red 2:47 AM build never happens, because the environment made the good path the default path.

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

Note: Second, the tools. An agent calls MCP servers - filesystem, git, github, databases - and each one is code running with access to your stuff. If those are unsigned images off the internet, you've governed the build and left the runtime wide open. DHI MCP is a catalog of hardened MCP servers: built on a DHI base, signed, with SBOMs, and the gateway verifies the signature before it runs one, then sandboxes it - read-only rootfs, all capabilities dropped, secrets scoped to the target. Same three moves - prove, harden, enforce - applied to the agent's tools, not just its artifacts.

---

<!-- layout: default -->

# Or watch it live

```bash terminal-id=demo
docker scout policy catalog-service:baseline
```

::terminal{id=demo height=300}

Note: This is the simulator from the hands-on lab - the exact commands you just saw. Run the policy on the 2:47 AM image and watch it fail three of three. In the lab you take that same image and walk it all the way to green: SBOM, hardened base, signature, and the CI gate. Let me point you there.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="The finish line: from an unapproved build to a governed one" style="position:absolute;inset:0;width:100%;height:100%;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="100" y="120" font-size="56" font-weight="800" fill="#ffffff">One frame</text>
  <text x="100" y="172" font-size="26" fill="#9AA6C2">From the build nobody approved to one you can prove.</text>
  <rect x="70" y="330" width="250" height="250" rx="16" fill="#2A1215" stroke="#F0533F" stroke-width="2"/>
  <text x="100" y="376" font-size="22" font-weight="800" fill="#F0533F" letter-spacing="2">02:47 BUILD</text>
  <text x="100" y="424" font-size="24" fill="#ffffff">FROM node:20</text>
  <text x="100" y="460" font-size="24" fill="#ffffff">431 packages</text>
  <text x="100" y="496" font-size="24" fill="#ffffff">runs as root</text>
  <text x="100" y="532" font-size="24" fill="#ffffff">unsigned</text>
  <text x="100" y="566" font-size="20" font-weight="700" fill="#F0533F">nothing you can prove</text>
  <g>
    <rect x="368" y="330" width="196" height="250" rx="14" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="466" y="378" text-anchor="middle" font-size="24" font-weight="800" fill="#1E9BF0">PROVE</text>
    <text x="466" y="448" text-anchor="middle" font-size="21" fill="#ffffff">SBOM</text>
    <text x="466" y="482" text-anchor="middle" font-size="21" fill="#ffffff">VEX</text>
    <text x="466" y="516" text-anchor="middle" font-size="21" fill="#ffffff">SLSA</text>
    <rect x="596" y="330" width="196" height="250" rx="14" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="694" y="378" text-anchor="middle" font-size="24" font-weight="800" fill="#1E9BF0">DEFAULT</text>
    <text x="694" y="456" text-anchor="middle" font-size="21" fill="#ffffff">Hardened</text>
    <text x="694" y="490" text-anchor="middle" font-size="21" fill="#ffffff">base (DHI)</text>
    <rect x="824" y="330" width="196" height="250" rx="14" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="922" y="378" text-anchor="middle" font-size="24" font-weight="800" fill="#1E9BF0">ENFORCE</text>
    <text x="922" y="456" text-anchor="middle" font-size="21" fill="#ffffff">Sign +</text>
    <text x="922" y="490" text-anchor="middle" font-size="21" fill="#ffffff">policy gate</text>
    <rect x="1052" y="330" width="196" height="250" rx="14" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="1150" y="378" text-anchor="middle" font-size="24" font-weight="800" fill="#1E9BF0">CONTAIN</text>
    <text x="1150" y="456" text-anchor="middle" font-size="21" fill="#ffffff">Runtime</text>
    <text x="1150" y="490" text-anchor="middle" font-size="21" fill="#ffffff">boundary</text>
  </g>
  <rect x="1296" y="330" width="234" height="250" rx="16" fill="#0E2A1E" stroke="#34D399" stroke-width="2"/>
  <text x="1413" y="376" text-anchor="middle" font-size="22" font-weight="800" fill="#34D399" letter-spacing="1">SHIPPED</text>
  <text x="1413" y="440" text-anchor="middle" font-size="22" fill="#ffffff">signed</text>
  <text x="1413" y="474" text-anchor="middle" font-size="22" fill="#ffffff">attested</text>
  <text x="1413" y="508" text-anchor="middle" font-size="22" fill="#ffffff">policy-gated</text>
  <text x="1413" y="556" text-anchor="middle" font-size="19" font-weight="700" fill="#34D399">close your laptop</text>
  <g stroke="#3A4A6E" stroke-width="4" fill="#3A4A6E">
    <line x1="324" y1="455" x2="360" y2="455"/><polygon points="360,447 376,455 360,463"/>
    <line x1="792" y1="455" x2="816" y2="455"/><polygon points="816,447 832,455 816,463"/>
    <line x1="1020" y1="455" x2="1044" y2="455"/><polygon points="1044,447 1060,455 1044,463"/>
  </g>
  <g stroke="#34D399" stroke-width="4" fill="#34D399"><line x1="1252" y1="455" x2="1288" y2="455"/><polygon points="1288,447 1304,455 1288,463"/></g>
  <text x="100" y="700" font-size="30" font-weight="700" fill="#ffffff">Prove · Default · Enforce · Contain - <tspan fill="#1E9BF0">all in the pipeline the change already runs through.</tspan></text>
  <text x="100" y="748" font-size="24" fill="#9AA6C2">One bar for humans and agents. Fails closed.</text>
</svg>

Note: Three moves, one model. Prove, default, enforce. None of them slow the agent down - they run in the pipeline the change already goes through. That's the goal: keep the agents fast, keep yourself in control. Those aren't in tension.

---

# Takeaways

1. A governance model that **fits your existing CI/CD** - not a parallel process.
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

Note: Right next to this deck is a hands-on lab that runs entirely in your browser - no install. You take the exact image the agent shipped and walk it through every move here: SBOM, hardened base, signature, CI gate. Bookmark it.

---

<!--
layout: section
chrome: false
-->

# Who approved that build?

Now you can answer - and prove it.

**Thank you. Questions?**

Note: Who approved that build? With evidence, a governed base, and an enforceable gate, the answer is: the same policy that approves every build, applied to the agent exactly as to a human, with a trail to prove it. Thank you - questions?

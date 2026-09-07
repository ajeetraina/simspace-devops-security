<!--
layout: title
chrome: false
eyebrow: "Docker · Agentic DevOps"
byline: "Ajeet Singh Raina - Developer Advocate, Docker"
-->

# Who Approved That Build?

## Docker AI Governance for the Agentic DevOps Pipeline

Note: The question in the title is a real one - who approved that build? By the end you'll have a way to answer it for your own pipeline. We spent a decade making delivery trustworthy - versioned pipelines, peer review, auditable deploys. Then we handed a set of keys to AI agents. Today we walk the whole road from development to production and make every segment of it provable - and we finish by packaging that boundary as a kit and operating it in prod. Let's get into it.

---

<!-- chrome: false -->

<img src="assets/slide-02.webp" alt="Meet your instructor: Ajeet Singh Raina, Developer Advocate at Docker, co-author of Operational AI with Docker" width="1600" height="900" loading="eager" fetchpriority="high" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Quick hello - I'm Ajeet Singh Raina, Developer Advocate at Docker. Twenty-plus years across system integration testing, consulting and developer relations; former Docker Captain, and I run the 17,000-member Docker Bengaluru meetup. I co-authored Operational AI with Docker with Harsh Manvar - on deploying, scaling and operating agentic AI services with Docker and Kubernetes, which is exactly the world this talk lives in, right down to the operations section at the end. Treat this as hands-on and interrupt me with questions.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="Agenda" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#ffffff" font-family="Arial, Helvetica, sans-serif">
  <defs>
    <linearGradient id="agrow1" x1="0" y1="0" x2="1" y2="0">
      <stop offset="0" stop-color="#2E86E6"/><stop offset="1" stop-color="#5FA8F1"/>
    </linearGradient>
  </defs>
  <rect x="0" y="0" width="1600" height="900" fill="#ffffff"/>
  <text x="96" y="168" font-size="96" font-weight="800" fill="#0A0A23">Agenda</text>
  <g>
    <g transform="translate(560,208)"><rect width="960" height="96" rx="48" fill="url(#agrow1)"/><circle cx="60" cy="48" r="37" fill="#EDEEF2"/><text x="60" y="48" text-anchor="middle" dominant-baseline="central" font-size="30" font-weight="800" fill="#3A3A4A">01</text><text x="122" y="48" dominant-baseline="central" font-size="32" font-weight="700" fill="#ffffff">The 02:47 AM incident</text></g>
    <g transform="translate(560,318)"><rect width="960" height="96" rx="48" fill="#2A6BD6"/><circle cx="60" cy="48" r="37" fill="#EDEEF2"/><text x="60" y="48" text-anchor="middle" dominant-baseline="central" font-size="30" font-weight="800" fill="#3A3A4A">02</text><text x="122" y="48" dominant-baseline="central" font-size="32" font-weight="700" fill="#ffffff">Evidence · SBOM · VEX · SLSA</text></g>
    <g transform="translate(560,428)"><rect width="960" height="96" rx="48" fill="#0B1550"/><circle cx="60" cy="48" r="37" fill="#EDEEF2"/><text x="60" y="48" text-anchor="middle" dominant-baseline="central" font-size="30" font-weight="800" fill="#3A3A4A">03</text><text x="122" y="48" dominant-baseline="central" font-size="32" font-weight="700" fill="#ffffff">Baseline · Docker Hardened Images</text></g>
    <g transform="translate(560,538)"><rect width="960" height="96" rx="48" fill="#0B1550"/><circle cx="60" cy="48" r="37" fill="#EDEEF2"/><text x="60" y="48" text-anchor="middle" dominant-baseline="central" font-size="30" font-weight="800" fill="#3A3A4A">04</text><text x="122" y="48" dominant-baseline="central" font-size="32" font-weight="700" fill="#ffffff">Gate · Securing your CI pipeline</text></g>
    <g transform="translate(560,648)"><rect width="960" height="96" rx="48" fill="#0B1550"/><circle cx="60" cy="48" r="37" fill="#EDEEF2"/><text x="60" y="48" text-anchor="middle" dominant-baseline="central" font-size="30" font-weight="800" fill="#3A3A4A">05</text><text x="122" y="48" dominant-baseline="central" font-size="32" font-weight="700" fill="#ffffff">Boundary · Securing the agentic stack</text></g>
    <g transform="translate(560,758)"><rect width="960" height="96" rx="48" fill="#0B1550"/><circle cx="60" cy="48" r="37" fill="#EDEEF2"/><text x="60" y="48" text-anchor="middle" dominant-baseline="central" font-size="30" font-weight="800" fill="#3A3A4A">06</text><text x="122" y="48" dominant-baseline="central" font-size="32" font-weight="700" fill="#ffffff">Make it yours · sbx kit + operate in prod</text></g>
  </g>
</svg>

Note: Here's the road, and it doubles as the four questions we'll keep coming back to. We open with the incident. Then Evidence - SBOM, VEX, SLSA. Then Baseline - Docker Hardened Images. Then the Gate - the CI pipeline that turns policy into a fail-closed boundary. Then the Boundary - sandboxing the agent and governing its tools at a gateway. And finally, the part that's new today: make it yours - package that boundary as an sbx kit, and operate the whole thing in production. A note on audience: the first half feels like a developer talk, but the gate, the baseline, the audit trail and the operations story are owned by platform, SRE and ops - I'll call out who owns what as we go. But first, why any of this is necessary at all - what agents now do unsupervised, and what happens when it goes wrong.

---

<!-- layout: section -->

# Autonomy requires guardrails

Agents now act unsupervised, at machine speed, on every repo. The case for governance - before any Docker feature.

Note: This is where the talk really starts. Before any product, the argument: agents now act on their own at machine speed, they're built on a risk model you can't prompt away, and the failures are already in the wild. Three quick beats - what agents do, why they're dangerous by design, and what goes wrong - then we meet the 2:47 AM commit.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="AI agents are here and doing real work" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="80" y="104" font-size="46" font-weight="800" fill="#ffffff">AI agents are here. <tspan fill="#F0A84A">And they're doing real work.</tspan></text>
  <g>
    <rect x="80" y="176" width="460" height="560" rx="16" fill="#14346E"/>
    <text x="120" y="252" font-size="22" font-weight="800" fill="#9FC0F0" letter-spacing="2">ENGINEERING</text>
    <text x="120" y="336" font-size="30" font-weight="800" fill="#ffffff">Read whole codebases.</text>
    <text x="120" y="378" font-size="30" font-weight="800" fill="#ffffff">Ship pull requests.</text>
    <text x="120" y="476" font-size="21" fill="#C8D3F5">Writes code and opens PRs</text>
    <text x="120" y="506" font-size="21" fill="#C8D3F5">with no engineer in the loop.</text>
  </g>
  <g>
    <rect x="570" y="176" width="460" height="560" rx="16" fill="#14346E"/>
    <text x="610" y="252" font-size="22" font-weight="800" fill="#9FC0F0" letter-spacing="2">MARKETING</text>
    <text x="610" y="336" font-size="30" font-weight="800" fill="#ffffff">Pull CRM data.</text>
    <text x="610" y="378" font-size="30" font-weight="800" fill="#ffffff">Launch campaigns.</text>
    <text x="610" y="476" font-size="21" fill="#C8D3F5">Research to creative to send,</text>
    <text x="610" y="506" font-size="21" fill="#C8D3F5">end to end.</text>
  </g>
  <g>
    <rect x="1060" y="176" width="460" height="560" rx="16" fill="#14346E"/>
    <text x="1100" y="252" font-size="22" font-weight="800" fill="#9FC0F0" letter-spacing="2">FINANCE</text>
    <text x="1100" y="336" font-size="30" font-weight="800" fill="#ffffff">Reconcile reports.</text>
    <text x="1100" y="378" font-size="30" font-weight="800" fill="#ffffff">Query systems live.</text>
    <text x="1100" y="476" font-size="21" fill="#C8D3F5">Ledger, dashboard, decision -</text>
    <text x="1100" y="506" font-size="21" fill="#C8D3F5">closed in one loop.</text>
  </g>
  <text x="800" y="810" text-anchor="middle" font-size="34" font-weight="800" fill="#ffffff">Agents are the biggest productivity shift in decades.</text>
</svg>

Note: Start with the upside, honestly. Agents are already doing real work across every function - engineering ships PRs, marketing runs campaigns end to end, finance reconciles and queries live systems. This is the biggest productivity shift in decades, and nobody is putting it back in the box. The point of this whole talk isn't to slow that down - it's to make it safe to go this fast.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="Then came Claws - the exploding agent ecosystem" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="72" y="150" font-size="50" font-weight="800" fill="#ffffff">Then came <tspan fill="#1E9BF0">Claws.</tspan></text>
  <g font-size="22" fill="#9AA6C2">
    <text x="72" y="212">Agents that don't just answer - they act.</text>
    <text x="72" y="246">You chat with a Claw and it acts on its own:</text>
    <text x="72" y="280">sends the email, updates the record, makes</text>
    <text x="72" y="314">the payment - all on your behalf.</text>
  </g>
  <rect x="72" y="368" width="600" height="184" rx="14" fill="#0E1B3A" stroke="#1E9BF0" stroke-width="2"/>
  <text x="100" y="418" font-size="20" font-weight="800" fill="#ffffff">Claws turn read access into write access.</text>
  <g font-size="18" fill="#C8D3F5">
    <text x="100" y="452">Every employee now runs an agent touching</text>
    <text x="100" y="480">customer records, financial systems, and the</text>
    <text x="100" y="508">open internet - with their identity attached.</text>
  </g>
  <text x="760" y="118" font-size="26" font-weight="800" fill="#ffffff">The Claw Ecosystem</text>
  <text x="760" y="148" font-size="17" fill="#9AA6C2">Open-source personal-agent projects in the OpenClaw family</text>
  <g transform="translate(760,180)"><rect width="240" height="150" rx="12" fill="#12203F" stroke="#26365C" stroke-width="1.5"/><rect x="20" y="22" width="34" height="34" rx="8" fill="#1E6FE6"/><text x="66" y="45" font-size="18" font-weight="800" fill="#ffffff">OpenClaw</text><rect x="160" y="26" width="60" height="24" rx="12" fill="#1B2A4A"/><text x="190" y="42" text-anchor="middle" font-size="12" font-weight="700" fill="#9FC0F0">★ 392k</text><text x="20" y="94" font-size="14" fill="#9AA6C2">Flagship personal AI,</text><text x="20" y="114" font-size="14" fill="#9AA6C2">the fediverse way</text></g>
  <g transform="translate(1016,180)"><rect width="240" height="150" rx="12" fill="#12203F" stroke="#26365C" stroke-width="1.5"/><rect x="20" y="22" width="34" height="34" rx="8" fill="#F0A84A"/><text x="66" y="45" font-size="18" font-weight="800" fill="#ffffff">ZeroClaw</text><rect x="160" y="26" width="60" height="24" rx="12" fill="#1B2A4A"/><text x="190" y="42" text-anchor="middle" font-size="12" font-weight="700" fill="#9FC0F0">★ 22k</text><text x="20" y="94" font-size="14" fill="#9AA6C2">Fast, autonomous</text><text x="20" y="114" font-size="14" fill="#9AA6C2">infra (Rust)</text></g>
  <g transform="translate(1272,180)"><rect width="240" height="150" rx="12" fill="#12203F" stroke="#26365C" stroke-width="1.5"/><rect x="20" y="22" width="34" height="34" rx="8" fill="#34D399"/><text x="66" y="45" font-size="18" font-weight="800" fill="#ffffff">PicoClaw</text><rect x="160" y="26" width="60" height="24" rx="12" fill="#1B2A4A"/><text x="190" y="42" text-anchor="middle" font-size="12" font-weight="700" fill="#9FC0F0">★ 38k</text><text x="20" y="94" font-size="14" fill="#9AA6C2">Tiny, deploy-anywhere</text><text x="20" y="114" font-size="14" fill="#9AA6C2">agent</text></g>
  <g transform="translate(760,350)"><rect width="240" height="150" rx="12" fill="#12203F" stroke="#26365C" stroke-width="1.5"/><rect x="20" y="22" width="34" height="34" rx="8" fill="#8B5CF6"/><text x="66" y="45" font-size="18" font-weight="800" fill="#ffffff">NanoClaw</text><rect x="160" y="26" width="60" height="24" rx="12" fill="#1B2A4A"/><text x="190" y="42" text-anchor="middle" font-size="12" font-weight="700" fill="#9FC0F0">★ 50k</text><text x="20" y="94" font-size="14" fill="#9AA6C2">Lightweight,</text><text x="20" y="114" font-size="14" fill="#9AA6C2">container-based</text></g>
  <g transform="translate(1016,350)"><rect width="240" height="150" rx="12" fill="#12203F" stroke="#26365C" stroke-width="1.5"/><rect x="20" y="22" width="34" height="34" rx="8" fill="#EF4444"/><text x="66" y="45" font-size="18" font-weight="800" fill="#ffffff">NemoClaw</text><rect x="160" y="26" width="60" height="24" rx="12" fill="#1B2A4A"/><text x="190" y="42" text-anchor="middle" font-size="12" font-weight="700" fill="#9FC0F0">★ 22k</text><text x="20" y="94" font-size="14" fill="#9AA6C2">Sandboxed agents,</text><text x="20" y="114" font-size="14" fill="#9AA6C2">managed inference</text></g>
  <g transform="translate(1272,350)"><rect width="240" height="150" rx="12" fill="#12203F" stroke="#26365C" stroke-width="1.5"/><rect x="20" y="22" width="34" height="34" rx="8" fill="#0EA5E9"/><text x="66" y="45" font-size="18" font-weight="800" fill="#ffffff">ClawHub</text><rect x="160" y="26" width="60" height="24" rx="12" fill="#1B2A4A"/><text x="190" y="42" text-anchor="middle" font-size="12" font-weight="700" fill="#9FC0F0">★ 8k</text><text x="20" y="94" font-size="14" fill="#9AA6C2">Skill + plugin</text><text x="20" y="114" font-size="14" fill="#9AA6C2">registry</text></g>
  <g transform="translate(760,520)"><rect width="240" height="150" rx="12" fill="#12203F" stroke="#26365C" stroke-width="1.5"/><rect x="20" y="22" width="34" height="34" rx="8" fill="#EC4899"/><text x="66" y="45" font-size="18" font-weight="800" fill="#ffffff">MicroClaw</text><rect x="160" y="26" width="60" height="24" rx="12" fill="#1B2A4A"/><text x="190" y="42" text-anchor="middle" font-size="12" font-weight="700" fill="#9FC0F0">★ 730</text><text x="20" y="94" font-size="14" fill="#9AA6C2">Chat-based agent,</text><text x="20" y="114" font-size="14" fill="#9AA6C2">built in Rust</text></g>
  <g transform="translate(1016,520)"><rect width="240" height="150" rx="12" fill="#12203F" stroke="#26365C" stroke-width="1.5"/><rect x="20" y="22" width="34" height="34" rx="8" fill="#14B8A6"/><text x="66" y="45" font-size="18" font-weight="800" fill="#ffffff">TinyClaw</text><rect x="160" y="26" width="60" height="24" rx="12" fill="#1B2A4A"/><text x="190" y="42" text-anchor="middle" font-size="12" font-weight="700" fill="#9FC0F0">★ 288</text><text x="20" y="94" font-size="14" fill="#9AA6C2">The original tiny</text><text x="20" y="114" font-size="14" fill="#9AA6C2">Claw companion</text></g>
  <g transform="translate(1272,520)"><rect width="240" height="150" rx="12" fill="#12203F" stroke="#26365C" stroke-width="1.5"/><rect x="20" y="22" width="34" height="34" rx="8" fill="#6366F1"/><text x="66" y="45" font-size="18" font-weight="800" fill="#ffffff">SeClaw</text><rect x="160" y="26" width="60" height="24" rx="12" fill="#1B2A4A"/><text x="190" y="42" text-anchor="middle" font-size="12" font-weight="700" fill="#9FC0F0">★ 166</text><text x="20" y="94" font-size="14" fill="#9AA6C2">Security auditing &amp;</text><text x="20" y="114" font-size="14" fill="#9AA6C2">agent-safety eval</text></g>
</svg>

Note: And it is accelerating. A whole ecosystem of personal agents - I'll call them Claws - has exploded, open-source, one for every stack. What is new isn't that they answer; it is that they act. A Claw sends the email, updates the record, makes the payment. That one shift is the whole security story: Claws turn read access into write access, and every employee now runs one with their own identity attached. The names here are a stand-in - the real landscape is just as crowded.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="The lethal trifecta every useful agent shares" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="80" y="100" font-size="44" font-weight="800" fill="#ffffff">Every useful agent is built on the <tspan fill="#F0A84A">"lethal trifecta."</tspan></text>
  <g transform="translate(80,150)">
    <rect width="460" height="440" rx="16" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="36" y="64" font-size="18" font-weight="700" fill="#6E86B8" letter-spacing="2">RISK · 01</text>
    <text x="36" y="118" font-size="28" font-weight="800" fill="#ffffff">Access to private data</text>
    <g font-size="19" fill="#C8D3F5">
      <text x="36" y="168">Codebases, internal systems,</text>
      <text x="36" y="198">customer records, financial</text>
      <text x="36" y="228">ledgers. The same data that</text>
      <text x="36" y="258">makes the agent useful makes</text>
      <text x="36" y="288">it dangerous.</text>
    </g>
    <text x="36" y="404" font-size="16" font-weight="700" fill="#F0A84A" letter-spacing="1">▸ INTERNAL · SENSITIVE · REGULATED</text>
  </g>
  <g transform="translate(570,150)">
    <rect width="460" height="440" rx="16" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="36" y="64" font-size="18" font-weight="700" fill="#6E86B8" letter-spacing="2">RISK · 02</text>
    <text x="36" y="118" font-size="28" font-weight="800" fill="#ffffff">Exposure to untrusted</text>
    <text x="36" y="152" font-size="28" font-weight="800" fill="#ffffff">content</text>
    <g font-size="19" fill="#C8D3F5">
      <text x="36" y="200">Web pages, emails, MCP</text>
      <text x="36" y="230">responses, files. Any of it can</text>
      <text x="36" y="260">carry instructions the agent</text>
      <text x="36" y="290">will follow as if you typed them.</text>
    </g>
    <text x="36" y="404" font-size="16" font-weight="700" fill="#F0A84A" letter-spacing="1">▸ PROMPT INJECTION · POISONED INPUT</text>
  </g>
  <g transform="translate(1060,150)">
    <rect width="460" height="440" rx="16" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="36" y="64" font-size="18" font-weight="700" fill="#6E86B8" letter-spacing="2">RISK · 03</text>
    <text x="36" y="118" font-size="28" font-weight="800" fill="#ffffff">Ability to act externally</text>
    <g font-size="19" fill="#C8D3F5">
      <text x="36" y="168">Sending email, calling APIs,</text>
      <text x="36" y="198">touching the open internet.</text>
      <text x="36" y="228">Once data leaves, it does</text>
      <text x="36" y="258">not come back.</text>
    </g>
    <text x="36" y="404" font-size="16" font-weight="700" fill="#F0A84A" letter-spacing="1">▸ EXFIL · UNBOUNDED EGRESS</text>
  </g>
  <rect x="80" y="628" width="1440" height="150" rx="14" fill="#171E30" stroke="#3A4A6E" stroke-width="2"/>
  <text x="116" y="674" font-size="16" font-weight="800" fill="#F0A84A" letter-spacing="2">THE POINT</text>
  <text x="116" y="712" font-size="24" fill="#ffffff">A useful agent has <tspan font-weight="800" fill="#F0A84A">all three by design.</tspan> You can't train it out, prompt it</text>
  <text x="116" y="748" font-size="24" fill="#ffffff">out, or policy-doc it out. The only fix is an enforcement layer at the runtime.</text>
</svg>

Note: Here's why you can't prompt your way out of this. Every genuinely useful agent has three properties at once - access to private data, exposure to untrusted content, and the ability to act in the outside world. Simon Willison named this the lethal trifecta. Any one alone is fine; all three together mean untrusted input can turn your own data into an outbound action. And you can't train it out or write a policy doc that fixes it - the only real fix is an enforcement layer at runtime. Hold that thought; it's the whole back half of this talk.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="Traditional versus agentic developer workflow" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <defs>
    <g id="p8p"><circle cx="0" cy="-9" r="10" fill="#7FB2E6"/><path d="M-15,15 C-15,4 -7,0 0,0 C7,0 15,4 15,15 Z" fill="#7FB2E6"/></g>
    <g id="p8w"><path d="M0,-13 L14,12 L-14,12 Z" fill="none" stroke="#F0A84A" stroke-width="2.5" stroke-linejoin="round"/><rect x="-1.6" y="-5" width="3.2" height="9" rx="1.6" fill="#F0A84A"/><circle cx="0" cy="8" r="1.9" fill="#F0A84A"/></g>
  </defs>
  <text x="72" y="92" font-size="44" font-weight="800" fill="#ffffff">Traditional vs Agentic Developer Workflow</text>
  <rect x="72" y="150" width="680" height="520" rx="18" fill="#12203F" stroke="#26365C" stroke-width="2"/>
  <text x="112" y="218" font-size="22" font-weight="800" fill="#9FC0F0" letter-spacing="1">TRADITIONAL WORKFLOW</text>
  <use href="#p8p" transform="translate(132,300)"/>
  <text x="164" y="306" font-size="19" fill="#E8EDF7"><tspan font-weight="700">Developer pulls base image</tspan> <tspan fill="#9AA6C2">- manually, with intent</tspan></text>
  <use href="#p8p" transform="translate(132,370)"/>
  <text x="164" y="376" font-size="19" fill="#E8EDF7"><tspan font-weight="700">Developer installs dependencies</tspan> <tspan fill="#9AA6C2">- reviewed in a PR</tspan></text>
  <use href="#p8p" transform="translate(132,440)"/>
  <text x="164" y="446" font-size="19" fill="#E8EDF7"><tspan font-weight="700">CI pipeline runs</tspan> <tspan fill="#9AA6C2">- with human-authored config</tspan></text>
  <rect x="792" y="150" width="680" height="520" rx="18" fill="#12203F" stroke="#F0A84A" stroke-width="2"/>
  <text x="832" y="218" font-size="22" font-weight="800" fill="#F0A84A" letter-spacing="1">AGENTIC WORKFLOW</text>
  <use href="#p8w" transform="translate(832,290)"/>
  <text x="868" y="296" font-size="19" fill="#E8EDF7"><tspan font-weight="700">Agent pulls base image</tspan> - autonomously</text>
  <use href="#p8w" transform="translate(832,352)"/>
  <text x="868" y="358" font-size="19" fill="#E8EDF7"><tspan font-weight="700">Agent installs packages</tspan> - no human review</text>
  <use href="#p8w" transform="translate(832,414)"/>
  <text x="868" y="420" font-size="19" fill="#E8EDF7"><tspan font-weight="700">Agent invokes external tools</tspan> - with real credentials</text>
  <use href="#p8w" transform="translate(832,476)"/>
  <text x="868" y="482" font-size="19" fill="#E8EDF7"><tspan font-weight="700">Agent modifies Dockerfile</tspan> - mid-pipeline</text>
  <rect x="72" y="712" width="6" height="46" fill="#1E9BF0"/>
  <text x="98" y="746" font-size="24" font-style="italic" fill="#C8D3F5">"The better the agent, the bigger the blast radius."</text>
</svg>

Note: Bring it home to our world - the build pipeline. In the traditional workflow a developer pulls the base image, installs dependencies in a reviewed PR, and CI runs config a human wrote - intent at every step. In the agentic workflow the agent does all of it autonomously: pulls the base image, resolves packages with no review, invokes external tools with real credentials, and rewrites the Dockerfile mid-pipeline. Same supply chain, no human in the loop. The better the agent, the bigger the blast radius.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="The traditional developer workflow, a human at every stage" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <defs>
    <g id="tperson"><circle cx="0" cy="-11" r="12" fill="#7FB2E6"/><path d="M-18,17 C-18,4 -9,0 0,0 C9,0 18,4 18,17 Z" fill="#7FB2E6"/></g>
  </defs>
  <text x="72" y="82" font-size="44" font-weight="800" fill="#ffffff">The Traditional Workflow</text>
  <text x="72" y="124" font-size="22" fill="#9AA6C2">A human writes, reviews, and ships at every stage - the attack surface is only what you choose to pull.</text>
  <circle cx="430" cy="500" r="120" fill="none" stroke="#1E9BF0" stroke-width="16"/>
  <text x="430" y="494" text-anchor="middle" font-size="26" font-weight="700" fill="#ffffff">Inner</text>
  <text x="430" y="524" text-anchor="middle" font-size="26" font-weight="700" fill="#ffffff">Loop</text>
  <use href="#tperson" transform="translate(430,362)"/>
  <text x="430" y="326" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Build</text>
  <use href="#tperson" transform="translate(298,500)"/>
  <text x="248" y="505" text-anchor="end" font-size="22" font-weight="700" fill="#C8D3F5">Code</text>
  <use href="#tperson" transform="translate(345,602)"/>
  <text x="345" y="650" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Open Source</text>
  <use href="#tperson" transform="translate(515,602)"/>
  <text x="515" y="650" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Test</text>
  <line x1="600" y1="500" x2="852" y2="500" stroke="#1E9BF0" stroke-width="6"/>
  <polygon points="852,490 872,500 852,510" fill="#1E9BF0"/>
  <text x="726" y="478" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Push</text>
  <circle cx="1092" cy="500" r="170" fill="none" stroke="#1E9BF0" stroke-width="16"/>
  <text x="1092" y="494" text-anchor="middle" font-size="26" font-weight="700" fill="#ffffff">Outer</text>
  <text x="1092" y="524" text-anchor="middle" font-size="26" font-weight="700" fill="#ffffff">Loop</text>
  <use href="#tperson" transform="translate(1092,320)"/>
  <text x="1092" y="286" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Integrate</text>
  <use href="#tperson" transform="translate(1264,500)"/>
  <text x="1300" y="505" text-anchor="start" font-size="22" font-weight="700" fill="#C8D3F5">Test</text>
  <use href="#tperson" transform="translate(1092,680)"/>
  <text x="1092" y="728" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Deploy</text>
</svg>

Note: One more way to see it. This is the development lifecycle we've always had - an inner loop of code, build, test, and an outer loop of integrate, test, deploy, with a human standing at every node. The attack surface was bounded: it was only what you chose to pull. Every arrow here ran through a person.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="The agentic developer workflow, an agent at every stage" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <defs>
    <g id="abot"><rect x="-17" y="-15" width="34" height="27" rx="7" fill="#34D399"/><rect x="-10" y="-7" width="20" height="12" rx="2.5" fill="#0B1533"/><text x="0" y="3" text-anchor="middle" font-size="9" font-weight="800" fill="#34D399">AI</text><line x1="0" y1="-15" x2="0" y2="-24" stroke="#34D399" stroke-width="2.5"/><circle cx="0" cy="-26" r="3.2" fill="#34D399"/></g>
  </defs>
  <text x="72" y="82" font-size="44" font-weight="800" fill="#ffffff">The Agentic Workflow</text>
  <text x="72" y="124" font-size="22" fill="#9AA6C2">Now an agent sits at every stage - the attack surface is no longer just what you pull.</text>
  <circle cx="430" cy="500" r="120" fill="none" stroke="#1E9BF0" stroke-width="16"/>
  <text x="430" y="494" text-anchor="middle" font-size="26" font-weight="700" fill="#ffffff">Inner</text>
  <text x="430" y="524" text-anchor="middle" font-size="26" font-weight="700" fill="#ffffff">Loop</text>
  <use href="#abot" transform="translate(430,360)"/>
  <text x="430" y="322" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Build</text>
  <use href="#abot" transform="translate(298,500)"/>
  <text x="246" y="505" text-anchor="end" font-size="22" font-weight="700" fill="#C8D3F5">Code</text>
  <use href="#abot" transform="translate(345,600)"/>
  <text x="345" y="650" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Open Source</text>
  <use href="#abot" transform="translate(515,600)"/>
  <text x="515" y="650" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Test</text>
  <line x1="600" y1="500" x2="852" y2="500" stroke="#1E9BF0" stroke-width="6"/>
  <polygon points="852,490 872,500 852,510" fill="#1E9BF0"/>
  <text x="726" y="478" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Push</text>
  <circle cx="1092" cy="500" r="170" fill="none" stroke="#1E9BF0" stroke-width="16"/>
  <text x="1092" y="494" text-anchor="middle" font-size="26" font-weight="700" fill="#ffffff">Outer</text>
  <text x="1092" y="524" text-anchor="middle" font-size="26" font-weight="700" fill="#ffffff">Loop</text>
  <use href="#abot" transform="translate(1092,318)"/>
  <text x="1092" y="284" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Integrate</text>
  <use href="#abot" transform="translate(1264,500)"/>
  <text x="1300" y="505" text-anchor="start" font-size="22" font-weight="700" fill="#C8D3F5">Test</text>
  <use href="#abot" transform="translate(1092,680)"/>
  <text x="1092" y="728" text-anchor="middle" font-size="22" font-weight="700" fill="#C8D3F5">Deploy</text>
</svg>

Note: Now replace every human with an agent. Same loops, but an agent sits at each stage, acting autonomously with real credentials. The attack surface is no longer just what you pull - it's every autonomous action, every tool call, every credential the agents touch across the whole road. That's the 2:47 AM commit, generalized to every stage - which is exactly where we're headed next.

---

<!-- layout: section -->

# What can go wrong?

Note: So: agents everywhere, acting for us, built on a trifecta you can't prompt away, at every stage of the pipeline. What actually happens when that runs unsupervised? Not hypotheticals - here's what already shipped.

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

<!-- chrome: false -->

<img src="assets/slide-horror-1.webp" alt="AI coding agent horror story: asked to clean up a project folder, an agent with root access runs rm -rf * and deletes .ssh, .aws, .env, logs and the production database" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Story one, from Docker's own write-up. Someone asked an agent to "clean up my project folder." The agent had root the whole time, ran rm -rf *, and took .ssh keys, .aws credentials, and the production database with it. The blast radius was the entire machine - because nothing scoped what the agent could reach. Source: docker.com/blog/ai-coding-agent-horror-stories-security-risks.

---

<!-- chrome: false -->

<img src="assets/slide-horror-2.webp" alt="Claude Cowork horror story: asked to organize a desktop with 'temporary files only' permission, the agent runs rm -rf family_photos, bypasses the macOS Trash, and 15 years of photos are only saved by iCloud 30-day retention" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Story two. "Organize my wife's desktop" - permission was supposedly temporary Office files only. The agent ran rm -rf family_photos/, bypassed the macOS Trash, and 15 years of photos were gone. They got lucky - iCloud's 30-day retention still had a copy. Same story, different path, same damage - and luck is not a control. Source: docker.com/blog/coding-agent-horror-stories-the-rm-rf-incident.

---

<!-- chrome: false -->

<img src="assets/slide-09.webp" alt="The ungoverned agent: agent running straight on your host with no boundary, FROM node:20 chosen with no guidance, 6 high CVEs" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: This is our ungoverned baseline - the agent running straight on your host, exactly how a lot of teams run today. It sits right on your machine, with the host daemon and host credentials, no boundary at all. We hand it a simple prompt - "containerize this app" - and with full permissions and open registries, it grabs whatever it wants: FROM node:20, chosen with no guidance. The result is the number we keep coming back to: 0 critical, 6 high, 30 medium, 54 low CVEs, 431 packages, no SBOM, no attestation, running as root. That's the start line.

---

<!-- chrome: false -->

<img src="assets/slide-08.webp" alt="The Product Catalog service we will secure: catalog-service writing to PostgreSQL, pushing images to S3, publishing to Kafka, and calling an Inventory service" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: And the app it mangled isn't a toy. It's a Product Catalog service, deliberately realistic: the catalog-service writes to PostgreSQL, pushes images to S3, publishes updates through Kafka, and calls an Inventory service and other downstream systems. Every one of those boxes is something we eventually have to trust and prove. This is the real supply chain we'll walk from development all the way to production.

---

<!-- chrome: false -->

<img src="assets/slide-framework.webp" alt="Every agent-driven change answers four questions: Evidence (what is in it, where from), Baseline (did it start trustworthy), Gate (is it allowed to pass), Boundary (what could it reach)" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: No matter how the agent produced a change, governing it comes down to four questions - and these are the layers of the road we're about to walk. Evidence: what's in this artifact, and where did it come from - SBOM, VEX, SLSA. Baseline: did it start from something trustworthy - a Docker Hardened Image. Gate: is it allowed to pass - build policies, signing, admission. Boundary: what could it reach while it worked - the sandbox runtime. Evidence and baseline make governance possible; gate and boundary make it real. Next, the road itself.

---

<!-- chrome: false -->

<img src="assets/slide-journey-0.webp" alt="The journey, checkpoint 0 of 4: the whole development-to-production road, everything still to prove, red baseline hot" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: This is the whole road we travel today, and it flows left to right, development to production. On the left, DEVELOPMENT: the agent works inside an sbx microVM, host read-only. On the right, PRODUCTION: the runtime is locked down - read-only, cap-drop ALL, non-root. The CI GATE in the middle is the dev-to-prod boundary, and it fails closed - nothing crosses unless it's provable. Right now none of it is provable: the ungoverned baseline is FROM node:20, 431 packages, no SBOM, root - 0 of 4 stages green. Each segment turns green as we go. This is checkpoint 0 - the start line.

---

<!-- chrome: false -->

<img src="assets/slide-framework-1.webp" alt="Question 1 of 4 - Evidence: what is in this, and where did it come from? SBOM, VEX, SLSA provenance" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Question one - Evidence: what is actually in the image, and where did it come from? The answer is SBOM, VEX, and SLSA provenance - the audit trail that finally lets you answer "who approved that build?" You can't govern what you can't see, so this is the layer everything else is built on.

---

<!-- chrome: false -->

<img src="assets/slide-19.webp" alt="The three building blocks: SBOM, VEX and SLSA - especially when agents are doing the pulling" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Three building blocks carry this whole layer. SBOM tells you what's inside. VEX tells you which of those CVEs are actually exploitable in your context, so you're not chasing noise. And SLSA gives you the provenance that proves how it was built. The subtitle is the whole reason we're here: this matters especially when agents are doing the pulling, because the agent won't ask permission before grabbing a base image.

---

<!-- chrome: false -->

<img src="assets/slide-22.webp" alt="SBOM - your software ingredient list: Docker Scout matches PURLs against an advisory database aggregated from 23 sources" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Start with the SBOM - your software ingredient list. It's the complete list of every package inside the image. Docker Scout matches those packages, expressed as PURLs, against an advisory database aggregated from 23 sources - PURL-based, not CPE, which keeps false positives down. Generating one is a single flag on your existing build: docker buildx build --attest type=sbom. No separate pipeline to stand up.

---

<!-- chrome: false -->

<img src="assets/slide-25.webp" alt="Without VEX vs with VEX: a raw scan lists every CVE; VEX marks each Not Affected / Affected / Fixed - 190 not affected, 10 fixed" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Here's why VEX matters, side by side. Without VEX, a standard scan lists every CVE against every package - libc6 over and over, "won't fix," and a human has to chase every line. With VEX, each CVE carries a statement: Not Affected, Affected, Fixed, or Under investigation. You pull it with one command - docker scout vex get. The punchline at the bottom: 190 not affected, 10 fixed. That's the noise gone and the signal left - the difference between drowning in alerts and making a risk-based decision.

---

<!-- chrome: false -->

<img src="assets/slide-26.webp" alt="SLSA - Supply chain Levels for Software Artifacts: four levels L0-L3, DHI targets L3 with signed, non-falsifiable provenance" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: And SLSA - Supply chain Levels for Software Artifacts - is the provenance framework. Four progressive levels: L0 is no guarantees, L1 means provenance merely exists, L2 is a hosted build with signed provenance - GitHub Actions with OIDC gets you there - and L3 is the hardened, non-falsifiable target DHI aims for. The one question it makes answerable: can you prove this artifact came from that source and wasn't tampered with in transit? DHI gives you the signed provenance envelope and verification with Cosign or Notation - a one-liner to consume.

---

<!-- chrome: false -->

<img src="assets/slide-journey-1.webp" alt="The journey, checkpoint 1 of 4: Lab 1 done, the BUILD stage is now green - you can see what is in the image" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Checkpoint one - Evidence is done, so BUILD goes green on the road. Buildx attached an SBOM plus provenance at build time, so the box is no longer a black hole - 1 of 4 stages provable. That baseline strip underneath is the reminder of where we started: FROM node:20, 431 packages, no SBOM, root, nothing you can prove. Next we tackle the segment just to the left of BUILD - the base image itself.

---

<!-- chrome: false -->

<img src="assets/slide-framework-2.webp" alt="Question 2 of 4 - Baseline: did it start from something trustworthy? Docker Hardened Images" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Question two - Baseline: did the image start from something trustworthy? The answer is a Docker Hardened Image instead of whatever the agent grabbed off the internet. Evidence told us what's in the box; baseline makes sure we began from a good one. This is the highest-leverage decision in the whole pipeline, because the agent makes it on every single build.

---

<!-- chrome: false -->

<img src="assets/slide-31.webp" alt="Three properties of a Docker Hardened Image: Minimal (95% smaller), Attested (SBOM, VEX, SLSA L3, signature), Patched (near-zero CVEs)" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Three properties define a hardened image. Minimal: built from source with only what your runtime needs - no shell, no curl - which is where the 95% smaller number and the shrunken attack surface come from. Attested: every image ships with an SBOM, a VEX document, SLSA L3 provenance, and a signature, so you verify in one command instead of trusting a label. Patched: continuously updated, so you get near-zero CVEs on day one and the Docker team keeps it that way. Minimal shrinks the surface, attested makes it provable, patched keeps it clean.

---

<!-- chrome: false -->

<img src="assets/slide-32.webp" alt="docker scout compare: node:22-slim (2C 26H 25M 122L, 806 packages, 398MB) versus dhi.io/node:24-debian13 (0,0,0,0, 211 packages, 40MB)" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Here's the same docker scout compare you'd run yourself. On the left, node:22-slim: 2 Critical, 26 High, 25 Medium, 122 Low, 806 packages, 398 MB. On the right, dhi.io/node:24-debian13: zero in every bucket. Packages drop from 806 to 211 - 595 fewer things to patch and audit - and size falls 90%, to 40 MB. Fewer packages is why there are fewer CVEs: you can't have a vulnerability in software you never shipped. That column of zeros is what a one-line base swap buys you.

---

<!-- chrome: false -->

<img src="assets/slide-33.webp" alt="The catalog-service migration: node:22-slim becomes a two-stage build on dhi.io/node dev + distroless runtime; USER/useradd lines drop because DHI is non-root" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: This is the actual migration for catalog-service, before and after. The change that matters is the FROM line: node:22-slim becomes a two-stage build - dhi.io/node dev image for the build stage where you run npm ci, and the distroless final stage that copies node_modules across. Notice what drops out: no more RUN useradd, no USER appuser, because DHI already runs non-root. The runtime stage is distroless - no shell, no npm - the source is unchanged, and the Compose file doesn't change at all. This is a base swap, not a rewrite.

---

<!-- chrome: false -->

<img src="assets/slide-11.webp" alt="Catalog service, where vulnerabilities enter: without the DHI MCP the agent picks base images freely (2 Critical, 46+ High); with the DHI MCP every service resolves to a hardened image (0 Critical, 0 High)" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Zoom out from one service to the whole stack - this is where vulnerabilities actually enter. On the left, without the DHI MCP the agent picks base images freely: the app on node:20, PostgreSQL, Kafka, aws-sdk - total stack exposure 2 Critical and 46-plus High. On the right, with the DHI MCP the agent queries first and every service resolves to a hardened image - and the whole column collapses to 0 across the board. The point isn't just that DHI is cleaner; it's that when the agent has to ask before it picks, the vulnerabilities never enter the stack in the first place.

---

<!-- chrome: false -->

<img src="assets/slide-journey-2.webp" alt="The journey, checkpoint 2 of 4: Lab 2 done, BASE is now green - hardened base, the CVEs collapse" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Checkpoint two - Baseline turns green. The hardened image now feeds the build: DHI, 0 CVEs, SLSA L3, and the CVEs collapse right where we started this segment. 2 of 4 stages provable. The base and the build are both trustworthy now. Next we push toward the CI gate that turns all of this into an enforced boundary.

---

<!-- chrome: false -->

<img src="assets/slide-framework-3.webp" alt="Question 3 of 4 - Gate: is it allowed to pass? Build policies, image signing, admission" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Question three - Gate: is this artifact allowed to pass? The answer is build policies, image signing, and admission - the check in the middle of the pipeline that fails closed, so nothing crosses into production unless it's provable. Evidence and baseline made governance possible; this is where we make it real.

---

<!-- chrome: false -->

<img src="assets/slide-36.webp" alt="Security as code: docker scout policy catalog-service:dhi --exit-code fails the build on fixable criticals/highs, missing attestations, unapproved base, or root user" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: This is security as code. Instead of a human eyeballing a scan report, you define rules that automatically fail the build before anything insecure reaches your registry. One command: docker scout policy --exit-code. That exit code is the whole point - non-zero stops the pipeline dead. The policies: no fixable critical or high CVEs, supply-chain attestations present, no unapproved base images, default non-root user. And it's tunable with an optional policy-config.json.

---

<!-- chrome: false -->

<img src="assets/slide-37.webp" alt="Image signing with Cosign keyless: build with attestations, sign via OIDC, signature lands in the Sigstore transparency log, verify at deploy time" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Passing a policy tells you an image is clean; signing tells you it's authentic - that this exact digest is the one your pipeline produced and nobody swapped it. We use Cosign with keyless signing: build with attestations, sign via OIDC, the signature lands in the Sigstore transparency log, and you verify at deploy time. Keyless is the magic word - a short-lived certificate minted from your OIDC identity, so there's no private key to manage, rotate, or leak. Works with any OCI registry.

---

<!-- chrome: false -->

<img src="assets/slide-38.webp" alt="The secure CI pipeline in four steps on GitHub Actions: checkout, build + attest, policy gate (exit-on policy), then push - the gate sits before push so an unprovable image can't be promoted" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Here's the whole secure pipeline in four steps on GitHub Actions. Checkout. Build and attest - SBOM and provenance generated at build time and bound to the digest. Then the key step, the policy gate: command policy, exit-on policy. That's the dev-to-prod boundary in one line - if any policy fails, the step exits non-zero and push never runs. Only when the gate passes do we reach push. The gate sits before the push on purpose: an unprovable image simply cannot be promoted.

---

<!-- chrome: false -->

<img src="assets/slide-39.webp" alt="The 7 built-in Docker Scout policies: no fixable critical/high CVEs, no high-profile vulnerabilities, no copyleft licenses, no outdated/unapproved base images, supply-chain attestations, default non-root - configurable via JSON or Rego" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: These are the seven built-in Scout policies, and the headline is zero config required. Vulnerability side: no fixable critical or high CVEs, and no high-profile vulnerabilities - Log4Shell, the XZ backdoor, anything in CISA KEV. Hygiene: no copyleft licenses, no outdated base images, no unapproved base images. And the two that tie back to our supply-chain work: supply-chain attestations present, and default non-root user. All configurable via JSON, extensible with custom Rego policies, and it runs fully local - which matters for air-gapped pipelines.

---

<!-- chrome: false -->

<img src="assets/slide-41.webp" alt="Same pipeline, opposite outcomes: with a DHI base the gate passes and the image is pushed; with a standard base CVEs and no SBOM mean the gate fails and push never runs" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Here's why all the earlier hardening pays off at the gate. With a DHI base: no critical or high CVEs, SBOM and provenance present, non-root, up-to-date - the gate passes, the image is pushed. With a standard base: CVEs found, no SBOM, running as root - the gate fails, push never runs. Same pipeline, same policies, opposite outcomes - the only variable is the base the agent built on. The hardened base isn't just hygiene; it's what lets you cleanly clear a fail-closed gate.

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

Note: This is the slide for the ops folks in the room, and it's the heart of why this is a DevOps talk, not just a developer one. The same gate serves two audiences. Developers get fast pass/fail in the PR and a base that's already good, so they fix things at authoring time instead of after a rejection. Platform, SRE and ops own the other side: they set the policy and hardened base once and it applies fleet-wide, and the signed attestations become the audit trail. When the next 2:47 AM commit happens, answering "what shipped and where did it come from" is a query, not a week of forensics. Governance is a platform capability, not a developer chore - and we'll come back to that query at the very end.

---

<!-- chrome: false -->

<img src="assets/slide-journey-3.webp" alt="The journey, checkpoint 3 of 4: Lab 3 done, SIGN, GATE and DEPLOY are now green - signed, gated, promoted to production" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Checkpoint three. Trace the road: the agent develops in a sandbox, builds on a hardened base with zero CVEs, attaches SBOM and provenance, and now SIGNs keylessly, bound to the digest. The CI GATE - no critical CVEs, SBOM present, provenance verified - fails closed at the dev-to-prod boundary, and because our image is provable it passes and gets promoted: DEPLOY goes green. Three of four stages provable. The one box still grey is INVOKE - the running agent and MCP client at the far right. Same discipline at both ends: the agent that builds runs in a box, and the service it becomes runs in a box too. That runtime end is next.

---

<!-- chrome: false -->

<img src="assets/slide-framework-4.webp" alt="Question 4 of 4 - Boundary: what could it reach while it worked? Sandbox runtime - network, filesystem, credentials" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Question four - Boundary: what could the agent reach while it worked? The answer is the sandbox runtime - network, filesystem, and credentials, bounded so the agent can act without an open blast radius. This is the layer that would have stopped both horror stories cold. It closes the loop: same discipline at both ends of the road.

---

<!-- chrome: false -->

<img src="assets/slide-governance.webp" alt="A layered approach to AI governance: Gordon, Agentic Compose & Docker Agent, Docker Model Runner, MCP Toolkit & Gateway, Docker Sandboxes, and Docker Hardened Images as the trusted foundation" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Before we sandbox, here's the shape of the whole thing. Securing the agentic stack isn't one product - it's a layered set of enforceable controls, and the policies travel with the workload whether it runs on a laptop or in the cloud. Gordon gives in-product guidance; Agentic Compose and the Docker Agent give declarative, golden-template orchestration; Docker Model Runner keeps LLM execution local; the MCP Toolkit and Gateway limit agents to the servers you authorize; Docker Sandboxes give each agent an isolated runtime; and underneath it all, Docker Hardened Images are the trusted foundation. The next few slides zoom into that sandbox layer.

---

<!-- chrome: false -->

<img src="assets/slide-10.webp" alt="Agent with a Sandbox: sbx microVM boundary, DHI MCP server, hardened base, zero CVEs" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: This is the clean end-state - the counterpart to the ungoverned baseline from earlier. Everything the agent does now happens inside a sandbox boundary, an sbx microVM with its own daemon, its own network, and the host mounted read-only. Same prompt - "containerize app" - but the agent queries the DHI MCP server, which only serves signed tools, and writes FROM dhi.io/node because it checked the trusted source before writing the line. The result: 0 critical, 0 high, 0 medium, 0 low, 211 packages, SBOM attached, signed, non-root. Same agent, radically different outcome.

---

<!-- chrome: false -->

<img src="assets/slide-sandbox-arch.webp" alt="Sandbox architecture: the agent container runs inside a microVM-based sandbox fed by workspace directories, network policies and secrets; outbound traffic flows through a network proxy" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: Here's the architecture. Everything sits on your host, but the agent runs inside a microVM-based sandbox - a real isolation boundary, not just a container namespace. You feed it three things from outside the box: the workspace directories it's allowed to see, the network policies that govern what it can reach, and the secrets it needs. Outbound traffic doesn't go straight out - it flows through a network proxy that enforces those policies, and the proxy injects credentials so raw keys never enter the VM. The agent gets exactly the access you granted and nothing more.

---

<!-- chrome: false -->

<img src="assets/slide-sandbox-tui.webp" alt="The Sandbox TUI: sandboxes on the left with status and workspace, and the per-sandbox network log on the right showing allowed and blocked hosts" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: This is the sbx TUI - a live view of every sandbox on your machine. On the left, each sandbox with its status, workspace, and resource use, with controls to stop, exec, or remove. On the right, the network log for the selected sandbox: every outbound connection the agent attempted, with a hit count and an allowed-or-blocked status. api.anthropic.com, api.github.com, registry.npmjs.org allowed - a datadog logs endpoint blocked. This is the boundary made observable: you can see exactly what the agent reached for and what the policy stopped. Hold that thought - it becomes the audit trail in our operations section.

---

<!-- chrome: false -->

<img src="assets/slide-mcp-gateway.webp" alt="The agent talks to one gateway, never to servers directly: a sandboxed agent reaches an mcp-gateway via SBX_MCP_URL with local-wiki, GitHub, Notion and DuckDuckGo aggregated behind it" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: The other half of the boundary: the tools. An agent acts on the world through MCP servers, so those tools are part of your supply chain. The agent inside the sandbox never talks to servers directly - it talks to one endpoint, the mcp-gateway, through a single SBX_MCP_URL. Behind it, all your servers are aggregated. The payoff: every tool call flows through one chokepoint. That single point is where policy and audit apply - one place to govern instead of N servers to chase.

---

<!-- chrome: false -->

<img src="assets/slide-cedar-policy.webp" alt="Default-deny allow-list over (server, tool) authored in Cedar: a permit policy allowing exactly get_me on github-official; everything else blocked, evaluated at the gateway on every invoke" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: And here's what the policy looks like - a default-deny allow-list over server-and-tool pairs, authored in Cedar, the open-source authorization engine from AWS. This example permits exactly one tool - get_me on github-official - and by default-deny, every other tool and every other server is blocked. It's evaluated at the gateway on every invoke, using the same engine as your network and filesystem policy - one surface, no bypass. Author once, sync everywhere: a developer can add any server they like, but if org policy doesn't permit its tools, the calls are denied and audited.

---

<!-- chrome: false -->

<img src="assets/slide-journey-4.webp" alt="The journey, checkpoint 4 of 4: Lab 4 done, DEVELOP and INVOKE green, both sandbox boxes solid, the road is provable end to end" width="1600" height="900" loading="eager" decoding="async" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:fill" />

Note: The final checkpoint, 4 of 4 - the whole road is green. On the left, DEVELOPMENT sits inside its own box: the agent develops in an sbx microVM with the host read-only, on a hardened base with 0 CVEs and SLSA L3, buildx attaches SBOM and provenance, and signing binds everything to a digest. In the middle, the CI GATE fails closed. On the right, PRODUCTION is boxed too: the signed image is deployed pinned by digest, and the agent invokes MCP as a signed, read-only client under cap_drop ALL and non-root. Same discipline at both ends. Four of four stages provable - and that's exactly the point where most talks stop. We're going two steps further.

---

<!-- layout: section -->

# Now make it yours - the sbx kit

You've boxed one agent. Next: package that exact boundary as a **kit** your whole org inherits - so the good path is the default path, everywhere.

Note: Everything so far you did by hand for one agent on one laptop. That doesn't scale, and it drifts. The answer is to make the boundary an artifact - a kit - so every developer's agent starts from the same hardened, governed environment without anyone having to remember the flags. This is the platform team's leverage point, and it's the first of the two steps that complete our story.

---

<!-- chrome: false -->

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="Anatomy of an sbx kit" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="100" y="110" font-size="52" font-weight="800" fill="#ffffff">Anatomy of an <tspan fill="#1E9BF0">sbx kit</tspan></text>
  <text x="100" y="158" font-size="26" fill="#9AA6C2">Declarative: a spec.yaml (+ files/). The sbx engine turns it into a sandbox at create time.</text>
  <rect x="100" y="210" width="640" height="470" rx="18" fill="#12203F" stroke="#26365C" stroke-width="2"/>
  <text x="132" y="258" font-size="22" font-weight="800" fill="#1E9BF0" letter-spacing="2">spec.yaml · schemaVersion "2"</text>
  <g font-size="24" fill="#ffffff">
    <text x="132" y="312"><tspan fill="#7FB2E6" font-family="monospace">sandbox:</tspan>  image / build - hardened base</text>
    <text x="132" y="356"><tspan fill="#7FB2E6" font-family="monospace">credentials:</tspan>  apiKey · oauth · sshAgent</text>
    <text x="132" y="400"><tspan fill="#7FB2E6" font-family="monospace">caps.network:</tspan>  deny-all + allowlist</text>
    <text x="132" y="444"><tspan fill="#7FB2E6" font-family="monospace">environment:</tspan>  vars, no raw secrets</text>
    <text x="132" y="488"><tspan fill="#7FB2E6" font-family="monospace">commands:</tspan>  install · startup hooks</text>
    <text x="132" y="532"><tspan fill="#7FB2E6" font-family="monospace">agentContext:</tspan>  guardrails + guidance</text>
    <text x="132" y="576"><tspan fill="#7FB2E6" font-family="monospace">files/:</tspan>  configs baked into the box</text>
  </g>
  <text x="132" y="640" font-size="21" fill="#9AA6C2">One declarative file - diffable, reviewable, versioned.</text>
  <rect x="800" y="210" width="700" height="215" rx="18" fill="#12203F" stroke="#26365C" stroke-width="2"/>
  <text x="832" y="258" font-size="22" font-weight="800" fill="#9AA6C2" letter-spacing="2">TWO KINDS</text>
  <rect x="832" y="286" width="300" height="112" rx="12" fill="#0E2A1E" stroke="#34D399" stroke-width="2"/>
  <text x="852" y="326" font-size="24" font-weight="800" fill="#34D399">kind: sandbox</text>
  <text x="852" y="362" font-size="21" fill="#ffffff">a full agent environment</text>
  <rect x="1168" y="286" width="300" height="112" rx="12" fill="#12325E" stroke="#1E9BF0" stroke-width="2"/>
  <text x="1188" y="326" font-size="24" font-weight="800" fill="#1E9BF0">kind: mixin</text>
  <text x="1188" y="362" font-size="21" fill="#ffffff">a reusable overlay</text>
  <rect x="800" y="445" width="700" height="235" rx="18" fill="#0E1B3A" stroke="#1E9BF0" stroke-width="2"/>
  <text x="832" y="493" font-size="22" font-weight="800" fill="#1E9BF0" letter-spacing="2">COMPOSE</text>
  <text x="832" y="540" font-size="24" fill="#ffffff"><tspan font-family="monospace" fill="#7FB2E6">extends:</tspan>  inherit a base kit</text>
  <text x="832" y="584" font-size="24" fill="#ffffff"><tspan font-family="monospace" fill="#7FB2E6">--kit</tspan>  stack mixins at run time</text>
  <text x="832" y="636" font-size="21" fill="#9AA6C2">Last wins, per section. Hardened base + your org's mixins.</text>
  <text x="100" y="748" font-size="30" font-weight="700" fill="#ffffff">Author once → every agent inherits the same <tspan fill="#34D399" font-weight="800">hardened, governed boundary.</tspan></text>
</svg>

Note: A kit is just a declarative spec.yaml plus an optional files tree - the sbx engine reads it and turns it into a sandbox at create time. Everything we set by hand becomes a field: the sandbox section pins a hardened base image, credentials declares what auth the agent gets - as references, never raw values - caps.network sets deny-all plus an allowlist, commands wires install and startup hooks, agentContext carries the guardrails and guidance, and files bakes configs into the box. Two kinds: a sandbox kit is a full agent environment; a mixin is a reusable overlay you stack on top. And they compose - extends to inherit a base kit, or --kit to stack mixins at run time. So the platform team authors one hardened base kit, and every developer's agent inherits the same boundary. That's the leverage.

---

# Author, pin, and ship the kit

```yaml save-as=catalog-agent/spec.yaml
schemaVersion: "2"
kind: sandbox
sandbox:
  image: dhi.io/node:24-debian13   # hardened base, 0 CVEs
credentials:
  - apiKey: anthropic              # injected by the proxy, never in the VM
caps:
  network:
    default: deny                  # fail closed
    allow: [registry.npmjs.org, api.github.com, dhi.io]
agentContext: |
  Always query the DHI MCP before writing a FROM line.
```

```console
$ sbx kit validate ./catalog-agent
  ✓ schemaVersion "2" · kind: sandbox · network: deny + allowlist
$ sbx kit push oci://registry.example.com/kits/catalog-agent
  ✓ pushed  digest sha256:9f2c…  (immutable, pinned)
$ sbx run claude --kit catalog-agent@sha256:9f2c…
  microVM booted from the org's hardened kit.
```

Kits pin by **digest** - the same discipline as the image. Distribute over OCI or a git commit-SHA; consumers get exactly what you shipped.

Note: Here's the kit in the flesh. The spec on top is the whole boundary as data: a hardened DHI base, credentials declared as references the proxy injects so raw keys never touch the VM, a deny-by-default network with a tight allowlist, and an agentContext that tells the agent to query the DHI MCP before every FROM line - the good path, baked in. Then the lifecycle on the bottom: validate it, push it to an OCI registry or pin it to a git commit SHA, and any developer runs sbx run --kit against that digest. Same pinning discipline as the image itself - immutable, reproducible, no drift. The platform team ships the kit; every agent on every laptop starts governed. That's step one of finishing the story - now let's run what we built.

---

<!-- layout: section -->

# Past the gate - operate it

The gate promotes a signed, attested image. **Now it has to run** - deployed, scaled, and observed, with the same discipline at run time.

Note: Everything up to here got a provable artifact through the gate. But a build that never runs helps no one. This is the day-2 half - the Operational AI story - deploy it, scale it, and keep the boundary and the audit trail alive in production. This is the part the ops and SRE folks own outright.

---

<!-- chrome: false -->

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="Operate agentic services in production" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="100" y="110" font-size="52" font-weight="800" fill="#ffffff">Operational AI: <tspan fill="#1E9BF0">run the governed service</tspan></text>
  <text x="100" y="158" font-size="26" fill="#9AA6C2">The road doesn't stop at the gate - it extends into deploy, scale, and observe.</text>
  <g>
    <rect x="100" y="230" width="330" height="150" rx="16" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="130" y="278" font-size="24" font-weight="800" fill="#1E9BF0">DEPLOY</text>
    <text x="130" y="320" font-size="22" fill="#ffffff">signed image,</text>
    <text x="130" y="350" font-size="22" fill="#ffffff">pinned by digest</text>
    <rect x="470" y="230" width="330" height="150" rx="16" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="500" y="278" font-size="24" font-weight="800" fill="#1E9BF0">RUN</text>
    <text x="500" y="320" font-size="22" fill="#ffffff">read_only · non-root</text>
    <text x="500" y="350" font-size="22" fill="#ffffff">cap_drop ALL</text>
    <rect x="840" y="230" width="330" height="150" rx="16" fill="#12203F" stroke="#26365C" stroke-width="2"/>
    <text x="870" y="278" font-size="24" font-weight="800" fill="#1E9BF0">SCALE</text>
    <text x="870" y="320" font-size="22" fill="#ffffff">Docker / Kubernetes</text>
    <text x="870" y="350" font-size="22" fill="#ffffff">golden templates</text>
    <rect x="1210" y="230" width="290" height="150" rx="16" fill="#0E2A1E" stroke="#34D399" stroke-width="2"/>
    <text x="1240" y="278" font-size="24" font-weight="800" fill="#34D399">OBSERVE</text>
    <text x="1240" y="320" font-size="22" fill="#ffffff">logs · metrics · traces</text>
    <text x="1240" y="350" font-size="22" fill="#ffffff">via Datadog MCP</text>
  </g>
  <g stroke="#3A4A6E" stroke-width="4" fill="#3A4A6E">
    <line x1="432" y1="305" x2="466" y2="305"/><polygon points="466,297 482,305 466,313"/>
    <line x1="802" y1="305" x2="836" y2="305"/><polygon points="836,297 852,305 836,313"/>
    <line x1="1172" y1="305" x2="1206" y2="305"/><polygon points="1206,297 1222,305 1206,313"/>
  </g>
  <rect x="100" y="430" width="1400" height="250" rx="18" fill="#0E1B3A" stroke="#1E9BF0" stroke-width="2"/>
  <text x="132" y="478" font-size="22" font-weight="800" fill="#1E9BF0" letter-spacing="2">RUNTIME BOUNDARY = THE SANDBOX DISCIPLINE, IN PROD</text>
  <g font-size="24" fill="#ffffff">
    <text x="132" y="530">· <tspan font-weight="700">read_only rootfs · cap_drop ALL · non-root</tspan> - least privilege at run time</text>
    <text x="132" y="574">· image <tspan font-weight="700">pinned by digest</tspan>, verified at admission - only what the gate signed runs</text>
    <text x="132" y="618">· MCP called as a <tspan font-weight="700">signed, read-only client</tspan> through the governed gateway</text>
    <text x="132" y="662">· scale on Docker or Kubernetes from <tspan font-weight="700">golden, declarative templates</tspan></text>
  </g>
  <text x="100" y="752" font-size="30" font-weight="700" fill="#ffffff">The agent that BUILDS runs in a box - the service it BECOMES <tspan fill="#34D399" font-weight="800">runs in a box too.</tspan></text>
</svg>

Note: This is Operational AI in one frame - the same discipline, now at run time. Deploy the signed image, pinned by digest and verified at admission so only what the gate signed ever runs. Run it under least privilege: read-only rootfs, all capabilities dropped, non-root - the exact posture we gave the sandbox, now in production. The service invokes MCP as a signed, read-only client through the same governed gateway. And you scale it on Docker or Kubernetes from golden, declarative templates, not hand-rolled YAML. The line at the bottom is the whole talk: the agent that builds runs in a box, and the service it becomes runs in a box too - least privilege on both ends of the road.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="Agents at ops time - the counterpart to DHI MCP" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="100" y="104" font-size="50" font-weight="800" fill="#ffffff">Agents at ops time - <tspan fill="#1E9BF0">the counterpart to DHI MCP</tspan></text>
  <text x="100" y="150" font-size="25" fill="#9AA6C2">DHI MCP is a developer tool - it governs the build. Observability MCP governs the fire drill.</text>
  <rect x="100" y="196" width="640" height="336" rx="18" fill="#0E2A1E" stroke="#34D399" stroke-width="2"/>
  <text x="132" y="244" font-size="22" font-weight="800" fill="#34D399" letter-spacing="2">DEV TIME · PREVENT</text>
  <text x="132" y="300" font-size="30" font-weight="700" fill="#ffffff">Agent + DHI MCP</text>
  <text x="132" y="350" font-size="23" fill="#C8D3F5">"what's the hardened base for Node,</text>
  <text x="132" y="382" font-size="23" fill="#C8D3F5">and what CVEs does it carry?"</text>
  <text x="132" y="448" font-size="22" fill="#9AA6C2">Queries the trusted catalog before it</text>
  <text x="132" y="480" font-size="22" fill="#9AA6C2">writes a single FROM line.</text>
  <rect x="860" y="196" width="640" height="336" rx="18" fill="#12325E" stroke="#1E9BF0" stroke-width="2"/>
  <text x="892" y="244" font-size="22" font-weight="800" fill="#1E9BF0" letter-spacing="2">OPS TIME · DIAGNOSE</text>
  <text x="892" y="300" font-size="30" font-weight="700" fill="#ffffff">Agent + Datadog MCP</text>
  <text x="892" y="344" font-size="21" fill="#7FB2E6">Logs · Metrics · APM traces · Monitors</text>
  <text x="892" y="402" font-size="23" fill="#C8D3F5">"why did catalog-service p99 spike</text>
  <text x="892" y="434" font-size="23" fill="#C8D3F5">after the 02:47 deploy? what's in the logs?"</text>
  <text x="892" y="500" font-size="22" fill="#9AA6C2">Reads the running system to find the cause.</text>
  <rect x="100" y="566" width="1400" height="238" rx="18" fill="#0E1B3A" stroke="#1E9BF0" stroke-width="2"/>
  <text x="132" y="612" font-size="22" font-weight="800" fill="#1E9BF0" letter-spacing="2">SAME GOVERNANCE, BOTH ENDS</text>
  <g font-size="24" fill="#ffffff">
    <text x="132" y="662">· sandboxed · one MCP gateway · Cedar scoped to <tspan font-weight="700">READ-ONLY</tspan> query tools · every call audited</text>
    <text x="132" y="706">· the agent can <tspan font-weight="700">diagnose</tspan> - it cannot mutate a dashboard, silence an alert, or deploy</text>
    <text x="132" y="754">· to <tspan font-weight="700" fill="#F0A84A">act</tspan> on the fix it goes back through the <tspan font-weight="700">CI gate</tspan> - the same bar as any change</text>
  </g>
</svg>

Note: DHI MCP was a developer tool - it governs the build, before FROM. Its ops-time counterpart is observability MCP. When catalog-service goes slow at 3am, you don't want a human grepping logs - you want an agent that can read the logs, metrics and traces and tell you the cause. Here that's Datadog - Logs, Metrics and APM traces, with Monitors and Watchdog - wired in as a kit. But it's the exact same boundary as the build agent: sandboxed, through the one gateway, Cedar-scoped to read-only query tools, every call audited. It can diagnose - it cannot silence an alert or deploy a change. And when it proposes a fix, that fix goes back through the same CI gate from Move 3. Diagnose freely; act only through the gate.

---

# Day-2 governance: audit the connection, not the payload

```console
$ # 3:00 AM: catalog-service is slow. The ops agent already looked - now you audit it:
$ sbx audit log --since 02:00
  02:47  invokeTool  datadog__query_metrics       allow
  02:51  invokeTool  datadog__search_logs         allow
  02:52  invokeTool  datadog__create_incident     DENY  (policy: read-only)
  02:53  network     paste.example.com            DENY  (not in allowlist)

$ # and provenance for the fix it proposed - the same query as the 02:47 build:
$ docker scout attest get --predicate-type slsa --verify catalog-service@sha256:9f2c…
  ✓ signed · builder docker.com/dhi/builder · source git+github.com/acme/catalog@<sha>
```

**The audit shows the _connection_, not the _payload_** - that `search_logs` was *called and allowed*, never the log lines it returned. It's a governance/forensics trail (who reached what, allowed or denied), **not** DLP. Policy is authored once in **Docker Hub AI Governance**, synced at `docker login`, and **fails closed**.

And it doesn't stay in Docker: that same decision stream forwards **server-side from Docker Cloud** to your SIEM - Splunk, Datadog, Dynatrace, or any HTTPS endpoint - so the evidence lands where your SOC already lives, with nothing for the agent to disable. `source:docker-audit @decision:AUDIT_DECISION_DENY`

Note: The payoff, and the callback to where we opened. When the next 2:47 AM happens, you don't run a forensics project - you ask. The sbx audit log shows every action the ops agent took: the Datadog queries it ran, allowed; the create_incident it tried, denied by the read-only policy; the paste-site it reached for, denied by the allowlist. And docker scout attest verifies the fix's provenance - builder and source commit, signed. But be honest about what the audit is: it records the connection, not the payload - that search_logs was called and allowed, not which log lines came back. It answers "what did the agent reach, and was it allowed?" - a governance trail, not content inspection or DLP. For request and response bodies you need app-level observability, a different layer. Same as always: policy authored once in Docker Hub, synced at login, fails closed, can't be overridden locally. And one line to land for the SOC: this stream doesn't stay in Docker - it forwards server-side from Docker Cloud to whatever SIEM your security team already lives in, Splunk, Datadog or Dynatrace, so the deny you just saw is searchable next to everything else they watch, and there's nothing on the agent's side to switch off. Who approved that build - and who touched it at 3am? Now both are a query.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="The catalog agent's allow and deny decisions in Datadog" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="80" y="88" font-size="44" font-weight="800" fill="#ffffff">The catalog agent, on the record - <tspan fill="#A78BFA">in Datadog.</tspan></text>
  <text x="80" y="130" font-size="22" fill="#9AA6C2">One org policy allowlists what the build agent needs, denies the rest by default - and streams every decision to the SOC's Datadog.</text>
  <rect x="80" y="170" width="1440" height="40" rx="8" fill="#12203F"/>
  <g font-size="15" font-weight="800" fill="#7F8DB0" letter-spacing="1.5">
    <text x="150" y="196">AGENT ACTION</text>
    <text x="650" y="196">GOVERNANCE</text>
    <text x="940" y="196">WHY IT MATTERS</text>
  </g>
  <g>
    <rect x="80" y="214" width="1440" height="70" fill="#0E2415"/>
    <circle cx="118" cy="249" r="13" fill="#34D399"/>
    <path d="M112,249 l4,5 l8,-10" stroke="#0B1533" stroke-width="2.6" fill="none" stroke-linecap="round" stroke-linejoin="round"/>
    <text x="150" y="256" font-size="19" fill="#ffffff" font-family="Menlo, monospace">registry.npmjs.org, pypi.org</text>
    <text x="650" y="256" font-size="19"><tspan font-weight="800" fill="#34D399">ALLOW</tspan><tspan fill="#6E86B8"> · allowlist</tspan></text>
    <text x="940" y="256" font-size="18" fill="#C8D3F5">Install catalog service deps</text>
  </g>
  <g>
    <rect x="80" y="288" width="1440" height="70" fill="#0E2415"/>
    <circle cx="118" cy="323" r="13" fill="#34D399"/>
    <path d="M112,323 l4,5 l8,-10" stroke="#0B1533" stroke-width="2.6" fill="none" stroke-linecap="round" stroke-linejoin="round"/>
    <text x="150" y="330" font-size="19" fill="#ffffff" font-family="Menlo, monospace">api.anthropic.com</text>
    <text x="650" y="330" font-size="19"><tspan font-weight="800" fill="#34D399">ALLOW</tspan><tspan fill="#6E86B8"> · allowlist</tspan></text>
    <text x="940" y="330" font-size="18" fill="#C8D3F5">LLM writes product descriptions</text>
  </g>
  <g>
    <rect x="80" y="362" width="1440" height="70" fill="#24121A"/>
    <circle cx="118" cy="397" r="13" fill="#F0533F"/>
    <line x1="112" y1="391" x2="124" y2="403" stroke="#0B1533" stroke-width="2.6" stroke-linecap="round"/>
    <line x1="124" y1="391" x2="112" y2="403" stroke="#0B1533" stroke-width="2.6" stroke-linecap="round"/>
    <text x="150" y="404" font-size="19" fill="#ffffff" font-family="Menlo, monospace">catalog-db.internal:5432</text>
    <text x="650" y="404" font-size="19"><tspan font-weight="800" fill="#F0533F">DENY</tspan><tspan fill="#6E86B8"> · default-deny</tspan></text>
    <text x="940" y="404" font-size="18" fill="#C8D3F5">Direct DB access - <tspan font-weight="700" fill="#ffffff">data-exfil prevented</tspan></text>
  </g>
  <g>
    <rect x="80" y="436" width="1440" height="70" fill="#24121A"/>
    <circle cx="118" cy="471" r="13" fill="#F0533F"/>
    <line x1="112" y1="465" x2="124" y2="477" stroke="#0B1533" stroke-width="2.6" stroke-linecap="round"/>
    <line x1="124" y1="465" x2="112" y2="477" stroke="#0B1533" stroke-width="2.6" stroke-linecap="round"/>
    <text x="150" y="478" font-size="19" fill="#ffffff" font-family="Menlo, monospace">api.stripe.com:443</text>
    <text x="650" y="478" font-size="19"><tspan font-weight="800" fill="#F0533F">DENY</tspan><tspan fill="#6E86B8"> · default-deny</tspan></text>
    <text x="940" y="478" font-size="18" fill="#C8D3F5">Payments API - <tspan font-weight="700" fill="#ffffff">scope creep blocked</tspan></text>
  </g>
  <g>
    <rect x="80" y="510" width="1440" height="70" fill="#24121A"/>
    <circle cx="118" cy="545" r="13" fill="#F0533F"/>
    <line x1="112" y1="539" x2="124" y2="551" stroke="#0B1533" stroke-width="2.6" stroke-linecap="round"/>
    <line x1="124" y1="539" x2="112" y2="551" stroke="#0B1533" stroke-width="2.6" stroke-linecap="round"/>
    <text x="150" y="552" font-size="19" fill="#ffffff" font-family="Menlo, monospace">images.unsplash.com:443</text>
    <text x="650" y="552" font-size="19"><tspan font-weight="800" fill="#F0533F">DENY</tspan><tspan fill="#6E86B8"> · default-deny</tspan></text>
    <text x="940" y="552" font-size="18" fill="#C8D3F5">Untrusted CDN - <tspan font-weight="700" fill="#ffffff">supply-chain risk</tspan></text>
  </g>
  <rect x="80" y="606" width="1160" height="46" rx="8" fill="#111A30" stroke="#2A3A5C" stroke-width="1.5"/>
  <text x="104" y="635" font-family="Menlo, monospace" font-size="18" fill="#7FB2E6">source:docker-audit @org_name:whalecollab @decision:<tspan fill="#F0533F">AUDIT_DECISION_DENY</tspan></text>
  <text x="1270" y="635" font-size="18" font-weight="800" fill="#A78BFA">Datadog › Logs › Explorer</text>
  <text x="80" y="708" font-size="23" font-weight="700" fill="#ffffff">It reached for your database and Stripe - governance denied both automatically, <tspan fill="#34D399">no human in the loop.</tspan></text>
  <text x="80" y="742" font-size="23" fill="#C8D3F5">The SOC saw it in Datadog within seconds - <tspan font-weight="700" fill="#ffffff">no source code, no prompt</tspan> in the record.</text>
  <text x="80" y="792" font-size="20" font-style="italic" fill="#7F8DB0">Governance stops the connection; the kit stops the intent - together, defense in depth.</text>
</svg>

Note: This is the demo, made concrete - Datadog and Docker AI Governance. The whalecollab org gives a coding agent one job: build and maintain the Product Catalog service. Governance allowlists exactly what that job needs - npm and pypi to install dependencies, api.anthropic.com so the LLM can write product copy - and denies everything else by default. So when the agent reaches for the production database on 5432, the Stripe payments API, or an unvetted image CDN - whether nudged by a prompt injection, a poisoned dependency, or just an over-eager plan - each one is denied at the connection, no human in the loop. And every decision streams to the security team's Datadog: they filter source:docker-audit, org whalecollab, decision DENY, and watch the denials land live - without ever seeing a line of source code or a single prompt. Governance is the platform guardrail every sandbox inherits and the SOC sees; the kit is how a team ships that same agent with an extra in-agent safety layer. Governance stops the connection, the kit stops the intent - defense in depth.

---

<!-- chrome: false -->

<svg viewBox="0 0 1600 900" width="100%" height="100%" role="img" aria-label="Your security framework in seven steps" style="position:absolute;inset:0;width:100%;height:100%;max-width:none;max-height:none;object-fit:contain;background:#0B1533" font-family="Arial, Helvetica, sans-serif">
  <rect width="1600" height="900" fill="#0B1533"/>
  <text x="100" y="120" font-size="52" font-weight="800" fill="#ffffff">Your security framework - <tspan fill="#1E9BF0">seven steps</tspan></text>
  <g>
    <g transform="translate(100,180)"><rect width="1400" height="82" rx="12" fill="#12203F"/><rect width="7" height="82" rx="3" fill="#1E9BF0"/><text x="48" y="41" dominant-baseline="central" font-size="34" font-weight="800" fill="#5B8CFF">1</text><text x="120" y="41" dominant-baseline="central" font-size="27" font-weight="700" fill="#ffffff">Know what's in your images</text><text x="760" y="41" dominant-baseline="central" font-size="24" fill="#9AA6C2">SBOM + VEX</text></g>
    <g transform="translate(100,272)"><rect width="1400" height="82" rx="12" fill="#12203F"/><rect width="7" height="82" rx="3" fill="#1E9BF0"/><text x="48" y="41" dominant-baseline="central" font-size="34" font-weight="800" fill="#5B8CFF">2</text><text x="120" y="41" dominant-baseline="central" font-size="27" font-weight="700" fill="#ffffff">Verify where they came from</text><text x="760" y="41" dominant-baseline="central" font-size="24" fill="#9AA6C2">SLSA provenance + signing</text></g>
    <g transform="translate(100,364)"><rect width="1400" height="82" rx="12" fill="#12203F"/><rect width="7" height="82" rx="3" fill="#1E9BF0"/><text x="48" y="41" dominant-baseline="central" font-size="34" font-weight="800" fill="#5B8CFF">3</text><text x="120" y="41" dominant-baseline="central" font-size="27" font-weight="700" fill="#ffffff">Start from a trusted base</text><text x="760" y="41" dominant-baseline="central" font-size="24" fill="#9AA6C2">Docker Hardened Images</text></g>
    <g transform="translate(100,456)"><rect width="1400" height="82" rx="12" fill="#12203F"/><rect width="7" height="82" rx="3" fill="#1E9BF0"/><text x="48" y="41" dominant-baseline="central" font-size="34" font-weight="800" fill="#5B8CFF">4</text><text x="120" y="41" dominant-baseline="central" font-size="27" font-weight="700" fill="#ffffff">Enforce at the pipeline</text><text x="760" y="41" dominant-baseline="central" font-size="24" fill="#9AA6C2">Docker Scout build policies · fail closed</text></g>
    <g transform="translate(100,548)"><rect width="1400" height="82" rx="12" fill="#12203F"/><rect width="7" height="82" rx="3" fill="#1E9BF0"/><text x="48" y="41" dominant-baseline="central" font-size="34" font-weight="800" fill="#5B8CFF">5</text><text x="120" y="41" dominant-baseline="central" font-size="27" font-weight="700" fill="#ffffff">Isolate your agents</text><text x="760" y="41" dominant-baseline="central" font-size="24" fill="#9AA6C2">sbx microVM + MCP gateway</text></g>
    <g transform="translate(100,640)"><rect width="1400" height="82" rx="12" fill="#0E2A1E"/><rect width="7" height="82" rx="3" fill="#34D399"/><text x="48" y="41" dominant-baseline="central" font-size="34" font-weight="800" fill="#34D399">6</text><text x="120" y="41" dominant-baseline="central" font-size="27" font-weight="700" fill="#ffffff">Package the boundary</text><text x="760" y="41" dominant-baseline="central" font-size="24" fill="#9AA6C2">ship a hardened sbx kit, pinned by digest</text></g>
    <g transform="translate(100,732)"><rect width="1400" height="82" rx="12" fill="#0E2A1E"/><rect width="7" height="82" rx="3" fill="#34D399"/><text x="48" y="41" dominant-baseline="central" font-size="34" font-weight="800" fill="#34D399">7</text><text x="120" y="41" dominant-baseline="central" font-size="27" font-weight="700" fill="#ffffff">Operate &amp; govern in prod</text><text x="760" y="41" dominant-baseline="central" font-size="24" fill="#9AA6C2">runtime hardening + audit + policy sync</text></g>
  </g>
</svg>

Note: If you take one slide home, take this one - the framework, now seven steps. One: know what's in your images with SBOM and VEX. Two: verify where they came from with SLSA and signing. Three: start from a trusted base - DHI. Four: enforce at the pipeline with policies that fail closed. Five: isolate your agents with the sandbox and the MCP gateway. Those five were the labs. The two in green are what we added today: six, package that boundary as a hardened sbx kit pinned by digest, so it scales across the org without drift; and seven, operate and govern it in production - runtime hardening, an audit trail, and policy synced from the hub. Build it, box it, ship the kit, run it. That's the whole playbook.

---

<!-- layout: default -->

# Or watch it live

```bash terminal-id=demo
docker scout policy catalog-service:baseline
```

::terminal{id=demo height=300}

Note: This is the simulator from the hands-on lab - the exact commands you just saw. Run the policy on the 2:47 AM image and watch it fail. In the lab you take that same image and walk it all the way to green: SBOM, hardened base, signature, the CI gate - and then sandbox the agent and wire the MCP gateway. Let me point you there.

---

<!--
layout: title
byline: "agentic.dockerworkshop.com"
-->

# Do the lab

Take the 2:47 AM image and drive it to a signed, attested, policy-gated build - then box the agent. In your browser, nothing to install.

```console
$ open agentic.dockerworkshop.com
```

Note: Right next to this deck is a hands-on lab that runs entirely in your browser - no install. You take the exact image the agent shipped and walk it through every move here: SBOM, hardened base, signature, CI gate, then the sandbox and the gateway. Bookmark it.

---

<!--
layout: section
chrome: false
-->

# Who approved that build?

Now you can answer - and prove it.

**Thank you. Questions?**

Note: Who approved that build? With evidence, a governed base, an enforceable gate, a boxed agent, a kit that spreads that boundary across the org, and an operations story that keeps it alive in production - the answer is: the same policy that approves every build, applied to the agent exactly as to a human, with a trail to prove it. Thank you - questions?

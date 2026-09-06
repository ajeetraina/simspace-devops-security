# Who Approved That Build?

**Docker AI Governance for the Agentic DevOps Pipeline** — a conference talk deck plus a
hands-on, fully in-browser lab, both built on
[Simspace](https://github.com/dockersamples/simspace). Everything in the terminal is
simulated — no real Docker, backend, or network — so it runs the same for everyone, with
nothing to install.

**The premise:** at 02:47 AM a commit lands. The author is `svc-build-agent`, not a person.
It bumped a base image, regenerated the Dockerfile, and triggered a build — and CI approved
it, all checks green. No human did. This talk lays out a governance framework that keeps AI
agents productive *and* accountable, in three moves:

1. **Prove it** — SBOM, VEX and SLSA provenance as the evidence layer: *what's in this, and
   where did it come from?*
2. **Default it** — Docker Hardened Images as a governed, minimal baseline the agent starts
   from by default.
3. **Enforce it** — signing + build policies that turn CI into a control point where
   unverified or agent-introduced artifacts simply can't pass.

## What's in this repo

Two Simspace labs under [`lab/`](lab/), surfaced as one landing page (deck first, then the
hands-on lab):

| Path | What it is |
|------|------------|
| [`lab/who-approved-that-build-slides/`](lab/who-approved-that-build-slides/) | The **talk deck** — native markdown slides (`deck.md`), diffable and edited in place. Wired to the lab's simulator so live terminal demos can be spliced in. |
| [`lab/who-approved-that-build/`](lab/who-approved-that-build/) | The **hands-on lab** — `labspace.yaml` (config + seeded virtual filesystem), `simulator.yaml` (command behaviour), and one markdown file per section. |

The deck is authored text — no image assets, so it stays version-controlled and easy to
edit. The lab reuses a proven SBOM → DHI → signing → CI-gate simulator; both are loaded at
runtime by a prebuilt image, so there's no build step for content.

## Author locally

You only need Docker.

```bash
docker compose up dev              # live preview at http://localhost:5173
docker compose run --rm validate   # lint the labs (fails on errors)
```

Edit files under `lab/` and refresh the browser to see changes. Pin the toolchain for
reproducibility:

```bash
export SIMSPACE_AUTHORING_IMAGE=dockersamples/simspace-authoring:1
```

## Deploy

**GitHub Pages (default):** enable Pages (Settings → Pages → Source: "GitHub Actions"),
then push to `main`. [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml)
validates the labs and publishes them; [`.github/workflows/validate.yml`](.github/workflows/validate.yml)
validates pull requests first.

**As a container:** [`Dockerfile`](Dockerfile) bases on the runtime image and swaps in this
repo's `lab/` directory.

```bash
docker build -t who-approved-that-build .
docker run --rm -p 8080:80 who-approved-that-build   # http://localhost:8080
```

## Authoring with an AI agent

This repo is set up for agent authoring. In Claude Code, an `authoring-lab` skill (under
`.claude/`) knows the workflow, `docker compose` / `validate-lab` are pre-allowed, and a
hook auto-validates after every edit under `lab/`. See [`AGENTS.md`](AGENTS.md) for the
authoring cheat-sheet and the [Simspace specs](https://github.com/dockersamples/simspace/tree/main/spec)
for the full `simulator.yaml` / `labspace.yaml` / `slidedeck.md` reference.

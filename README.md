# jenkins-ci-demo

**Status: planning only — nothing built yet.** See [`docs/PLAN.md`](docs/PLAN.md)
for the full plan, the decisions already made, and what's still open.

## What this is

A small project built specifically to close a real gap found by grepping my
own job-application tracker: Jenkins was the most-requested CI/CD tool with
zero representation in my portfolio (every other project uses GitHub
Actions). The app itself is intentionally trivial — a page that displays its
own build provenance (commit hash, Jenkins build number, deploy time) — so
the Jenkins pipeline itself is what's actually being demonstrated.

Jenkins will run self-hosted on a Raspberry Pi 500, separate from the
Raspberry Pi 400 that hosts the rest of my self-hosted stack
([`homelab-pi`](https://github.com/genkuroo/homelab-pi)) — mirroring a real
pattern where CI infrastructure lives apart from what it deploys to.

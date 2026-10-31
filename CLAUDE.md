# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

Not built yet — this is a planning-stage repo. See `docs/PLAN.md` for the
full plan: why this project exists, the decisions already made, and what's
still open before building starts.

In one line: a small demo app whose sole job is to display its own build
provenance, built specifically to prove out a self-hosted **Jenkins**
pipeline — a real, frequently-requested skill with no other representation
in the workspace (everything else uses GitHub Actions).

## Before writing any code here

Read `docs/PLAN.md` in full first — it has open decisions (Pi 500 Tailscale
status, where the app actually runs) that need resolving before the build
order in that doc makes sense to follow.

## Conventions (inherited from `~/Code/CLAUDE.md`)

- **No AI attribution in commits/PRs.**
- **Sync the Obsidian vault note** (`~/Notes/MyVault/Projects/jenkins-ci-demo.md`)
  after every commit/push.

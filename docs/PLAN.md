# Plan: a Jenkins pipeline, to close a real portfolio gap

**Status: planning only — paused 2026-11-02 (Ethan OOT, no access to the Pi 500).**
Nothing built yet. This doc captures everything decided in conversation so it
can be picked back up without re-deriving it.

## Why this project exists

Grepped `~/Code/job-hunt/applications.md` for skill mentions across every real
job posting Ethan's applied to. Jenkins came up **10 times** — tied with
Ansible before `pi-ansible` closed that gap — and it's the single
most-requested tool with zero representation in the portfolio. Every existing
CI/CD project (`dnd-campaign-tracker`, `url-shortener-aws`, `url-shortener-k8s`)
uses GitHub Actions; none demonstrate running/operating a self-hosted CI
server, which is a different (and still very commonly asked-for) skill.

Other gaps the same grep turned up, ranked by mentions, not yet acted on:
Datadog (6), Splunk (3), GitLab CI (3), HashiCorp Vault (1). Jenkins was
picked first because it's the highest-count gap and the most natural fit for
more self-hosted infra (consistent with the rest of the workspace's
philosophy).

## The app (what Jenkins will actually build/deploy)

A small, dedicated demo app — not a retrofit of an existing one, so the
Jenkins pipeline is the star, not competing for attention with app logic.

**Single page that displays its own build provenance:** git commit hash,
Jenkins build number, and deploy timestamp. This is the visible proof the
pipeline worked each run — same principle as this workspace's usual "leave
demo data visible" habit, just applied to a CI pipeline instead of app data.
Keep the app itself trivial (a single Flask route is plenty) — all the real
engineering here is the pipeline, not the app.

## Hosting decision: Jenkins on the Pi 500, deploying to the Pi 400

**Decided:** Jenkins runs on the **Pi 500** (a separate physical device from
`homelab-pi`'s Pi 400), not bundled into the `homelab-pi` Docker Compose
stack. Reasoning discussed:

- Avoids Jenkins (a genuinely heavy JVM app) competing with `dnd`/`stock`/
  `fitness`/`sleeper-bot` for the Pi 400's limited RAM.
- Mirrors a real, common enterprise pattern: the CI server lives on separate
  infrastructure from what it deploys to — same shape as GitHub's hosted
  runners reaching the Pi 400 over Tailscale for the `dnd` pipelines, just
  self-hosted instead of GitHub-hosted.
- Jenkins would then deploy over Tailscale via a **restricted, forced-command
  SSH key** to the Pi 400 — the exact same pattern already proven in
  `dnd-campaign-tracker`'s `deploy-staging.sh`/`deploy-prod.sh` (see that
  repo's `docs/CICD_PLAN.md`). Nothing new to invent there, just reapply it
  with Jenkins as the caller instead of a GitHub Actions runner.

**Open / blocking, confirm when back in SD:**
- [ ] **Is the Pi 500 actually on the same Tailscale network as the Pi 400?**
      Checked `tailscale status` from the Mac on 2026-11-02 — only `homelab`
      (Pi 400) and an offline `desktop-8hcgd6u` showed up. No Pi 500 visible.
      Ethan believes it should be joined already, but this wasn't confirmed
      directly against the Pi 500 (no physical/network access while OOT).
      **First thing to check on return:** SSH into the Pi 500 directly and
      run `tailscale status` / `tailscale ip` there.
- [ ] If the Pi 500 isn't joined yet, join it (`tailscale up`) before
      anything else — everything downstream depends on Jenkins (Pi 500)
      being able to reach the Pi 400 over the tailnet.
- [ ] Where does the small demo app itself actually *run*? Two options,
      not yet decided:
      - **(a)** Jenkins (Pi 500) deploys the container to the Pi 400,
        alongside the other `homelab-pi` services — more realistic
        (mirrors a real "CI reaches over to a separate deploy target"
        setup), but adds one more service to `homelab-pi`'s compose file
        and Caddy config if it needs to be reachable at all.
      - **(b)** Jenkins builds and runs the container locally, on the
        Pi 500 itself — simpler, no remote-deploy step, but loses the
        "CI server reaches a separate target" lesson that's the main
        reason Pi 500 was chosen over just bundling into `homelab-pi`.
      **Leaning (a)** since it's the more realistic pattern and reuses the
      forced-command SSH approach already proven — but worth deciding
      deliberately, not by default.

## Trigger: Poll SCM, not a webhook

**Decided, and why:** a GitHub webhook means GitHub's servers push an HTTP
request straight at Jenkins the instant a commit lands — fast, but it
requires Jenkins to have a publicly reachable address. Jenkins deliberately
is **not** going to be public (an internet-facing Jenkins admin UI is a real,
common attack target — arbitrary build scripts run with real permissions).
Keeping it Tailscale-only means GitHub's servers physically can't reach it.

Instead: Jenkins' **Poll SCM** feature — Jenkins itself periodically asks
GitHub "anything new on this branch?" (e.g. every 2 minutes) and builds if
so. All outbound, no inbound exposure needed. The only cost is a small delay
(up to the poll interval) between push and build start — a non-issue for a
demo project, and worth being able to explain the tradeoff in an interview
(there's a middle path real companies use — a Cloudflare-Tunnel-style narrow
webhook endpoint, same tool `homelab-pi` already uses for `dnd` — but that's
more plumbing than this project needs).

## Security note worth remembering

Jenkins will need `docker build`/`docker compose` access on whichever host
actually builds/runs the image. The common way to give it that is mounting
the host's Docker socket into the Jenkins container — but that's effectively
root-equivalent access to that host (anything that can run arbitrary `docker`
commands can escape to the host). This is **standard practice**, not a
mistake, but it's a real tradeoff worth being able to articulate, not a free
lunch. Exact approach (socket mount vs. Jenkins SSHing out to run `docker`
commands remotely, same as GitHub Actions already does for `dnd`) should
follow from whichever hosting option ((a) or (b) above) gets chosen.

## Rough build order, once the Pi 500 question is resolved

1. Confirm/join the Pi 500 to the tailnet.
2. Decide (a) vs (b) above for where the app actually runs.
3. Create the actual app (one Flask route, build-info page) + Dockerfile.
4. Install Jenkins on the Pi 500 (Docker container, Tailscale-only —
   `127.0.0.1` binding or tailnet-interface binding, never `0.0.0.0`).
5. Write the `Jenkinsfile`: checkout → test → build image → deploy (either
   locally on Pi 500, or via the forced-command SSH key to the Pi 400).
6. Set up Poll SCM against this repo's `main` branch.
7. Prove it end to end: push a real small change, watch Jenkins pick it up
   and deploy it, confirm the build-info page reflects the new build.
8. Write up the project (README, CLAUDE.md) same as every other repo in
   this workspace, and sync the Obsidian vault note.

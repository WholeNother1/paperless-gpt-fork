# rooth-patched fork notes

This is `WholeNother1/paperless-gpt-fork`, a private fork of [`icereed/paperless-gpt`](https://github.com/icereed/paperless-gpt) used to apply local patches and rebuild the container image consumed by the per-tenant `paperless-gpt` services on Spark.

## Auto-push lane

This repo is in the rooth `BOUNDARIES.md` auto-push lane allowlist (added 2026-05-06). The flow (single-motion deploy, updated 2026-05-07):

```
Desktop (this checkout) -- git push spark main --> Spark bare (/srv/spark-source-mirrors/paperless-gpt-fork.git)
                                                              |
                                                              | post-receive hook (background)
                                                              v
                                         /home/jeremiah/paperless-gpt-fork/  (working tree)
                                                              |
                                                              | 1. docker build -t paperless-gpt:rooth-patched .
                                                              | 2. recreate jeremiah paperless-gpt container (CANARY)
                                                              | 3. sleep 30s, check canary still Up
                                                              | 4. if canary OK: recreate cis-paperless-proof + household
                                                              |    if canary FAIL: skip cis + household; operator investigates
                                                              v
                                              all 3 tenants on paperless-gpt:rooth-patched
```

All three tenants (jeremiah, cis-paperless-proof, household) pin `image: paperless-gpt:rooth-patched` in their compose files, so a single `git push spark main` rebuilds the image and deploys to all three. No per-tenant operator step required for routine pushes.

Build status / progress: `tail -f /var/log/paperless-gpt-fork-build.log` on Spark — shows build output, then `=== rollout start ... ===` block, then per-container recreate output, then `=== rollout end ... ===`.

Rollback paths:
- **Bad commit but build succeeded:** `git revert <bad>` on Desktop, `git push spark main` — same hook re-deploys the prior code to all three tenants.
- **Want one tenant on a different image temporarily:** edit that tenant's compose file (`image:` line), re-pin to whatever you want, `docker compose -f <compose> up -d --force-recreate paperless-gpt`. Snapshots from the 2026-05-07 rollout sit at `<compose>.pre-rooth-patched-rollout-20260507-003229` for cis-paperless-proof and household if you need the upstream `icereed/paperless-gpt:latest` pin back.
- **Hook itself misbehaves:** prior hook is at `/srv/spark-source-mirrors/paperless-gpt-fork.git/hooks/post-receive.pre-canary-rollout-20260507-003438`; copy it back over the live `post-receive`.

## What's patched

(Initial state: clean upstream HEAD; no patches yet. The setup itself is the deliverable.)

Future patches will land here as commits with `fix:` / `patch:` prefixes documenting the intent. See `notes/` for in-flight investigations.

## Branch policy

- `main` is the deployable branch — what Spark builds from.
- `upstream/main` is the upstream tracking branch — pull periodically and rebase patches on top, or merge as appropriate.
- Topic branches for in-flight work; merge to `main` only after testing.

## Off-Spark durable backup

GitHub: `https://github.com/WholeNother1/paperless-gpt-fork` (private). Push to both `origin` and `spark` for each release-worthy commit.

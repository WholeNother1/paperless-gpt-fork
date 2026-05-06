# rooth-patched fork notes

This is `WholeNother1/paperless-gpt-fork`, a private fork of [`icereed/paperless-gpt`](https://github.com/icereed/paperless-gpt) used to apply local patches and rebuild the container image consumed by the per-tenant `paperless-gpt` services on Spark.

## Auto-push lane

This repo is in the rooth `BOUNDARIES.md` auto-push lane allowlist (added 2026-05-06). The flow:

```
Desktop (this checkout) -- git push spark main --> Spark bare (/srv/spark-source-mirrors/paperless-gpt-fork.git)
                                                              |
                                                              | post-receive hook
                                                              v
                                         /home/jeremiah/paperless-gpt-fork/  (working tree)
                                                              |
                                                              | docker build -t paperless-gpt:rooth-patched .
                                                              v
                                              local image (consumed by tenant compose)
```

Build status / progress: `tail -f /var/log/paperless-gpt-fork-build.log` on Spark.

## What's patched

(Initial state: clean upstream HEAD; no patches yet. The setup itself is the deliverable.)

Future patches will land here as commits with `fix:` / `patch:` prefixes documenting the intent. See `notes/` for in-flight investigations.

## Branch policy

- `main` is the deployable branch — what Spark builds from.
- `upstream/main` is the upstream tracking branch — pull periodically and rebase patches on top, or merge as appropriate.
- Topic branches for in-flight work; merge to `main` only after testing.

## Off-Spark durable backup

GitHub: `https://github.com/WholeNother1/paperless-gpt-fork` (private). Push to both `origin` and `spark` for each release-worthy commit.

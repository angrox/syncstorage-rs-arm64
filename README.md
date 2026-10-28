# syncstorage-rs-arm64

Unofficial **linux/arm64** builds of Mozilla's
[syncstorage-rs](https://github.com/mozilla-services/syncstorage-rs) (the
Firefox Sync storage + tokenserver), **Postgres backend**.

Mozilla only publishes amd64 images and has said it won't maintain others
([#304](https://github.com/mozilla-services/syncstorage-rs/issues/304)). This
repo holds no source of its own: it checks out upstream at a release tag and
builds it with upstream's unmodified `Dockerfile`.

```
ghcr.io/angrox/syncserver-postgres-arm64:<upstream-tag>   # e.g. 0.24.0
ghcr.io/angrox/syncserver-postgres-arm64:latest
```

Pin the release tag in production; `latest` is just the newest build.

> **Disclaimer:** This repo (workflow and docs) was written with
> [Claude Code](https://claude.com/claude-code). The image build was tested
> locally on a Raspberry Pi (arm64) before the repo was published and the
> build moved to GitHub Actions. It is not affiliated with or supported by
> Mozilla. Use at your own risk.

## How it builds

`.github/workflows/build.yml`:

1. **resolve** — takes upstream's latest release (or the `tag` input on a
   manual run). On the daily cron it stops if that tag is already in GHCR.
2. **build** — on a native `ubuntu-24.04-arm` runner, builds with
   `SYNCSTORAGE_DATABASE_BACKEND=postgres` and
   `TOKENSERVER_DATABASE_BACKEND=postgres` (same args as Mozilla's own
   Postgres image).
3. **smoke test** — runs the image against a throwaway Postgres 18 and
   requires `/__heartbeat__` to answer. A failing build is never pushed.
4. **push** — to GHCR with the workflow's own `GITHUB_TOKEN`.

Rebuild a release by hand: *Actions → build → Run workflow*, optionally with a
tag. A manual run always rebuilds and overwrites that tag.

## Running it

Configuration is upstream's; see the
[Docker how-to](https://mozilla-services.github.io/syncstorage-rs/how-to/how-to-run-with-docker.html).
Minimum: `SYNC_MASTER_SECRET`, `SYNC_SYNCSTORAGE__DATABASE_URL`,
`SYNC_TOKENSERVER__DATABASE_URL`, `SYNC_TOKENSERVER__INIT_NODE_URL` (the
public URL), plus `SYNC_HOST=0.0.0.0` and `SYNC_TOKENSERVER__ENABLED=true`.

## Maintenance notes

- GitHub disables scheduled workflows in a public repo after 60 days without
  activity. It emails a warning first; re-enable under *Actions → build*.
- Action versions are pinned to commit SHAs and bumped by Dependabot.

## License

MPL-2.0, like upstream.

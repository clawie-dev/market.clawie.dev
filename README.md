# market.clawie.dev

Plugin marketplace for **Clawie**. Discovery, install, stats, reviews for community-published skills, drivers, model providers, channel adapters, connectors, validators, starter packs, eval fixtures.

## Architecture

Hybrid: **git is source-of-truth, Cloudflare D1 is the read-index, R2 holds artifacts.**

```
Submission flow                                        Runtime read flow
─────────────────                                      ─────────────────
Author opens PR with                                   Browser → Pages (NextJS)
  manifest YAML + signed                                 ↓
  artifact reference                                   Workers (API)
   ↓                                                     ↓ ↓
PR review (humans + CI)                              D1 (search/filter/stats)   R2 (download artifact)
   ↓
Merge to main
   ↓
Build pipeline:
  • verify signatures
  • upload artifact → R2
  • upsert listing → D1
   ↓
Listing live on market.clawie.dev
```

### Why hybrid?

- **Every submission is a reviewable PR** — matches spec 024's "human reviewer MUST approve a plugin before its first listing".
- **Audit trail is `git log`** — no separate moderation DB.
- **D1 stays fast** — search, filter, install counts, score histograms in sub-100ms.
- **R2 stores binaries cheaply** — signed artifacts, no egress fees from same-region Workers.
- **Aligns with Constitutional Principle II** (git is the source of truth for everything configurable).

## Stack

- **NextJS** — static pages + API routes
- **Cloudflare Pages** — hosting
- **Cloudflare Workers** — dynamic endpoints
- **D1** — SQLite-as-a-service for the read index + stats
- **R2** — S3-compatible object storage for plugin artifacts

## Listings (the git source of truth)

```
listings/
├── skills/
│   └── <name>/
│       ├── manifest.yaml      # name, version, author, signature, declared perms
│       ├── README.md
│       └── CHANGELOG.md
├── drivers/
├── providers/
├── adapters/
├── connectors/
├── validators/
├── starter-packs/
└── fixtures/
```

A new submission = a PR adding `listings/<type>/<name>/manifest.yaml`. CI runs signature + smoke + schema checks; merge triggers the build pipeline.

## Status

Bootstrap pending. See [`clawie-dev/specs/speckit/024-plugin-marketplace`](https://github.com/clawie-dev/specs/tree/main/speckit/024-plugin-marketplace) for the spec.

## License

MIT — see [LICENSE](LICENSE).

# market.clawie.dev

Planned plugin marketplace for **Clawie**. Discovery, install, stats, reviews for community-published skills, drivers, model providers, channel adapters, connectors, validators, starter packs, eval fixtures.

> **Status:** Pending. The repo currently contains only README + LICENSE — no
> NextJS app, no Workers, no `listings/` tree, no D1 schema. The design below
> is the contract this repo will implement when [spec 024](https://github.com/clawie-dev/specs/tree/main/speckit/024-plugin-marketplace)
> enters delivery in a v1.x release. v1.0 of the framework ships the
> intent + agent + team substrate plugins will hang off; the marketplace
> itself is later.

## Planned architecture

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

## Planned listings layout (the git source of truth)

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

## Spec

[`clawie-dev/specs/speckit/024-plugin-marketplace`](https://github.com/clawie-dev/specs/tree/main/speckit/024-plugin-marketplace).

## License

MIT — see [LICENSE](LICENSE).

# Android 17 + Jetpack Migration Research

Living research for WorkIndia Android (`workindiaandroid`) platform and modernization migrations.

## Shareable dashboard

Open the live dashboard (updates when this repo is updated):

**https://wi-rajeev-ranjan.github.io/android-migration-research/**

Source of truth for every row:

- [`docs/migrations.json`](docs/migrations.json) — all migration IDs, decisions, evidence
- [`docs/index.html`](docs/index.html) — interactive filters / triage UI

### How updates work

1. Edit `docs/migrations.json` (or ask the agent to update decisions / add items).
2. Commit + push to `main`.
3. GitHub Pages republishes — everyone with the link sees the new state.

Decision values per item: `undecided` | `do_now` | `do_next` | `park` | `reject`.

## Inventory markdown

[`01_possible_migrations_inventory.md`](01_possible_migrations_inventory.md) is the narrative inventory. Prefer updating the JSON for the shared dashboard; keep the markdown in sync when the catalog changes.

## Baseline (from last scan)

| Item | Value |
|------|-------|
| compile / targetSdk | 36 |
| minSdk | 21 |
| AGP / Gradle | 8.6.1 / 8.10.2 |
| Kotlin | 2.1.21 |
| Compose BOM | 2025.09.01 |

## Local preview

```bash
cd docs
python3 -m http.server 8765
# open http://localhost:8765
```

(`file://` will not load `migrations.json` because browsers block local fetch.)

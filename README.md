# Affiliate-Assistant

Automation experiments for affiliate promotions powered by OpenClaw.

## Current Status (2026-02-27)

- Baseline PRD drafted outlining the "Affiliate Lister" workflow (see [`docs/affiliate-lister-PRD.md`](docs/affiliate-lister-PRD.md)).
- Google Sheet intake schema defined (`Affiliate Associate Tracker` sheet with columns: `WP ID, Status, Store, Product URL, Affiliate URL, Review, Listing, Notes, Image URL, WP URL, Timestamp`) plus a `TWS Stores` lookup (`Store, Slug, URL`). For MVP, `Review` holds the social blurb (with image placeholders) and `Listing` holds lightweight machine-readable WP listing details.
- Infrastructure groundwork: OpenClaw instance stabilized with Nexos + OpenRouter, sidecar gateway documented, Brave Search requirements captured.
- Next up: implement sheet polling -> OpenClaw workflow to fetch listing/review content, generate branded copy, post to the selected WordPress store, and email the social blurb for approval.
- Domain/site mappings are intentionally excluded from this repo; see private references in `../TWS-References/affiliate-assistant-private/store-endpoints.yaml`.

## Repo Layout

- `docs/affiliate-lister-PRD.md` - working product requirements and notes.
- `README.md` - quick progress snapshots.

More scaffolding (skills, intake scripts, etc.) will land here as the workflow matures.

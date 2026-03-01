## Affiliate Lister PRD

### 1. Summary
Build an OpenClaw-based "Affiliate Lister" workflow that ingests an Amazon listing URL + review link (plus a supplied affiliate link), generates product copy, publishes a WordPress product, and produces a social blurb for later posting. MVP targets two stores (art/craft supplies + board games) with a human review step before publishing.

### 2. Objectives & KPIs
- Primary goals
  - Ship an end-to-end MVP using Google Sheets as the operational control plane.
  - Enforce human approval before any WordPress publish action.
- Initial MVP KPIs (to refine after first working release)
  - Approval rate: `% of drafted items that reach approved/published status`.
  - Error rate: `% of workflow runs that fail before draft or publish completion`.
  - Time to publish: `median elapsed time from intake row creation to publish completion`.

### 3. User Stories / Use Cases
- As the curator, I can drop new product leads (Amazon link, review link, affiliate link) into a repeatable Google Sheet intake (with Airtable as a future option) and have OpenClaw draft a WP product entry I can review/approve.
- As the curator, I receive a ready-to-post social blurb (with affiliate + WP links) persisted in the sheet `Review` field so I can schedule it later.

### 4. Functional Requirements
| Step | Requirement | Notes |
| ---- | ----------- | ----- |
| Intake | Accept Amazon listing URL, review URL, affiliate link | MVP source is Google Sheet. Affiliate link may be missing at intake; if missing, set `Status` to `NEED AFF LINK`. Airtable may be added later as an alternate intake. |
| Content fetch | Pull structured data from Amazon page and review link | Support fetch skill + light parsing for title, specs, sentiment. |
| Copy generation | LLM produces deploy-safe copy with configurable tone | Default tone comes from public template; overrides resolved by `client` + `store_channel` from private config. |
| WordPress publish | Create/update WP product on specified site (store A/store B) with best-guess categories | Later enhancement: keyword library and category optimization. |
| Social blurb | Generate short post containing WP link + affiliate link and persist it in-sheet | Store blurb text in the `Review` field. Include image placeholders such as `[IMAGE_1]`, `[IMAGE_2]` until media flow is implemented. |
| Listing record | Persist machine-usable listing draft details in-sheet | Write lightweight YAML (or compact JSON) into the `Listing` field so humans can read and automation can parse. |
| Approval flow | Require human review before publish; toggle off later | Source of truth is the intake record `Status` field. Publisher only acts on sheet status. MVP statuses: `NEW`, `NEED AFF LINK`, `DRAFT_READY`, `APPROVED`, `REJECTED`, `PUBLISHED`, `ERROR`. |

### 5. Data & Integrations
- Inputs: Amazon listing URL, review URL, affiliate link (optional at intake for MVP), target store indicator.
- Systems:
  - WordPress REST APIs for two production store endpoints (domain mapping kept in private reference docs).
  - Intake source: Google Sheet (MVP), Airtable (future option).
  - Email path (optional in MVP): sends review notices and can ingest replies that update the intake `Status` field.
  - OpenRouter LLM models (initially `openrouter/auto`, with later free/budget models).
  - Tone configuration:
    - Public template: `docs/tone-config.template.yaml`
    - Private overrides: `../TWS-References/affiliate-assistant-private/tone-overrides.private.yaml`

#### 5.1 Listing Field Mini-Schema (MVP)
- Field: Google Sheet `Listing` column
- Preferred format: YAML (single cell text blob)
- Required keys:
  - `version` (string, e.g. `v1`)
  - `store` (string)
  - `status_snapshot` (string; status when payload was written)
  - `source.product_url` (string URL)
  - `source.review_url` (string URL)
  - `source.affiliate_url` (string URL; blank allowed only with `NEED AFF LINK`)
  - `draft.title` (string)
  - `draft.summary` (string)
  - `draft.key_points` (list of strings)
  - `draft.categories` (list of strings)
  - `draft.tags` (list of strings)
  - `draft.disclosure` (string)
  - `draft.social_blurb` (string; should match `Review`)
  - `draft.image_placeholders` (list, e.g. `[IMAGE_1]`)
  - `meta.generated_at` (ISO timestamp string)
  - `meta.model` (string)
- Example:
```yaml
version: v1
store: art-craft-store
status_snapshot: DRAFT_READY
source:
  product_url: https://example.com/product
  review_url: https://example.com/review
  affiliate_url: https://example.com/aff
draft:
  title: "Example Product Title"
  summary: "Short buyer-focused summary."
  key_points:
    - "Point one"
    - "Point two"
    - "Point three"
  categories:
    - "Category A"
  tags:
    - "tag-a"
    - "tag-b"
  disclosure: "This post contains affiliate links."
  social_blurb: "Quick hook. Value sentence. [IMAGE_1] https://wp.example/item https://aff.example/item #ad"
  image_placeholders:
    - "[IMAGE_1]"
meta:
  generated_at: "2026-02-28T15:04:05Z"
  model: "openrouter/auto"
```

### 6. Workflow Diagram / Description
1. Intake trigger (Google Sheet row) provides Amazon URL, review URL, affiliate link, target store.
2. If affiliate link is missing, set `Status` to `NEED AFF LINK` and wait for curator update.
3. Agent fetches listing + review content, extracts salient details.
4. Agent resolves tone profile using precedence: store override -> client override -> default deploy tone.
5. LLM generates product copy, meta fields, keywords with resolved tone.
6. Agent writes social blurb to `Review` (with image placeholders) and listing draft details to `Listing`; sets row status to `DRAFT_READY`.
7. Human approval updates the same row `Status` (directly in sheet, or via email reply ingestion that writes to sheet).
8. Publisher polls sheet `Status` as source of truth and publishes/updates WP only when status is `APPROVED`.
9. On successful publish, set `Status` to `PUBLISHED` and store final WP URL.

### 7. Content & Branding Guidelines
- Public default deploy tone (safe to expose in this repo):
  - Clear, concise, factual.
  - Benefits-first with balanced pros/cons.
  - No aggressive hype, no unverifiable claims, no insider/personal references.
  - CTA style: low-pressure and informative.
- Tone override model:
  - `default`: fallback tone used when no override matches.
  - `clients.<client>.default`: client-level override.
  - `clients.<client>.store_channels.<store_channel>`: store/channel-specific override.
  - Resolution order: `clients.<client>.store_channels.<store_channel>` -> `clients.<client>.default` -> `default`.
- Privacy rule:
  - Public repo may include only placeholder/default tone text.
  - Brand-specific voice packs and store-specific tone details must remain in private references.
- Required output elements:
  - Product copy: title, short summary, 3-5 key points, caveat/disclaimer line, affiliate disclosure line.
  - Social blurb: 1 short hook, 1 value sentence, WP link, affiliate link, disclosure tag.

### 8. Non-Functional Requirements
- MVP review step required before WP publish; later ability to disable.
- Track provenance (which Amazon link, affiliate link used).
- Handle both stores with minimal duplication.

### 9. Status Model (MVP)
- Status enum (Google Sheet `Status` column):
  - `NEW`: intake row created, not processed yet.
  - `NEED AFF LINK`: required affiliate URL is missing; waiting for curator to fill `Affiliate URL`.
  - `DRAFT_READY`: content draft + publish payload prepared; awaiting human decision.
  - `APPROVED`: human approved; eligible for publish.
  - `REJECTED`: human rejected; no publish attempt.
  - `PUBLISHED`: publish succeeded and WP URL captured.
  - `ERROR`: workflow failed; investigate and retry.
- Expected transition path:
  - `NEW -> DRAFT_READY -> APPROVED -> PUBLISHED`
- Alternative paths:
  - `NEW -> NEED AFF LINK -> DRAFT_READY`
  - `DRAFT_READY -> REJECTED`
  - `NEW|NEED AFF LINK|DRAFT_READY|APPROVED -> ERROR`
- Operational rule:
  - Publisher must only publish rows in `APPROVED` status and set `PUBLISHED` only after confirmed WP success.

### 10. Deferred
- Automated affiliate-link generation is deferred until after MVP stabilization.

### 11. Open Questions
- Future persistence strategy for social blurbs beyond the `Review` placeholder field.

## Affiliate Lister PRD

### 1. Summary
Build an OpenClaw-based "Affiliate Lister" workflow that ingests an Amazon listing URL + review link (plus a supplied affiliate link), generates branded copy, publishes a WordPress product, and produces a social blurb for later posting. MVP targets two stores (art/craft supplies on threewheeledsloth.com and board games on theanaloggamingsociety.org) with a human review step before publishing.

### 2. Objectives & KPIs
- Primary goals
- Success metrics

### 3. User Stories / Use Cases
- As the curator, I can drop new product leads (Amazon link, review link, affiliate link) into a repeatable intake (email, sheet, Airtable—TBD) and have OpenClaw draft a WP product entry I can review/approve.
- As the curator, I receive a ready-to-post social blurb (with affiliate + WP links) stored/emailed so I can schedule it later.

### 4. Functional Requirements
| Step | Requirement | Notes |
| ---- | ----------- | ----- |
| Intake | Accept Amazon listing URL, review URL, affiliate link | Source can be Google Sheet / Airtable / email trigger (TBD). |
| Content fetch | Pull structured data from Amazon page and review link | Support fetch skill + light parsing for title, specs, sentiment. |
| Copy generation | LLM produces branded summary, meta tags, keywords | Brand voice: sarcastic East-Coast Gen-X independent artist / curator tied to crafting & board-game scenes. |
| WordPress publish | Create/update WP product on specified site (3WS or TAGS) with best-guess categories | Later enhancement: keyword library and category optimization. |
| Social blurb | Generate short post containing WP link + affiliate link; store/email for review | MVP: email or store as draft, review required before posting. |
| Approval flow | Require human review before publish; toggle off later | Provide short summary + “Approve/Revise” path. |

### 5. Data & Integrations
- Inputs: Amazon listing URL, review URL, affiliate link (user-supplied for MVP), target store indicator.
- Systems:
  - WordPress REST APIs for threewheeledsloth.com & theanaloggamingsociety.org.
  - Intake source (candidate: Google Sheet or Airtable).
  - Email or queue to deliver the social blurb / approval request.
  - OpenRouter LLM models (initially `openrouter/auto`, with later free/budget models).

### 6. Workflow Diagram / Description
1. Intake trigger (sheet row / email) provides Amazon URL, review URL, affiliate link, target store.
2. Agent fetches listing + review content, extracts salient details.
3. LLM generates product copy, meta fields, keywords (brand voice).
4. Agent selects best-fit categories and prepares WP product payload; holds for approval.
5. On approval, agent publishes/updates WP product and stores generated social blurb (email to curator).
6. Blurb archived for future auto-posting (post-MVP enhancement).

### 7. Content & Branding Guidelines
- Voice / tone
- Required elements for product copy and social blurb

### 8. Non-Functional Requirements
- MVP review step required before WP publish; later ability to disable.
- Track provenance (which Amazon link, affiliate link used).
- Handle both stores with minimal duplication.

### 9. Open Questions
- Preferred intake method? (Google Sheet vs Airtable vs email keyword trigger)
- How to persist social blurbs for future scheduling?
- Long-term plan for automated affiliate-link generation?

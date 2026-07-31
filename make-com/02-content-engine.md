# Make.com Scenario Blueprint
## 02 — Content Engine (08:00 Daily / Triggered)

**Scenario Name:** `WenceStudio — Content Generation Engine`  
**Trigger Options:**
1. Schedule → Every day at 08:00
2. Preferred: Webhook / Notion “Watch Database Items” when status = `Validated` or `New` + high opportunity_rating

---

### High-Level Flow

1. Trigger (Schedule or Notion Watch)
2. Fetch high-priority signals from Momentum Intelligence
3. Sequential agent chain:
   - Opportunity Analysis
   - Context Layer (knowledge base lookup)
   - Long-Form Draft
   - Visual Asset Generation (optional external image service)
   - Distribution Package
4. Update Notion record with all assets + set status = `Approved` (or `Ready for Distribution`)
5. Error handling + fallback model routing

---

### Module Sequence

#### 1. Trigger
- **Option A (Scheduled):** Schedule → 08:00 daily
- **Option B (Event-driven – recommended):** Notion → Watch Database Items  
  Database: Momentum Intelligence 2026  
  Filter: status = `New` OR `Validated` AND opportunity_rating = `High`

#### 2. Notion — Search / Get Database Items
- Filter: status in [`New`, `Validated`] + opportunity_rating = `High` (or top N by momentum_score)
- Limit: 3–5 per run (prevent overload)
- Sort: momentum_score DESC

#### 3. Iterator
- Iterate over selected signals

#### 4. OpenAI / Claude — Opportunity Analysis
- Model: High-reasoning (gpt-4o / claude-sonnet / o1-class)
- System + User prompt: Use `prompts/opportunity-analysis.md`
- Input: signal title, notes, source, momentum_score, etc.
- Output: structured Opportunity Brief (JSON preferred)

#### 5. Context Layer (Knowledge Base)
- Notion → Search pages in WenceStudio knowledge base / brand guidelines database
- Or HTTP → internal vector search / Pinecone / Weaviate if you have one
- Or simply inject a fixed “Brand Positioning” text block
- Pass relevant context snippets into the next prompt

#### 6. OpenAI / Claude — Long-Form Draft
- Model: Best writing model available (Claude Sonnet / GPT-4o)
- Prompt: Use `prompts/long-form-brief.md` + Opportunity Brief + Context
- Temperature: 0.4–0.6
- Max tokens: sufficient for ~1,200 words
- Output: Full markdown article

#### 7. (Optional) Visual Asset Generation
- HTTP → OpenAI Images / Flux / Ideogram / Midjourney API
- Generate three assets:
  - 16:9 article cover
  - 4:5 LinkedIn
  - 1:1 Instagram
- Store image URLs (or upload to Cloudinary / S3 / Webflow assets)

#### 8. OpenAI / Claude — Distribution Package
- Prompt: Use `prompts/distribution-package.md`
- Input: Long-form draft + Opportunity Brief
- Output: Strict JSON containing:
  - seo_title
  - meta_description
  - linkedin_post
  - x_thread (array)
  - newsletter_summary
  - carousel_json

#### 9. Notion — Update Database Item
- Update the original signal record:
  - status → `Approved`
  - notes → append full Opportunity Brief + any warnings
  - (Add new properties if needed: `draft_md`, `seo_title`, `linkedin_post`, `x_thread`, `carousel`, `cover_image_url`, etc.)

  Recommended additional Notion properties to create once:
  - `draft_content` (rich text / long text)
  - `seo_title` (text)
  - `meta_description` (text)
  - `linkedin_post` (text)
  - `x_thread` (text)
  - `newsletter_summary` (text)
  - `carousel_json` (text)
  - `cover_16x9` (url / files)
  - `linkedin_4x5` (url / files)
  - `instagram_1x1` (url / files)

#### 10. Fallback Routing (Error Handler on LLM modules)
- Primary model fails → switch to secondary model (Claude ↔ OpenAI)
- Still fails → set status = `Needs Manual Review` + notify human
- Use Make.com Error Handler + Router for this

#### 11. Success / Failure Notification
- Slack / Discord / Email: “Content Engine finished — X briefs ready for distribution”

---

### Reliability Features (as specified)
- Automated retry: Enable “Break” + “Resume” or use Make’s built-in retry on HTTP/LLM modules
- Model fallback routing: explicit Router after first LLM error
- Target: full asset bundle < 4 minutes per signal (run in parallel where safe)

### Notes
- Keep the long-form draft and distribution package as separate steps so you can re-run only packaging if needed.
- Store raw LLM responses in a Data Store or Notion for auditability.

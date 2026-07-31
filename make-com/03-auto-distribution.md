# Make.com Scenario Blueprint
## 03 — Auto-Distribution (10:00 / Status Trigger)

**Scenario Name:** `WenceStudio — Omni-Channel Distribution`  
**Trigger:** Notion → Watch Database Items  
Database: Momentum Intelligence 2026  
Filter: status = `Approved`

Alternative: Schedule at 10:00 that searches for all `Approved` items not yet published.

---

### High-Level Flow

1. Detect newly Approved signal
2. Create Webflow CMS item + publish
3. Publish X thread
4. Publish LinkedIn post / carousel
5. Queue newsletter summary
6. Update Notion with live_url, indexing_status, engagement_tracker_id
7. Notify internal channels

---

### Module Sequence

#### 1. Trigger
- Notion → Watch Database Items
- Database: Momentum Intelligence 2026
- Watched property: status
- Condition: status equals `Approved`

#### 2. (Optional) Delay
- 1–2 minutes to ensure all content fields are fully written by Content Engine

#### 3. Webflow — Create CMS Item
- Connection: Webflow
- Site + Collection: (your Articles / Intelligence collection)
- Map:
  - Name / Title ← seo_title or Name
  - Slug ← auto-generate from title
  - Body ← draft_content (markdown → HTML if needed)
  - Meta Title ← seo_title
  - Meta Description ← meta_description
  - Cover Image ← cover_16x9 URL
  - Status → Published (or Draft then Publish)
- After create → Webflow → Publish Item / Publish Site (if required)

#### 4. Structured Data (optional but recommended)
- HTTP → PATCH the Webflow item or use a secondary module to inject JSON-LD
- Or handle via Webflow custom code / Make + Webflow API

#### 5. X (Twitter) — Create Thread
- X module → Create a Tweet (first tweet)
- Then Iterator over remaining tweets in `x_thread` array
- Reply to previous tweet ID to form the thread
- Final tweet should contain the live Webflow URL

#### 6. LinkedIn — Create Post / Carousel
- LinkedIn module → Create an Organic Post
- Text ← linkedin_post
- Media: upload 4:5 image or carousel images if supported
- (If carousel is complex, post single image + link, or use LinkedIn multi-image)

#### 7. Newsletter Queue
- Option A: Kit / ConvertKit → Add subscriber tag or create broadcast draft
- Option B: Notion → Create item in a “Newsletter Queue” database with the newsletter_summary + link
- Option C: Make Data Store → push summary for the Mon/Fri scenario to pick up

#### 8. Notion — Update Database Item
- Set:
  - status → `Published`
  - live_url → Webflow item URL
  - indexing_status → `Submitted` (or leave for later Search Console automation)
  - engagement_tracker_id → X tweet ID / LinkedIn post ID (for later analytics)
  - attribution_data → any UTM or campaign ID you use

#### 9. Community Notification (optional)
- Slack / Discord / Telegram → post “New Intelligence Brief live: [title] — [url]”

#### 10. Error Handler
- On any critical failure (Webflow or X):
  - Revert status to `Approved` or set `Needs Manual Distribution`
  - Notify human with error details + signal ID

---

### Additional Recommended Properties in Notion
Add these once if not already present:
- `live_url` (URL)
- `indexing_status` (Select: Pending / Submitted / Indexed)
- `engagement_tracker_id` (Text)
- `attribution_data` (Text)
- `published_at` (Date)

---

### Notes
- Respect X and LinkedIn rate limits. One high-quality post per signal is usually enough.
- For carousels, pre-generate images in Content Engine and store URLs.
- Webflow CMS API has its own rate limits — queue if you ever batch more than a few items.

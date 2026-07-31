# Make.com Scenario Blueprint
## 04 — Newsletter Dispatch (Mon & Fri 07:00)

**Scenario Name:** `WenceStudio — Protocol Newsletter`  
**Schedule:** Every Monday and Friday at 07:00

---

### High-Level Flow

1. Schedule trigger (Mon + Fri 07:00)
2. Gather the week’s best signals + published content
3. AI assembly of “The WenceStudio Protocol” sections
4. Personalization / segmentation logic
5. Create + send (or schedule) via Kit / ConvertKit / Beehiiv / Loops
6. Log send + update dashboard metrics

---

### Module Sequence

#### 1. Schedule
- Type: Schedule
- Days: Monday, Friday
- Time: 07:00

#### 2. Notion — Search Database Items (Published this week)
- Database: Momentum Intelligence 2026
- Filter:
  - status = `Published`
  - published_at ≥ start of current week (or last 7 days)
- Sort: momentum_score DESC
- Limit: top 5–8

#### 3. Aggregate Content
- Array Aggregator → collect titles, newsletter_summaries, live_urls, opportunity_ratings

#### 4. OpenAI / Claude — Protocol Assembly
- Prompt structure:

  You are assembling “The WenceStudio Protocol” newsletter.

  From the provided signals and summaries, produce exactly these five sections:

  1. **The Signal** — The single most important development of the period (1–2 paragraphs)
  2. **The Noise** — One low-value trend or hype cycle to ignore (short, sharp)
  3. **The Asset** — One concrete prompt, framework, or workflow the reader can use today
  4. **The Opportunity** — Clear revenue or efficiency application
  5. **The Offer** — Dynamic recommendation (placeholder for segmentation)

  Tone: Luxury Intelligence. Clear. Authoritative. Zero fluff.

- Input: the aggregated published items + any internal knowledge
- Output: structured markdown or JSON with the five sections

#### 5. (Optional) Segmentation / Personalization
- Kit / ConvertKit → Get subscriber segments or tags
- Or use Make Data Store / Notion subscriber profile table
- Router based on segment:
  - Operators → emphasize efficiency / systems
  - Creators → emphasize content leverage
  - Builders → emphasize technical implementation
- Adjust “The Offer” section accordingly

#### 6. Email Platform — Create Campaign / Broadcast
**Kit (ConvertKit) example:**
- Create a Broadcast
- Subject: dynamic (e.g. “The WenceStudio Protocol — [Signal Title]”)
- Content: assembled markdown → HTML
- Segment / Tag: as determined above
- Action: Send immediately **or** Schedule for 07:15

Alternative platforms (Beehiiv, Loops, Customer.io, etc.) follow the same pattern.

#### 7. Notion / Data Store — Log the Send
- Create record in a “Newsletter Log” database:
  - send_date
  - subject
  - signal_ids included
  - segment
  - status: Sent

#### 8. Internal Notification
- Slack / Discord: “Protocol newsletter dispatched — [subject]”

#### 9. Error Handler
- On failure → set status in log to `Failed` + notify human
- Do **not** auto-retry send (risk of double emails). Manual review only.

---

### Recommended Supporting Notion Databases
1. **Newsletter Queue** (optional intermediate)
2. **Newsletter Log** (send history + metrics)
3. **Subscriber Segments** (if not fully managed inside Kit)

---

### Notes
- Keep the AI assembly step deterministic (low temperature) so the voice stays consistent week to week.
- “The Asset” section is high-leverage — consider maintaining a library of proven prompts/frameworks the model can pull from.
- Always include unsubscribe + physical address compliance blocks required by your ESP.

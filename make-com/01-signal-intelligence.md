# Make.com Scenario Blueprint
## 01 — Signal Intelligence (06:00 Daily)

**Scenario Name:** `WenceStudio — Signal Collection & Analysis`  
**Schedule:** Every day at 06:00 (timezone of choice, preferably America/Chicago or UTC)  
**Trigger:** Schedule → Every day → 06:00

---

### High-Level Flow

1. Schedule Trigger
2. Parallel source collectors (HTTP / RSS / API modules)
3. Aggregate & normalize signals
4. OpenAI / Claude validation + scoring
5. Filter by thresholds
6. Create records in Notion (Momentum Intelligence 2026)
7. Error handling + notification

---

### Module Sequence

#### 1. Schedule
- Type: Schedule
- Interval: Days
- Days: Every day
- Time: 06:00

#### 2–7. Parallel Source Collectors (use Router or multiple parallel branches)

**Branch A — Reddit**
- HTTP → Make a Request
- Method: GET
- URL: `https://www.reddit.com/r/MachineLearning+singularity+AI+LocalLLaMA+Automation/hot.json?limit=25`
- Headers: `User-Agent: WenceStudioBot/1.0`
- Parse JSON → Iterator over `data.children`
- Map: title, score, num_comments, url, created_utc, subreddit

**Branch B — X (Twitter)**
- Use X (Twitter) module or HTTP with Bearer token
- Search recent tweets with query: `#AgenticAI OR #AIOS OR #MCP OR #ComputerUse OR #VoiceAI OR #Automation -is:retweet lang:en`
- Limit: 50
- Map: text, public_metrics, created_at, author

**Branch C — Google Trends**
- HTTP → SerpApi or RapidAPI Google Trends endpoint (or custom scraper)
- Keywords: `AI agents`, `ChatGPT-5.5`, `Gemini 3 Pro`, `Claude Sonnet 5`, `workflow automation`
- Extract interest over time + rising queries

**Branch D — Product Hunt**
- HTTP → Product Hunt GraphQL API
- Query recent posts in AI / Productivity categories
- Sort by votes / comments velocity

**Branch E — Hacker News**
- HTTP → `https://hn.algolia.com/api/v1/search?tags=story&query=AI%20OR%20agent%20OR%20LLM&hitsPerPage=30`
- Or official Firebase HN API

**Branch F — GitHub Trending**
- HTTP → `https://api.github.com/search/repositories?q=created:>YYYY-MM-DD&sort=stars&order=desc` (last 24h)
- Or scrape trending page / use third-party trending API
- Languages: Python, TypeScript + topics: ai, agent, llm

#### 8. Aggregate (Array Aggregator)
- Collect all signals from branches into one array
- Normalize schema:
  ```json
  {
    "title": "",
    "source": "Reddit|X|Google Trends|Product Hunt|Hacker News|GitHub",
    "url": "",
    "engagement": 0,
    "raw_score": 0,
    "timestamp": "",
    "snippet": ""
  }
  ```

#### 9. OpenAI / Claude — Signal Scoring
- Module: OpenAI → Create a Chat Completion (or Anthropic)
- Model: gpt-4o / claude-3-5-sonnet / equivalent high-reasoning model
- System prompt: Use the Opportunity Analysis + novelty scoring logic
- User prompt: Pass the aggregated array
- Ask for structured JSON output:
  ```json
  {
    "validated_signals": [
      {
        "title": "",
        "category": "AI Agents|Infrastructure|Automation|Models|Tools|Other",
        "momentum_score": 0-100,
        "primary_source": "",
        "sentiment": "Bullish|Neutral|Bearish",
        "opportunity_rating": "High|Medium|Low",
        "novelty_score": 0-100,
        "rationale": ""
      }
    ]
  }
  ```
- Temperature: 0.2–0.4

#### 10. Filter
- Only keep signals where:
  - `momentum_score` ≥ 60 **AND**
  - Multi-source confirmation OR `novelty_score` ≥ 70
  - (Implement as Filter module after Iterator)

#### 11. Notion — Create a Database Item
- Connection: Notion
- Database: Momentum Intelligence 2026
- Map fields:
  - Name ← title
  - signal_id ← generated UUID or `SIG-{{timestamp}}-{{index}}`
  - category ← category
  - momentum_score ← momentum_score
  - primary_source ← primary_source
  - sentiment ← sentiment
  - opportunity_rating ← opportunity_rating
  - status ← `New`
  - notes ← rationale

#### 12. Error Handler (on whole scenario or critical modules)
- On error → Slack / Email / Discord notification
- Include scenario name + error message + timestamp
- Optional: Create a “Failed Run” log in a separate Notion database

#### 13. (Optional) Success Summary
- Aggregate count of new signals created
- Send daily digest to internal channel: “Signal Intelligence complete — X new opportunities ingested”

---

### Recommended Settings
- Max execution time: 15–20 min
- Concurrent runs: 1 (prevent overlapping)
- Data store (optional): Keep last-run timestamps per source for delta fetching

### Notes
- Rate limits: Respect Reddit, X, GitHub, Product Hunt quotas. Use delays between parallel branches if needed.
- For production, store API keys in Make.com Connections / Secrets.
- Novelty baseline can be maintained in a separate Data Store or Notion “Historical Signals” table.

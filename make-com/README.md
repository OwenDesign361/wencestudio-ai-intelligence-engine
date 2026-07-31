# Make.com Scenario Blueprints

Complete, production-ready blueprints for the WenceStudio AI Intelligence Engine.

## Scenarios

| # | Scenario | Trigger | File |
|---|----------|---------|------|
| 01 | **Signal Intelligence** | Schedule 06:00 daily | [01-signal-intelligence.md](./01-signal-intelligence.md) |
| 02 | **Content Engine** | Schedule 08:00 or Notion Watch (status = New/Validated) | [02-content-engine.md](./02-content-engine.md) |
| 03 | **Auto-Distribution** | Notion Watch (status = Approved) | [03-auto-distribution.md](./03-auto-distribution.md) |
| 04 | **Newsletter Dispatch** | Schedule Mon & Fri 07:00 | [04-newsletter-dispatch.md](./04-newsletter-dispatch.md) |

---

## Recommended Execution Order & Dependencies

```
06:00  Signal Intelligence
         ↓ (writes status = New)
08:00  Content Engine
         ↓ (writes status = Approved + all assets)
≈10:00 Auto-Distribution (event-driven)
         ↓ (writes status = Published + live_url)
Mon/Fri 07:00  Newsletter Dispatch (pulls Published items)
```

---

## Shared Patterns (use across all scenarios)

### 1. Error Handling Template
- Attach an **Error Handler** route to every critical module (LLM, Webflow, X, Notion write).
- On error:
  1. Log to a Notion “Automation Errors” database (scenario name, module, error message, timestamp, signal_id if available).
  2. Send Slack/Discord/Email alert.
  3. For Content Engine & Distribution: set status to `Needs Manual Review` instead of failing silently.

### 2. Model Fallback Routing
```
Primary LLM module
    ↓ (on error)
Router → Secondary LLM (Claude ↔ OpenAI)
    ↓ (on error)
Set status = Needs Manual Review + notify
```

### 3. Idempotency
- Before creating a Notion record or publishing, check if a record with the same `signal_id` or `live_url` already exists.
- Prevents duplicate content on scenario retries.

### 4. Rate Limit Protection
- Insert **Sleep** modules (2–5 s) between parallel source collectors and between social posts.
- Use Make’s built-in queue / sequential processing for X and LinkedIn.

### 5. Secrets
Store all API keys, tokens, and connection credentials in Make.com Connections. Never hard-code.

---

## Suggested Additional Notion Databases

| Database | Purpose |
|----------|---------|
| Momentum Intelligence 2026 | Core signal + content store (already created) |
| Automation Errors | Centralized error log |
| Newsletter Log | Send history + metrics |
| Brand Knowledge Base | Positioning, past briefs, approved frameworks for Context Layer |

---

## Next Implementation Steps

1. Create the four scenarios in Make.com using the blueprints above.
2. Add the extra properties listed in each blueprint to the Momentum Intelligence database.
3. Connect Notion, OpenAI/Anthropic, Webflow, X, LinkedIn, and your ESP (Kit).
4. Run Signal Intelligence once in test mode → verify records appear.
5. Manually set one record to `Validated` → trigger Content Engine.
6. Manually set to `Approved` → trigger Distribution.
7. Schedule the Newsletter scenario.

Once these four scenarios are live, the autonomous loop is operational.

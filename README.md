# HW #3: First API Call and Prompt Patterns in Code - Report

**Student:** Zhiger Bazarov  
**Model:** `gemini-3.5-flash-lite` (Google GenAI SDK)  
**Date:** 2026-11-12  
**Reviewer:** `ihar_rubanovich@epam.com`  
**Selected Track:** Both tracks (A + B)  

---

## 1. Input Data

### For Track A (Structured Output)
- **Sanitized Ticket / Email (No PII):** names, card numbers, phone numbers and addresses removed; identifiers replaced with placeholders.
```text
Hello, my company card was charged $89.99 twice today for the Pro plan renewal (invoice [INVOICE_ID]). Please refund the duplicate charge before Friday, when our finance team closes the month. Contact: [EMAIL_REDACTED].
```

### For Track B (Prompt Patterns on Video Transcript)
- **Video Source:** Option A — pre-loaded engineering post-mortem transcript from the course notebook (PostgreSQL connection-pool outage, 47 minutes).
- **Transcript Snippet:**
  > [00:00] In this post-mortem, we analyze our production database outage from last Tuesday that lasted 47 minutes.  
  > [00:25] At 14:10 UTC, our primary PostgreSQL cluster experienced a sudden spike in connection pool saturation.
- **Defined Task (`my_task`):**
```text
Extract the root cause, the 3 mitigation steps in order and the key lessons as short bullet points, then propose 3 video titles under 60 characters. Use only facts stated in the transcript.
```

### Design Choices
- **Schema (Track A):** `TicketTriage` extended with two fields. `language: Literal["en","ru","kk","other"]` and `suggested_action: Literal["refund_duplicate","escalate_to_engineering","reply_with_info","manual_review"]`. Closed enums instead of free text, so a database or router can consume the values without post-processing; `amount_usd: float | None` stays nullable for tickets without money.
- **Data isolation (Track B):** the transcript is wrapped in `<transcript>…</transcript>`; the system instruction (`my_role`) declares the tagged text untrusted data that must never be followed as instructions and forbids inventing facts.
- **Secrets:** the API key is read from Colab Secrets (`userdata.get('GEMINI_API_KEY')`); it is not in the notebook, `result.json` or this repository.

---

## 2. Execution Results

### Track A: Structured Output (`triage`)
```json
{
  "category": "billing",
  "urgent": true,
  "amount_usd": 89.99,
  "language": "en",
  "suggested_action": "refund_duplicate"
}
```

### Track B: Model Generation (`my_response`)
```text
**Root Cause**
* An unindexed query introduced in release v2.14 performed a full table scan on the orders table during peak traffic, saturating the PostgreSQL connection pool.

**Mitigation Steps (in order)**
1. Terminated stuck backend queries using `pg_terminate_backend`.
2. Deployed an emergency hotfix adding the composite B-tree index on `user_id` and `created_at`.
3. Increased connection pooler limits in PgBouncer from 50 to 150 dedicated server connections.

**Key Lessons**
* Enforce query performance linting in CI/CD.
* Add automated alerts on connection pool utilization above 80%.
* Require database review for all new query paths.

**Proposed Video Titles**
* PostgreSQL Outage Post-Mortem: Connection Pool Saturation (55 chars)
* Fixing a 47-Minute Database Outage: Post-Mortem Breakdown (55 chars)
* SRE Post-Mortem: Unindexed Queries & Connection Spikes (54 chars)
```

---

## 3. Telemetry & Cost Breakdown

| Metric | Value |
| :--- | :--- |
| **Prompt Tokens (input)** | 417 |
| **Candidates Tokens (output)** | 228 |
| **Thoughts Tokens (thinking)** | 0 |
| **Total Billed Output Tokens** | `output + thinking = ` 228 |
| **Calculated Inference Cost (USD)** | `input × 0.30 / 1M + (output + thinking) × 2.50 / 1M = $` 0.000695 |
| **Cost per 1,000 requests (USD)** | `cost × 1,000 = $` 0.70 |

---

## 4. Engineering Takeaways

1. **Structured Output (JSON Contract):** Yes. The JSON matched `TicketTriage` with no validation errors: `billing`, `urgent: true`, `amount_usd: 89.99`, and both enum fields got allowed values (`language: "en"`, `suggested_action: "refund_duplicate"`).
2. **Data Isolation (XML Tags):** All facts (root cause, 3 mitigation steps in order, 3 lessons) come from the transcript; nothing was invented about the incident. But the model's own character counts are wrong: it wrote "(55 chars)" for two titles that actually have 57 characters. The titles are still under 60, but numbers the model reports about its own output must be checked in code.
3. **Thinking & Latency:** `thinking_level="minimal"` gave 0 thinking tokens, and no more reasoning was needed: this is extraction, not multi-step logic. Zero thinking tokens kept the cost at about $0.70 per 1,000 requests.

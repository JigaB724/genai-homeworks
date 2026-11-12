# AI Use-Case Memo: Crisis Disinformation Verification Co-pilot

**Student:** Zhiger Bazarov · **Course:** GenAI Basics — Lesson 01 · **Context:** pet project *Disinfo Monitor*

## 1. Problem Framing & Friction Point
Disinfo Monitor tracks crisis-related claims (floods, earthquakes, outages) in Kazakh, Russian and English Telegram channels and news sites. During an incident, hundreds of posts appear per hour. An analyst checks each flagged claim manually: finds the original post, searches official sources (emergency services, akimats, ministries), compares facts and writes a verdict note. One check takes 10–15 minutes, so during a spike the backlog grows by hours, while fake evacuation orders and invented casualty numbers spread fastest in the first hour.

## 2. Technology Classification
Hybrid pipeline:
- **Collection, deduplication, language detection, official-source lookup:** Software 1.0 (scrapers, hashing, SQL over a fixed whitelist). Deterministic, cheap and auditable.
- **Suspicion scoring:** Software 2.0 — the existing LaBSE + CNN-BiLSTM classifier ranks every post in milliseconds. Running an LLM on thousands of posts would add latency and token cost for a task a trained classifier already handles.
- **Verification note:** Software 3.0 with human review — an LLM compares one high-scoring claim with retrieved official statements and drafts a cited note. Cross-lingual semantic comparison is where foundation models are genuinely stronger.
- **Software 4.0 (autonomous agent): rejected** — the system must never act publicly on its own.

## 3. Data Contracts
- **Inputs:**
  1) Claim text, language, post URL and timestamp;
  2) Classifier score (0–1);
  3) Up to 5 official-source passages from the whitelist, each with URL and publication time.
- **Expected Output (JSON):** `verdict_draft` (`supported` | `contradicted` | `unverified`), `evidence` (list of `{url, quote}`), `note` (≤ 60 words, in the claim's language).
  Code downgrades any verdict without a cited official URL to `unverified`.

## 4. Blast Radius & Risk Tier
- **Risk Tier:** 🟡 Yellow Zone (co-pilot, analyst verifies).
- **Failure Modes:** hallucinated or misquoted official statements; labelling a real emergency warning as fake, which could make people ignore an evacuation order; prompt injection hidden in post text.
- **Guardrails:** claim wrapped in `<claim>` tags as untrusted data; evidence limited to retrieved passages; code checks that every quote exists verbatim in its source; structured output schema.

## 5. Human-in-the-Loop Control Gate
The draft appears in the analyst dashboard next to the original post and sources:
1. The analyst opens each cited source and confirms the quote;
2. The analyst approves, edits or rejects the verdict and note;
3. Only approved notes enter the verified database and the public digest; every approval is logged with analyst ID and time.

Without an explicit approval, nothing leaves the internal dashboard.

## 6. Architectural Taboo (What Will NEVER Be Automated)
The language model is strictly prohibited from:
1. Publishing any verdict, label or digest, or replying to post authors;
2. Labelling an official emergency warning as false, or reporting accounts to platforms or authorities;
3. Modifying the official-source whitelist.

Publication and reporting live in a separate service that requires analyst authorization.

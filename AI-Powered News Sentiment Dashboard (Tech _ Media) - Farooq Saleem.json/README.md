# AI‑Powered News Sentiment Dashboard (Tech / Media)

An n8n workflow that fetches top technology headlines, analyzes each article with an LLM for sentiment and relevance, filters the most relevant stories, and emails a polished HTML digest to your inbox.

## What it does

- Pulls top tech headlines from NewsAPI (pageSize=20, category=technology).
- Extracts and prepares key fields and an excerpt for each article.
- Uses OpenAI (gpt-4.1-mini) to:
  - Classify overall sentiment (positive/negative/neutral).
  - Generate a 1‑line summary.
  - Score relevance to technology trends (0–100).
- Merges AI outputs back onto each article.
- Keeps only highly relevant items (relevanceScore ≥ 90).
- Formats selected items, aggregates them into a digest, and prompts the LLM to produce a clean, mobile‑friendly HTML email.
- Sends the email via Gmail.

## High‑level flow

1. Manual Trigger
2. HTTP Request → NewsAPI top headlines (technology)
3. Code → Flatten and map fields per article
4. Set → Compute excerpt (content → description → title)
5. Split In Batches → Loop per article
6. AI Agent (OpenAI) → Sentiment, summary, relevanceScore (JSON)
7. Code → Robust JSON parsing/cleanup
8. Merge (by position) → Combine AI results with original article data
9. Filter → relevanceScore ≥ 90
10. Code → Format brief item summaries
11. Aggregate → Collect into a digest array
12. Code → Join into one digest string
13. AI Agent (OpenAI) → Compose polished HTML email
14. Code → Strip any code fences
15. Gmail → Send HTML email (subject includes current date/time)

## Inputs

- NewsAPI API key
  - Currently hardcoded in the HTTP Request URL. Replace with a secure credential or environment variable.
- OpenAI API key (for gpt-4.1-mini)
- Gmail OAuth2 credentials (to send the email)
- Optional runtime settings:
  - Category: technology (change via HTTP Request URL)
  - pageSize: 20 (in URL)
  - Relevance threshold: 90 (Filter node)
  - Recipient email address (Gmail node)
  - Subject template: “AI News Digest – {{ $now }}”

## Outputs

- HTML email sent to the configured recipient containing:
  - A short intro and section header
  - 3–5 concise highlights with company/product names in bold and emphasis where useful
  - Optional “Editor’s Take” line
  - A small gray footer: “Generated automatically by the AI-Powered News Sentiment Dashboard.”
- Intermediate (inside n8n):
  - Enriched articles with fields like:
    - title, author, description, url, urlToImage, publishedAt, source, content, excerpt
    - sentiment, summary, relevanceScore

## Setup

1. NewsAPI
   - Create an API key at newsapi.org.
   - In the HTTP Request node, replace the hardcoded key with a credential or env var.
     - Best practice: use header “X-Api-Key: {{ $env.NEWSAPI_KEY }}” and remove the apiKey query param.

2. OpenAI
   - Add OpenAI credentials in n8n and link them to both LLM nodes (already referenced as “OpenAi account”).

3. Gmail
   - Set up Gmail OAuth2 credentials in n8n and link the “Send a message” node.
   - Update the “sendTo” email if needed.

4. Optional tweaks
   - Change relevance threshold in the Filter node (default: 90).
   - Adjust pageSize/category in the HTTP Request URL.
   - Replace Manual Trigger with a Cron node for daily digests.

## Notes and caveats

- Cost/quotas: LLM calls and NewsAPI requests may incur usage or hit limits.
- Reliability: The parsing step cleans up non‑strict JSON from the model, but consider adding fallback logic if no items pass the filter (e.g., send a “No highlights today” email or skip send).
- Security: Avoid hardcoding API keys; store credentials securely in n8n.
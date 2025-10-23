# AI‑Powered News Sentiment Dashboard (Tech / Media)

A ready‑to‑run n8n workflow that pulls top technology headlines, uses an LLM to score sentiment and relevance, assembles a concise digest, converts it into a polished HTML email, and sends it to your inbox.

## What it does
- Fetches top technology headlines from NewsAPI.
- Extracts key fields and builds a clean excerpt per article.
- Loops through each article and uses an OpenAI model to:
  - classify sentiment (positive/negative/neutral),
  - produce a 1‑line summary,
  - assign a relevanceScore (0–100) for tech trends.
- Merges AI results with original article data.
- Filters for highly relevant items (relevanceScore ≥ 90 by default).
- Formats the winners into a digest, then asks the LLM to craft a production‑ready HTML email.
- Sends the HTML email via Gmail.

## Data flow (high‑level)
1. Manual Trigger → start workflow on demand.
2. HTTP Request → GET top tech headlines from NewsAPI.
3. Code (JS) → map NewsAPI response into individual article items.
4. Set → normalize fields: title, url, source, publishedAt, content, excerpt (smart fallback).
5. Split in Batches → iterate items one by one (uses a NoOp “Replace Me” node to advance the loop).
6. OpenAI Agent (gpt‑4.1‑mini) → return strict JSON: { sentiment, summary, relevanceScore }.
7. Code (JS) → sanitize and parse agent JSON safely.
8. Merge (by position) → combine AI output with the original article data.
9. Filter → keep items with relevanceScore ≥ 90.
10. Code (JS) → format each item into a readable snippet.
11. Aggregate → collect all snippets.
12. Code (JS) → join snippets into a single “digest” string.
13. OpenAI Agent (gpt‑4.1‑mini) → transform digest into a full HTML email (inline CSS, bullet points, editor’s take).
14. Code (JS) → strip code fences and ensure clean HTML.
15. Gmail → send the HTML email (subject includes current timestamp).

## Inputs and configuration
- News source
  - NewsAPI endpoint: /v2/top-headlines?category=technology&pageSize=20
  - Replace the API key with your own and avoid hardcoding it.
    - Recommended: store as an environment variable or n8n credential and reference via expression, e.g.:
      https://newsapi.org/v2/top-headlines?category=technology&pageSize=20&apiKey={{ $env.NEWSAPI_KEY }}

- Credentials (in n8n)
  - OpenAI: credential “OpenAi account” (used by both LLM nodes; model gpt‑4.1‑mini).
  - Gmail OAuth2: credential “Gmail account”.
  - NewsAPI: use a safe storage method as noted above.

- Tunable parameters
  - Relevance threshold: Filter node currently gte 90. Lower to include more stories.
  - NewsAPI query: adjust category, pageSize, or add country/sources as needed.
  - Email recipient and subject: set in the Gmail node (default recipient is a specific address in the template).
  - Batch behavior: Split in Batches used for sequential LLM calls (adjust if you want larger batches or concurrency elsewhere).

## Outputs
- Enriched article records (internal): title, author, source, publishedAt, url, excerpt, plus AI fields sentiment, summary, relevanceScore.
- Digest (internal): a joined, human‑readable summary list of top‑scoring items.
- Final HTML email (external): polished, inline‑styled HTML with:
  - greeting and intro,
  - “AI & Tech Daily Digest” section,
  - 3–5 bullet highlights,
  - optional “Editor’s Take,”
  - footer note: “Generated automatically by the AI-Powered News Sentiment Dashboard.”
- Email delivery: sent via the Gmail node with HTML body and timestamped subject.

## How to run
1. Import the workflow JSON into n8n.
2. Create/attach credentials:
   - OpenAI API (for the LLM nodes),
   - Gmail OAuth2 (for sending),
   - Provide your NewsAPI key (use env/credentials; update the HTTP Request URL).
3. Update the Gmail node “sendTo” field to your email.
4. Execute the workflow (Manual Trigger) or schedule it with a Cron node if desired.

## Notes and limitations
- Key handling: Do not commit API keys. Move the NewsAPI key out of the URL and into secure storage.
- Rate limits: NewsAPI and OpenAI rate limits apply; adjust frequency and pageSize accordingly.
- AI JSON parsing: The workflow includes a robust cleaner; on failure, sentiment/summary/relevanceScore become null and an error is attached.
- Empty results: If no items meet the threshold, the digest may be empty; consider adding a fallback message if needed.
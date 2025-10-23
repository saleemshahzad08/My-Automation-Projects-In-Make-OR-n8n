# AI‑Powered News Sentiment Dashboard (Tech / Media)

A reusable n8n workflow that fetches the latest technology headlines, scores each article’s sentiment and relevance with an LLM, filters the most relevant items, and delivers a polished HTML email digest to your inbox.

## What it does
1. Fetch tech headlines from NewsAPI (top-headlines, category=technology, pageSize=20).
2. Normalize each article (title, source, publishedAt, excerpt up to 1200 chars).
3. Use an OpenAI chat model to:
   - Classify sentiment (positive/negative/neutral)
   - Generate a one‑line summary
   - Assign a relevanceScore (0–100) to tech trends
4. Merge AI results with original article data.
5. Filter to only high‑relevance items (relevanceScore ≥ 90).
6. Format items into a digest, then have the model write a clean, responsive HTML email (3–5 bullet highlights).
7. Email the HTML digest via Gmail.

## Inputs and prerequisites
- News source
  - NewsAPI endpoint: https://newsapi.org/v2/top-headlines?category=technology&pageSize=20
  - Requires a NewsAPI key (replace the hardcoded key in the HTTP Request node with your own, ideally via env var/credential).
- AI model (OpenAI)
  - Model: gpt-4.1-mini (configured via two LangChain OpenAI Chat nodes)
  - Requires OpenAI API credentials in n8n.
- Email delivery
  - Gmail OAuth2 credential.
  - “Send to” address configured in the Gmail node.

Recommended: Store secrets in n8n credentials or environment variables (e.g., use ?apiKey={{$env.NEWSAPI_KEY}} in the HTTP Request URL).

## Key nodes (simplified flow)
- Manual Trigger → HTTP Request (NewsAPI)  
- Code (normalize articles) → Set (excerpt/title/url/source/publishedAt)  
- Split In Batches → AI Agent (Sentiment/summary/relevance as strict JSON) → Parse JSON  
- Merge (article + AI result by position) → Filter (relevanceScore ≥ 90)  
- Code (format items) → Aggregate → Join into “digest”  
- AI Agent (generate full HTML email) → Clean HTML → Gmail (send)

## Outputs
- Per-article structured data (internal to workflow):
  - title, url, source, publishedAt, excerpt
  - sentiment: positive | negative | neutral
  - summary: one line
  - relevanceScore: number (0–100)
- Final deliverable:
  - A complete HTML email digest sent via Gmail, including:
    - Greeting and intro
    - “AI & Tech Daily Digest” section
    - 3–5 short highlights with strong/em emphasis
    - Optional “Editor’s Take”
    - Footer: “Generated automatically by the AI-Powered News Sentiment Dashboard.”

## Configuration tips
- NewsAPI:
  - Change category, pageSize, or add country/language params as needed.
- Relevance threshold:
  - Adjust in the Filter node (default 90). Lower it if you receive empty digests.
- Email:
  - Update recipient, subject, and optionally connect a different SMTP provider.
- Scheduling:
  - Replace the Manual Trigger with a Cron node to run daily.

## Limitations and notes
- If the model returns malformed JSON, the workflow attempts to clean and parse it; failures are captured with an error field and will yield nulls.
- With a high relevance threshold, some runs may produce few or no items.
- Only excerpts (not full text) are analyzed to reduce token usage.
- Respect NewsAPI’s terms and rate limits; ensure you use your own API key and do not commit it to source control.

## Getting started
1. Add credentials: OpenAI API, Gmail OAuth2, and your NewsAPI key.
2. Update the HTTP Request URL to reference your NewsAPI key securely.
3. Set the Gmail “sendTo” address.
4. Test with Manual Trigger; then add a Cron node for automated daily delivery.
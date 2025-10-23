# AI‑Powered News Sentiment Dashboard (Tech / Media)

A ready‑to‑run n8n workflow that pulls the latest technology headlines, analyzes each article’s sentiment and relevance with OpenAI, assembles a concise digest, and emails a polished HTML briefing to your inbox.

## What it does
- Fetches top technology headlines from NewsAPI (page size 20).
- Normalizes article fields and builds a clean excerpt.
- Iterates through each article and uses OpenAI to return strict JSON:
  - sentiment: positive | negative | neutral
  - summary: 1‑line synopsis
  - relevanceScore: 0–100 (how relevant to technology trends)
- Merges AI results with the original article data.
- Filters to only highly relevant items (relevanceScore ≥ 90 by default).
- Formats highlights, aggregates them into a digest, and asks OpenAI to compose a complete HTML email.
- Sends the HTML email via Gmail with a timestamped subject.

## Prerequisites
- n8n instance
- Credentials:
  - NewsAPI API key
  - OpenAI API credentials (used by the “OpenAI Chat Model” nodes)
  - Gmail OAuth2 credentials (for sending the email)

## Key inputs and configuration
- News source:
  - Endpoint: https://newsapi.org/v2/top-headlines
  - Query: category=technology, pageSize=20
  - Add or adjust parameters (e.g., country=us) as needed.
  - Place your NewsAPI key in the HTTP Request node (avoid hardcoding secrets).
- OpenAI models:
  - gpt-4.1-mini (configured in both OpenAI Chat Model nodes)
- Relevance threshold:
  - Filter node keeps articles with relevanceScore ≥ 90 (change to tune volume).
- Email:
  - Recipient: set in the “Send a message” (Gmail) node.
  - Subject: “AI News Digest – {{ $now }}”
  - Output is full HTML (cleaned before sending).

## Outputs
- Per‑article enriched JSON (internal to the workflow):
  - title, url, source, publishedAt, excerpt
  - sentiment, summary, relevanceScore
- Digest:
  - Aggregated, formatted highlights for only the most relevant stories.
- Email:
  - A styled, production‑ready HTML briefing suitable for Gmail/Outlook/Apple Mail.

## How it works (node overview)
1. Manual Trigger: Starts the workflow on demand.
2. HTTP Request: Fetches top tech headlines from NewsAPI.
3. Code (expand articles): Converts the API response into one item per article.
4. Set (Edit Fields): Normalizes fields and builds an excerpt (content/description/title fallback).
5. Split in Batches (Loop Over Items): Iterates articles one at a time.
6. OpenAI Agent – Sentiment Analysis: Prompts OpenAI to return strict JSON with sentiment, summary, and relevanceScore.
7. Code (parse AI JSON): Cleans and parses the AI’s JSON safely.
8. Merge (by position): Combines original article data with AI analysis.
9. Filter: Keeps only items with relevanceScore ≥ 90.
10. Code (format summaries): Produces readable snippets for each kept article.
11. Aggregate + Code (join): Collates snippets into a single digest string.
12. OpenAI Agent (HTML email): Turns the digest into a complete, styled HTML email.
13. Code (HTML cleanup): Strips any markdown fences to ensure pure HTML.
14. Gmail: Sends the HTML email to the configured recipient.

Note: The “Replace Me” NoOp node is used to complete the Split in Batches looping pattern.

## Setup steps
1. Import the workflow JSON into n8n.
2. Configure credentials:
   - HTTP Request node: Insert your NewsAPI key (consider using environment variables).
   - OpenAI nodes: Attach your OpenAI credentials.
   - Gmail node: Connect Gmail OAuth2 and set the recipient address.
3. Optional tuning:
   - Adjust NewsAPI query (country, pageSize).
   - Change the relevance threshold in the Filter node.
   - Edit email subject or styling if desired.
4. Run:
   - Click “Execute Workflow” or schedule it via n8n.

## Notes and considerations
- Respect API rate limits and terms (NewsAPI, OpenAI, Gmail).
- The parser cleans common AI formatting issues to enforce valid JSON, but you can tighten prompts if needed.
- Security: Avoid committing API keys. Use n8n credentials or environment variables.
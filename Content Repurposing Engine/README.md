Content Repurposing Engine — README

Overview
This Make.com workflow monitors a YouTube channel, pulls the latest video transcript, repackages it into multi-platform content using OpenAI, and appends the results to a Google Doc.

How it works
1) Poll YouTube for new videos
- Module: youtube:pollingVideos
- Filters: channelId set; publishedAfter 2025-08-01; limit 1 per run
2) Get transcript
- Module: dumplingai:getYoutubeTranscript
- Builds URL from the video ID and fetches the transcript in English (no timestamps)
3) Generate repurposed content
- Module: openai-gpt-3:CreateCompletion (chat)
- Model: gpt-3.5-turbo
- Produces: blog summary, 5 tweets, 1 LinkedIn post, 1 Instagram caption (clearly labeled)
4) Save to Google Docs
- Module: google-docs:appendADocument
- Appends the AI output to the body of a specified document

Inputs to configure
- YouTube
  - channelId: the channel to monitor
  - publishedAfter / publishedBefore: date window for polling
  - limit: number of videos to process per run (default 1)
- Transcript (Dumpling AI)
  - preferredLanguage: en
  - includeTimestamps: false
  - timestampsToCombine: 5
- OpenAI
  - model: gpt-3.5-turbo (editable)
  - temperature: 1
  - top_p: 1
  - max_tokens: 2048
- Google Docs
  - document: target Document ID
  - destination: drive (My Drive)
  - insert mode: append to document body

Outputs
- A single text block appended to the Google Doc containing:
  - [Blog Post] 300–400 word summary
  - [Tweets] 5 tweet-sized highlights
  - [LinkedIn Post] 1 post (2–3 short paragraphs)
  - [Instagram Caption] 1 caption with 3–5 hashtags

Connections required
- YouTube account connection
- Dumpling AI API connection
- OpenAI API connection
- Google account connection (Docs)

Scheduling and run behavior
- This scenario is not instant; schedule it in Make.com as needed.
- Processes up to the configured limit of videos per execution.
- Environment: eu2.make.com

Notes and tips
- Update publishedAfter to control the starting point for new content.
- If a video lacks an accessible transcript (e.g., live streams or captions disabled), the AI step may produce empty or low-quality results; consider adding error handling or filters.
- Replace the hardcoded Document ID with your own target doc.
- You can tweak the AI prompt, model, and temperature to change tone and verbosity.
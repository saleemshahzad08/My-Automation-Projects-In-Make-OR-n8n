# AI Research Paper Companion (n8n Workflow)

A two-part workflow that:
1) ingests a research paper PDF, extracts its title/abstract, generates a student-friendly summary and 5–10 flashcards, stores them in Google Sheets, and emails a styled digest; and
2) powers a quiz chatbot that delivers questions from the sheet, auto-grades answers, and returns a final score.

## What it does
- Accepts a PDF via webhook, extracts text, and uses GPT-4.1-mini to:
  - parse Title + Abstract
  - create a Plain-language Summary, Conceptual Explanation, and Key Insights
  - generate 5–10 flashcards (Question, Answer)
- Stores the flashcards in Google Sheets (QA-Database).
- Sends a full HTML email with the summary and a CTA that includes a local URL to launch a quiz UI.
- Exposes a second webhook for a chatbot flow:
  - “start” returns greeting + first question
  - user answers are graded by LLM
  - sheet is updated (IsAnswered, IsCorrect)
  - returns next question or final score

## Key services and credentials
- OpenAI (gpt-4.1-mini)
- Google Sheets (append/update/get rows)
- Gmail (send HTML email)

## Inputs

1) Webhook: PDF ingestion
- Method/Path: POST /20bc1329-1442-41cc-a076-6d3e1ccb4e4a
- Expected payload: binary file under key paperFile
- Behavior:
  - Extracts PDF text
  - Cleans text
  - GPT: returns Title + Abstract as JSON
  - GPT: returns summary sections
  - GPT: returns 5–10 flashcards (QuestionID, Question, Answer, IsAnswered=false)
  - Google Sheets: appends rows to QA-Database (Sheet1)
  - Gmail: sends full HTML digest email with a CTA

2) Webhook: Quiz chatbot
- Method/Path: POST /500d3ad5-e654-44bd-a70a-334d696f3691
- Expected JSON: { "user_message": "start" | "<user answer>" }
- Behavior:
  - If user_message == "start": returns greeting + first question
  - Else: grades the given answer against CorrectAnswer (LLM), updates sheet, returns feedback + next question
  - When all questions answered: returns final score

## Google Sheets schema (QA-Database → Sheet1)
- QuestionID (string)
- QuestionText (string)
- CorrectAnswer (string)
- IsAnswered (string/boolean; grading sets True)
- IsCorrect (string/boolean; True when answer is correct)

Note: The workflow reads and writes IsAnswered/IsCorrect as strings or booleans; scoring treats TRUE/True/true as correct.

## Outputs

1) From PDF ingestion webhook
- Immediate JSON response: { "status": "success", "message": "File received and renamed successfully. Processing initiated." }
- Side effects:
  - New/updated rows in Sheet1
  - Styled HTML email sent to the configured recipient

2) From quiz chatbot webhook
- When starting: { "response": "<greeting + first question (escaped newlines)>" }
- During quiz: { "response": "<feedback + next question (escaped newlines)>" }
- When complete: { "response": "Quiz Complete! ... (escaped newlines)", "quiz_finished": true }

## Email digest
- Fully inlined HTML (friendly design)
- Includes the generated summary
- CTA section uses a raw, non-clickable local URL:
  file:///C:/Users/FINE/Desktop/research_chatbot_site.html
- Instructions tell the user to copy-paste the URL into the browser

## Configuration notes
- OpenAI model: gpt-4.1-mini (update in LM nodes if needed)
- Gmail recipient: configured in “Send a message”
- Google Sheet: QA-Database (docId: 14DJFHnM7zBgecgeDv-r4gQ1Xm3glIJnf8Y0RVv5Ovwc, Sheet1)
- Local URL in email CTA: update if your file path differs
- Webhook paths are UUID-based; adjust if deploying behind a gateway
- The ingestion flow expects the uploaded file under paperFile; ensure upstream client uses that key

## Limitations and tips
- The CTA link is local and non-clickable by design; users must copy it.
- Ensure Sheet1 exists with the columns listed above before running.
- The workflow escapes newlines in webhook responses for safe JSON transport.
- If you change column names or types, update all Google Sheets nodes and scoring logic.
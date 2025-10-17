# AI Recruitment Assistant

Automates candidate intake from Google Forms, evaluates applicants with OpenAI, updates a Google Sheet with AI results, and sends tailored emails to candidates and a manager summary.

## What it does

1. Capture new Google Forms submissions.
2. Append candidate data to a Google Sheet (Sheet1).
3. Read recent rows and evaluate each candidate with an AI rubric:
   - Skills match (40%), experience (30%), education (10%), communication quality (20%).
   - Produce a total score, concise justification, and a recommendation.
4. Update the sheet with:
   - K: AI Score (0–100)
   - L: AI Recommendation
   - M: AI Comments
5. Route by score:
   - Score ≥ 75: Shortlist
     - Send interview-invitation email to the candidate.
     - Build an HTML table of shortlisted candidates for a hiring manager summary.
   - 50 ≤ Score < 75: Hold/Phone Screening
     - Build an HTML table of on-hold candidates for the hiring manager summary.
   - Score < 50: Reject
     - Send a polite rejection email to the candidate.
6. Email a manager digest containing two HTML tables:
   - “Shortlisted Candidates”
   - “Candidates On Hold”

## Inputs

- Google Form responses (mapped fields):
  - Full Name
  - Email Address
  - Phone No.
  - Applying For the Position Of
  - Years of Experience
  - Key Skills
  - Education / Degree
  - Why are you applying
  - Cover Letter
- Google Sheet:
  - Spreadsheet: Candidates (ID: 14y13QZYzjp7h_6VquGepn2CavKzKjGDsCwnSK3NhZqg)
  - Sheet: Sheet1
  - Headers present (A:Z)
- OpenAI connection (model: gpt-4.1-2025-04-14; see Notes).
- Email connection (Google Restricted) to send:
  - Candidate notifications (shortlist/rejection)
  - Manager digest to gc.jbd.alumni.office@gmail.com

## Sheet schema (Sheet1)

- A: Timestamp
- B: Candidate Name
- C: Email
- D: Phone
- E: Job Role
- F: Experience in Years
- G: Skills
- H: Education
- I: Motivation
- J: Cover Letter
- K: AI Score (0–100)
- L: AI Recommendation
- M: AI Comments
- N–AA: Reserved for future use

New submissions populate columns A–J. AI writes to K–M.

## AI evaluation output (internal JSON)

The OpenAI step returns JSON per candidate, including:
- Row_No (sheet row for update)
- candidate_name
- Email
- Interview_Date (currently fixed)
- job_role
- scores: skills_match, relevant_experience, education, communication_quality
- total_score (0–100)
- justification (≤ ~300 chars)
- recommendation: one of “Reject”, “Hold/Phone Screening”, “Shortlist for Interview”

This JSON is parsed and used to update the Sheet and trigger the appropriate email flow.

## Routing thresholds

- Shortlist: total_score ≥ 75
- Hold/Phone Screening: 50 ≤ total_score < 75
- Reject: total_score < 50

## Outputs

- Google Sheets
  - One new row per form submission (A–J).
  - AI results written to K–M for the evaluated row.
- Candidate emails
  - Shortlisted: HTML invitation email with interview date.
  - Rejected: plain-text polite rejection email.
- Manager digest email
  - Subject: “Recruitment Update – Candidate Status”
  - To: gc.jbd.alumni.office@gmail.com
  - Body: two inline-styled HTML tables (Shortlisted and On Hold), built from the latest up to 5 entries for each category.

## Configuration

- Google Forms trigger
  - Module: google-forms:watchResponses
  - formId: 1BB0EzuDZwcTfZV4K_i6xjZ01iWSFzU3hPLw1sjwctkw
- Google Sheets
  - Spreadsheet: Candidates (ID above), Sheet: Sheet1, Headers included.
- OpenAI
  - Model in modules 6, 21, 33, 52, 10. If the specified model name is not available in your account, select a currently supported GPT-4.x model.
- Email
  - Uses a Google Restricted connection.
  - Shortlist email is HTML; rejection email is plaintext.
  - Manager email address is currently hard-coded (gc.jbd.alumni.office@gmail.com).

## Customization tips

- Interview date
  - Currently hard-coded in the AI prompt (“30/09/2025”). Update in the OpenAI evaluation module to a dynamic or current date as needed.
- Thresholds
  - Adjust the numeric conditions in the router filters to change Shortlist/Hold/Reject bands.
- Manager digest volume
  - The digest pulls up to 5 rows for each category. Increase/decrease the limit in the relevant filterRows modules (IDs 24 and 31).
- Email templates
  - Edit the AI prompts in the invitation (ID 21) and rejection (ID 10) modules for tone or formatting.
  - Change the manager email recipient in module ID 55.
- Reliability
  - If JSON parsing fails in modules 7/22/11, consider switching the OpenAI “response_format” to json_object and tightening prompts to “return only JSON”.

## Notes

- Processing limits
  - Initial watcher limit is set to 3; adjust if you expect higher volume.
- Costs and quotas
  - OpenAI and Google API usage incur costs and are subject to rate limits.
- Privacy
  - This workflow processes PII (names, emails, resumes/letters). Ensure compliance with your data protection policies and obtain consent as needed.
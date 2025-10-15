# AI Recruitment Assistant

## Purpose

The **AI Recruitment Assistant** automates the recruitment workflow by integrating Google Forms, Google Sheets, OpenAI (GPT-4), and email. It streamlines candidate evaluation, shortlisting, notification, and status reporting for hiring managers and applicants.

---

## Workflow Summary

1. **Capture Applications:** Automatically collects new job applications from a Google Form.
2. **Store Data:** Adds candidate details to a Google Sheet ("Candidates").
3. **AI Evaluation:** Uses OpenAI to score candidates (0–100) based on skills, experience, education, and communication, then generates recommendations (Shortlist, Hold, or Reject) with comments.
4. **Update Records:** Writes AI scores, recommendations, and comments back to the candidate's row in the sheet.
5. **Email Notifications:**
   - Sends personalized interview invitations to shortlisted candidates.
   - Sends polite rejection emails to unqualified applicants.
6. **HR Reporting:** Creates and emails neat HTML tables (ready for Google Docs) summarizing "Shortlisted" and "On Hold" candidates to the hiring manager.

---

## Workflow Steps

1. **Collect Responses**
   - **Input:** Google Form responses (fields include name, email, phone, job role, experience, skills, education, motivation, cover letter).
   - **Output:** Parsed values for further processing.

2. **Record to Spreadsheet**
   - **Input:** Parsed form response data.
   - **Output:** New row added to the "Candidates" Google Sheet.

3. **Candidate Filtering**
   - Filters latest candidates for evaluation.

4. **AI Candidate Assessment**
   - **Input:** Candidate's parsed details.
   - **Output:** JSON containing:
     - Row number, candidate name, email, job role, interview date
     - Score breakdown (skills match, experience, education, communication)
     - Total score (0–100)
     - Short justification
     - Recommendation ("Reject", "Hold/Phone Screening", "Shortlist for Interview")

5. **Update Spreadsheet with AI Evaluation**
   - Writes AI output (score, recommendation, comments) to the correct row in the Google Sheet.

6. **Branching – Actions Based on Recommendation**
   - **Shortlist (Score ≥ 75):**
     - Sends a formatted interview invitation email.
     - Includes instructions and interview date.
     - Updates candidate status and triggers HR status report.
   - **Hold (Score 50–74):**
     - Updates candidate as "On Hold".
     - Candidate included in on-hold reporting, no immediate email is sent.
   - **Reject (Score < 50):**
     - Sends a polite rejection email, using AI-generated justification.
     - Updates candidate status.

7. **Generate and Send HR Reports**
   - Prepares HTML tables (inline CSS) of shortlisted and on-hold candidates.
   - Emails these to the hiring manager for easy tracking and documentation.

---

## Inputs

- **Google Form Responses:** Candidate details.
  - Full Name
  - Email Address
  - Phone Number
  - Applying For Position
  - Years of Experience
  - Key Skills
  - Education/Degree
  - Motivation ("Why are you applying?")
  - Cover Letter

- **Google Sheet:** Destination for candidate data and ongoing status.

- **AI Model:** Scoring rubric and prompt instructions, managed within the workflow.

---

## Outputs

- **Google Sheet Updates:**
  - Adds new candidate rows.
  - Updates with AI scores, recommendations, and comments.

- **Candidate Emails:**
  - **Shortlisted:** Personalized interview invitations (HTML).
  - **Rejected:** Empathetic rejection emails (text).

- **HR Email Report:**
  - HTML tables (ready for copy-pasting to Google Docs) summarizing:
    - Shortlisted candidates (Name, Job Role, AI Score, AI Comments)
    - On-hold candidates

---

## Key Data Flow

1. **Input:** Form submission (by candidate)
2. **Processing:** Data written to sheet → filtered → scored by GPT-4 → sheet updated
3. **Outcomes:**
   - **Shortlist:** Invitation email + included in HR report
   - **Hold:** Included in HR report
   - **Reject:** Rejection email

---

## Extending or Customizing

- **Editing the Google Form:** Add/remove questions as desired. Ensure sheet mapping matches.
- **Changing the Scoring Rubric:** Update the OpenAI prompt in the workflow.
- **Email Content:** Modify the AI prompt for emails for tone/branding changes.
- **Reporting:** Edit HTML table columns or style in prompts.

---

## Technical Requirements

- Connected Google Workspace account (Forms, Sheets, Gmail)
- OpenAI API access (GPT-4)
- Access and permissions for Make.com (or compatible workflow automation platform)

---

## Sheet
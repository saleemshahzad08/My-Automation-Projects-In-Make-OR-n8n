# AI Recruitment Assistant – Workflow Overview

## Purpose

The **AI Recruitment Assistant** is an automated workflow that streamlines candidate evaluation and communication for job recruitment using Google Forms, Google Sheets, and OpenAI GPT-4. It handles the intake, AI-based assessment, shortlisting, rejection, and notification (email) of job applicants, while keeping stakeholders informed through status update emails.

---

## How It Works

1. **Collects candidate applications** via Google Forms.
2. **Records responses** in a Google Sheet.
3. **Evaluates candidates** using OpenAI GPT-4, auto-scoring based on:
   - Skills Match (40%)
   - Relevant Experience (30%)
   - Education (10%)
   - Communication Quality (20%)
4. **Categorizes candidates**:
   - **Shortlist for Interview** (score ≥75)
   - **Hold/Phone Screening** (score 50–74)
   - **Rejected** (score <50)
5. **Automates Notifications**:
   - Sends customized interview invitations or rejection emails to candidates.
   - Notifies HR by email with up-to-date status tables for "Shortlisted" and "On Hold" candidates, stylized for easy pasting into Google Docs.

---

## Inputs

### 1. Google Form Responses

**Fields Extracted:**
- Full Name
- Email Address
- Phone Number
- Position Applied For
- Years of Experience
- Key Skills
- Education / Degree
- Motivation (“Why are you applying?”)
- Cover Letter

### 2. Google Sheets

- **Sheet:** `Sheet1` in an existing "Candidates" spreadsheet
- **Columns Mapped:**
  - Timestamp
  - Candidate Name
  - Email
  - Phone
  - Job Role
  - Experience (Years)
  - Skills
  - Education
  - Motivation
  - Cover Letter
  - AI Score (0–100)
  - AI Recommendation
  - AI Comments

### 3. OpenAI GPT-4

- Used to interpret candidate data, assign rubric-based scores, and generate justifications and recommendations as JSON objects.

---

## Outputs

### Automated AI Evaluation
- Each candidate receives:
  - Sub-scores per rubric area
  - Total score (0–100)
  - Short justification comment (max 300 chars)
  - AI Recommendation (Reject, Hold/Phone Screening, Shortlist for Interview)

### Updates to Google Sheet
- Candidate rows are appended and updated with AI evaluation details.

### Candidate Communication (via Email)
- **Shortlisted:** Personalized HTML interview invitation.
- **Rejected:** Personalized plain text rejection letter with reason.
- **On Hold:** No direct email, but included in status update table for HR.

### HR/Stakeholder Email Report
- Regular status email with:
  - Table of **Shortlisted** candidates (Name, Role, AI Score, AI Comments)
  - Table of **On Hold** candidates (same fields)
- Tables are well-formatted HTML for simple copy-paste into Google Docs.

---

## Automation Logic

- **Scores ≥75:** Shortlisted
  - Candidate notified by email.
  - Listed in "Shortlisted Candidates" table for HR.
- **Scores 50–74:** On Hold
  - No email to candidate.
  - Listed in "On Hold Candidates" table for HR.
- **Scores <50:** Rejected
  - Candidate receives personalized rejection email.
  - Not included in HR tables.

---

## Requirements / Connections

- **Google Account** with access to:
  - The relevant Google Form (Form ID: `1BB0EzuDZwcTfZV4K_i6xjZ01iWSFzU3hPLw1sjwctkw`)
  - The "Candidates" Google Sheet (`14y13QZYzjp7h_6VquGepn2CavKzKjGDsCwnSK3NhZqg`)
- **OpenAI API Key** (for GPT-4 evaluation and email/HTML generation)
- **Email Service** (Google Restricted or SMTP) for outbound notification

---

## Customization

- **Rubric and thresholds** can be adjusted in the OpenAI scoring prompts.
- **Email templates** (subject/body) and the HTML table styling can be edited within prompts or script mappings.
- **Spreadsheet IDs, Sheet Names, Form IDs** should be changed to match your setup.

---

## Typical Workflow Sequence

1. **New candidate submission** → Google Form.
2. **Response is logged** in Google Sheets.
3. **AI evaluates** candidate and writes scores/comments back to Sheet.
4. **Candidate is categorized:** Shortlist / Hold / Reject.
5. **Corresponding email sent** to candidate (Shortlist or Reject).
6. **HR receives update email
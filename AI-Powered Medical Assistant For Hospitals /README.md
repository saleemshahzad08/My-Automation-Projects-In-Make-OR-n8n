# AI‑Powered Medical Assistant for Hospitals (n8n Workflow)

A form-driven triage workflow that validates patient submissions, generates an AI-based preliminary assessment, classifies urgency, notifies clinicians and patients, and logs all cases to Google Sheets. Designed to assist—not replace—medical professionals.

## What it does
- Collects symptoms via a public form.
- Validates inputs (required fields, email, age, Pakistan phone format).
- Uses AI to:
  - Classify urgency (critical, moderate, mild).
  - Produce a structured preliminary assessment with possible diagnoses.
  - Generate a short natural-language summary.
  - Fill HTML email templates for doctors and patients.
- Routes cases:
  - Critical: immediate alerts, patient SMS/email, doctor approval flow, reminders, and escalation to admin/alternate doctor if unattended.
  - Non‑critical: standard emails to doctor/patient.
- Logs every case to Google Sheets for tracking and review.

## Trigger and Form
- Trigger: n8n Form Trigger at path: /medical-symptom-form
- Form title: Medical Symptom Assessment Form
- Disclaimer: Educational purposes only; not medical advice.
- Fields (all required):
  - Full Name (text)
  - Age (number; validated 0–150)
  - Gender (dropdown: Male, Female, Other)
  - Symptoms (textarea)
  - Duration of Symptoms (text, e.g., “3 days”)
  - Known Conditions or Medications (textarea; allow “none”)
  - Your Email Address (email; regex validated)
  - Contact No. (Pakistani format +923XXXXXXXXX)

## Validation
- Missing fields cause a hard error with a clear message.
- Email: simple regex.
- Contact No.: strict +923XXXXXXXXX.
- Age: 0–150.
- Data is trimmed and normalized; timestamp attached.

## AI Components (OpenAI gpt‑4.1‑mini)
- Urgency Determination
  - Input: patient data.
  - Output JSON: { urgency_level: "critical|moderate|mild", reason, recommended_action }
  - Safety: no diagnoses; conservative escalation.
- Diagnostician
  - Input: patient data.
  - Output JSON: { possible_diagnoses: [{name, reason}, ...], urgency, advice, disclaimer }
  - Safety: not a diagnosis; short reasoning; no treatments.
- Summary of Diagnoses
  - Converts structured diagnoses into a single 60–100 word paragraph.
- Email Composers
  - Two variants: Normal Cases and Critical Cases.
  - Fill predefined HTML templates and return JSON with four fields:
    - "doctor's email's subject"
    - "doctor's email's body"
    - "patient's email's subject"
    - "patient's email's body"

## Workflow Routing
- If urgency_level = critical:
  - Compose critical emails.
  - Send email + SMS to patient advising immediate care.
  - Send “send-and-wait” Gmail approval to designated doctor.
    - If approved: log as Reviewed = Yes.
    - If not within window: send SMS reminder + reminder email.
    - If still unattended: escalate via emails to admin and alternate doctor; log as Reviewed = Escalated.
- Else (moderate/mild):
  - Run Diagnostician + Summary.
  - Send standard assessment emails to doctor and patient.
  - Log to Google Sheets.

## Outputs
- Emails:
  - Patient: AI summary, urgency, advice, and disclaimer.
  - Doctor: patient details, AI summary/diagnoses, urgency, advice; approval button for critical cases.
- SMS (critical branch):
  - Patient: immediate action message.
  - Doctor: reminder if unreviewed.
- Google Sheets logging:
  - Appends/updates a row per case with:
    - Timestamp, Name, Age, Gender, Contact, Symptoms, Duration, Conditions, Email
    - AI Diagnosis (structured output or critical notice)
    - Urgency, Advice, Reviewed? (Yes/Escalated)

## Configuration (Workflow Configuration node + n8n Credentials)
Set these values before activating:
- doctorEmail: designated triage doctor email
- doctorContact: doctor’s mobile for SMS reminders
- spreadsheetId: Google Sheet ID
- sheetName: target sheet name (e.g., “Medical Cases”)

Required credentials (in n8n):
- Gmail OAuth2 (for sending/approval emails).
- OpenAI API.
- Google Sheets OAuth2.
- SMS provider API (configured HTTP request; replace with your provider, API key, sender ID, and templates).

Important:
- Do not hard‑code secrets in nodes; store in n8n credentials or environment variables.
- Replace any placeholder/template IDs and API keys with your own.

## Google Sheets Schema (typical columns)
- Timestamp
- Name
- Age
- Gender
- Contact
- Symptoms
- Duration
- Conditions
- Email
- AI Diagnosis
- Urgency
- Advice
- Reviewed?

## How to run
1. Import the workflow into n8n.
2. Configure credentials (Gmail, OpenAI, Google Sheets, SMS).
3. Open “Workflow Configuration” and set doctorEmail, doctorContact, spreadsheetId, sheetName.
4. Ensure the Google Sheet exists or let the workflow create/append with the above columns.
5. Publish the Form Trigger endpoint at /medical-symptom-form and test a submission.
6. Monitor executions and the Google Sheet for logs.

## Error handling and safeguards
- Strict validation on form inputs; invalid submissions fail early.
- AI responses are parsed as JSON; malformed responses raise errors.
- Critical path errs on the side of safety; emergency guidance is informational and non‑diagnostic.
- Disclaimers are included in patient communication.

## Privacy and compliance
- Patient data is transmitted via email/SMS and stored in Google Sheets. Obtain consent and limit access appropriately.
- Configure secure credentials storage in n8n and avoid exposing IDs/keys publicly.
- This system is not a medical device and does not provide medical diagnoses or treatments.
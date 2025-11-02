# AI-Powered Medical Assistant for Hospitals (n8n Workflow)

A triage and notification workflow that collects patient symptoms, validates inputs, classifies urgency with AI, generates a preliminary assessment, notifies staff and patients via email/SMS, and logs cases to Google Sheets. It supports escalation if a critical case isn’t reviewed promptly.

## What it does
- Collects patient data via an n8n Form Trigger.
- Validates required fields, email, age, and Pakistan-format phone numbers.
- Uses AI to:
  - Classify urgency: critical | moderate | mild.
  - Generate a structured preliminary assessment (possible diagnoses + advice + disclaimer).
  - Produce a short natural-language summary.
  - Compose HTML emails for doctors and patients.
- Routes cases:
  - Critical: immediate alerts to patient and doctor, reminder/escalation if unreviewed.
  - Non‑critical: sends informational emails to patient and doctor.
- Logs every case to Google Sheets.

## Inputs

1. Patient form fields
   - Full Name (required)
   - Age (number, 0–150, required)
   - Gender (Male | Female | Other, required)
   - Symptoms (textarea, required)
   - Duration of Symptoms (required)
   - Known Conditions or Medications (textarea, required)
   - Your Email Address (required; basic email format)
   - Contact No. (required; regex: +923XXXXXXXXX)
2. Workflow configuration (Set node)
   - doctorEmail
   - doctorContact
   - spreadsheetId
   - sheetName
3. Credentials (in n8n)
   - OpenAI API
   - Gmail OAuth2
   - Google Sheets OAuth2
   - SMS provider (SendPK) API key

Note: Replace example emails, phone numbers, spreadsheet IDs, and API keys with your own. Do not hardcode secrets; store them in n8n credentials or environment variables.

## Validation rules
- All listed fields are required.
- Email: basic regex check.
- Contact No.: +923######### format (Pakistan). Update regex in the “Validate Input” Code node for other locales.
- Age: integer 0–150.
- Errors stop execution with clear messages for missing/invalid values.

## AI components
- Urgency Determination (LLM)
  - Output JSON: { "urgency_level": "critical|moderate|mild", "reason": "...", "recommended_action": "..." }
  - Safety: never diagnoses; errs on side of caution.
- Diagnostician (LLM)
  - Output JSON: { possible_diagnoses: [{name, reason}, ...], urgency, advice, disclaimer }
  - Parsed and flattened for downstream nodes.
- Summary of Diagnoses (LLM)
  - Produces a 60–100 word paragraph summarizing possible diagnoses and reasons.
- Email Composers (LLM)
  - Two templates:
    - Normal cases (informational).
    - Critical cases (emergency style + “send and wait” approval).
  - Output a single JSON with four fields:
    - "doctor's email's subject", "doctor's email's body"
    - "patient's email's subject", "patient's email's body"

## Flow (high level)
1. Patient Symptom Form (path: /form/medical-symptom-form) → Workflow Configuration → Validate Input.
2. Urgency Determination → Parse Response of UD.
3. Branch on urgency_level:
   - Critical
     - Compose critical emails → Send urgent email to patient + SMS alert to patient.
     - Send urgent alert to designated doctor (send-and-wait; 2-minute timeout).
       - If approved: log “Reviewed? = Yes”.
       - If not approved: send SMS reminder to doctor → reminder email (send-and-wait).
         - If approved: log “Reviewed? = Yes”.
         - If still not approved: escalate via emails to Admin and Alternate Doctor; log as “Escalated”.
   - Non‑critical
     - Diagnostician → Parse → Summary of Diagnoses.
     - Compose normal-case emails → email to doctor and patient.
     - Log to Google Sheets.
4. All logs stored in the configured Google Sheet (appendOrUpdate).

## Outputs

1. Emails
   - Patient (normal): AI summary, urgency, advice, disclaimer.
   - Doctor (normal): patient details + AI summary, urgency, advice, disclaimer.
   - Critical case:
     - Patient: urgent guidance and emergency instructions.
     - Doctor: “send-and-wait” alert with approval button.
     - Reminder to doctor if no approval.
     - Escalation emails to Admin and Alternate Doctor if still unapproved.
2. SMS
   - Patient: emergency notice for critical cases.
   - Doctor: reminder SMS if initial alert unapproved.
3. Google Sheets logging (columns used)
   - Timestamp, Name, Age, Gender, Contact, Symptoms, Duration, Conditions, Email, Urgency, Advice, AI Diagnosis, Reviewed?
   - Critical escalations mark “Reviewed?” as “Escalated”.
   - Normal/Reviewed critical cases mark “Reviewed?” as “Yes”.

## Setup

1. Prerequisites
   - n8n instance with HTTPS
   - OpenAI API key
   - Gmail account with OAuth2 credentials in n8n
   - Google account with Sheets access + OAuth2 in n8n
   - SMS provider (SendPK) account and API key

2. Configure in n8n
   - Update “Workflow Configuration” (Set node):
     - doctorEmail
     - doctorContact
     - spreadsheetId
     - sheetName
   - Add credentials:
     - OpenAI API → used by “OpenAI Chat Model*” nodes
     - Gmail OAuth2 → used by all Gmail nodes
     - Google Sheets OAuth2 → used by “Logging Record of Reviewed Cases” nodes
     - SMS API key → used by “Sending SMS Alert To Patient” and “Sending Reminder Using SMS”
   - Form URL: https://YOUR-N8N-HOST/form/medical-symptom-form

3. Optional changes
   - Phone regex: adapt for your region in the “Validate Input” Code node.
   - Approval wait time: adjust in Gmail “sendAndWait” nodes.
   - Email templates: keep structure/CSS; only adjust placeholders in the Email Composer prompts.

## Operational notes
- The workflow expects AI nodes to return valid JSON strings; parsing nodes will fail if malformed.
- Use n8n’s Execution view to troubleshoot validation or parsing errors.
- Do not rely on AI output for clinical decisions. All emails include disclaimers; ensure your organization’s compliance and consent policies are followed.

## Disclaimer
This system provides educational, preliminary assessments and urgency classification. It does not offer medical diagnoses or treatment. Always involve licensed healthcare professionals for clinical decisions.
# Client Proposal & Invoice Drafting Assistant

Automates creation and delivery of a client proposal and invoice from a Google Form submission. It drafts a professional proposal with OpenAI, extracts invoice-ready details, builds a Google Docs invoice from a template, converts it to PDF, and emails the client the proposal text with the invoice attached.

## How it works (flow)
1. Trigger: Watch Google Form responses
   - Form fields expected:
     - Client Name
     - Client Email
     - Project Description
     - Deliverables
     - Timeline
     - Price/Rate
     - Any Extra Notes

2. Proposal drafting (OpenAI)
   - Uses GPT-3.5-turbo to generate a persuasive proposal using the form inputs.

3. Parallel routes
   - Route A: Invoice generation
     - Extracts structured JSON from the proposal (client_name, client_email, project_description, deliverables as list, timeline, price, extra_notes, invoice_number INV-XXXX, issue_date, due_date).
     - Parses JSON.
     - Creates a Google Doc from a template (invoice) and fills placeholders.
     - Converts the created Google Doc to PDF via PDF.co.
     - Stores the PDF download URL and a job/content ID in variables.

   - Route B: Email delivery
     - Stores the proposal text in a variable.
     - Retrieves the proposal text and the PDF URL.
     - Downloads the PDF and emails it to the client email captured from the form.
     - Subject: “Proposal and Invoice”
     - Body: Proposal text
     - Attachment: Generated invoice PDF

## Inputs
- Trigger: New response in the specified Google Form (Form ID).
- Required Google Form fields (mapped internally by question IDs):
  - Client Name (ID: 0c572819)
  - Client Email (ID: 225fd079)
  - Project Description (ID: 4a3434c5)
  - Deliverables (ID: 2df7ba18)
  - Timeline (ID: 761aedf8)
  - Price/Rate (ID: 05c08238)
  - Any Extra Notes (ID: 054d390b)

## Outputs
- Email sent to the client with:
  - Body: Drafted proposal text
  - Attachment: Invoice PDF
- Google Drive:
  - New invoice Google Doc created from template, titled by the generated invoice number
  - A downloadable PDF version of the invoice

## Services and connections
- Google Forms (trigger)
- OpenAI (GPT-3.5-turbo) for:
  - Proposal drafting
  - Structured JSON extraction
- Google Docs for templated invoice creation
- PDF.co for document-to-PDF conversion
- Gmail (restricted) for sending the email with attachment

## Configuration
1. Connect accounts:
   - Google (Forms, Docs, Drive)
   - OpenAI
   - PDF.co
   - Gmail (restricted/IMAP/SMTP as configured)

2. Set resource IDs:
   - Google Form ID: formId
   - Google Docs template ID: document
   - Destination Google Drive folder ID: folderId

3. Ensure template placeholders in the Google Doc match the fields used in the create-from-template step:
   - invoice_number
   - issue_date
   - due_date
   - client_name
   - client_email
   - project_description
   - deliverables
   - price
   - extra_notes

4. Optional tweaks:
   - Update the OpenAI prompts or model.
   - Adjust PDF.co options (expiration, async mode).
   - Change email subject/content or recipients (currently uses Client Email from the form).

## Field generation logic (invoice JSON)
- invoice_number: INV-XXXX (4 random digits)
- issue_date: Today’s date (YYYY-MM-DD)
- due_date: 30 days after issue_date
- deliverables: Extracted as a list of bullet points (if possible)
- If a field is missing in proposal text, it is left as an empty string.

## Notes and limitations
- Both routes run after the proposal is generated; variables are used to pass the PDF URL between routes.
- If JSON extraction fails or is not valid JSON, the parse step will fail and downstream steps (invoice/PDF/email) will not complete.
- The Google Doc template must include the exact placeholders listed above or values will not populate as expected.
- The router executes both branches; ensure the variable-based dependency (PDF URL) is available before the email step in your environment. If race conditions occur, consider placing the email step after the PDF is created in the same route or adding a synchronization step.

## Maintenance tips
- When updating the Google Form, confirm the question IDs still map correctly to:
  - 0c572819 (Client Name)
  - 225fd079 (Client Email)
  - 4a3434c5 (Project Description)
  - 2df7ba18 (Deliverables)
  - 761aedf8 (Timeline)
  - 05c08238 (Price/Rate)
  - 054d390b (Any Extra Notes)
- Validate the Google Docs template and folder permissions for the connected account.
- Monitor PDF.co conversion links expiration and adjust as needed.
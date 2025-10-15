Client Proposal & Invoice Drafting Assistant — README

Purpose
Automates client proposal and invoice creation from a Google Form submission. It drafts a professional proposal, generates an invoice from a Google Docs template, converts it to PDF, and emails the client with the proposal text and attached invoice.

How it works
- Trigger: Watches a Google Form for new responses (Form ID: 1_eHINokTpe2afTcLRcPAVBYhd6rEgPl-IM2hX0S-YMk).
- Draft proposal: Uses OpenAI (gpt-3.5-turbo) to write a proposal based on the form answers.
- Structure data: Extracts key details from the proposal into JSON (client_name, client_email, project_description, deliverables list, timeline, price, extra_notes, invoice_number, issue_date, due_date).
  - invoice_number format: INV-XXXX (random 4 digits)
  - issue_date: today (YYYY-MM-DD)
  - due_date: 30 days after issue_date
- Create invoice: Populates a Google Docs template and saves a new doc in a Drive folder.
- Convert to PDF: Uses PDF.co to produce a downloadable PDF link.
- Email: Sends an email to the client with the proposal text in the body and the PDF invoice attached.

Inputs

Form fields required (from Google Forms)
- Client Name
- Client Email
- Project Description
- Deliverables
- Timeline
- Price/Rate
- Any Extra Notes

Service connections (Make.com)
- Google (Forms/Docs/Drive)
- Gmail (to send the email)
- OpenAI
- PDF.co

Template and storage
- Google Docs template ID: 1C8Xv11XdD3040-Z4C0NVUd2xYygihDsKTRuR6oyyw5E
- Destination folder ID: 1Pezwi6IukW_y-BfWEMrlhIEeHOdqrgMW

Template placeholders (must exist in the Google Doc)
- invoice_number
- issue_date
- due_date
- client_name
- client_email
- project_description
- deliverables (list; ensure the template can display multiline/bulleted content)
- price
- extra_notes

AI settings
- Model: gpt-3.5-turbo
- Max tokens: 1000
- Temperature: 1.0
Notes: Form answers and proposal text are sent to OpenAI and PDF.co.

Outputs
- Proposal: Generated proposal text (included in the email body).
- Invoice (Google Doc): A new document titled with the invoice number, saved in the specified Drive folder.
- Invoice (PDF): Downloadable URL from PDF.co (temporary link) plus a PDF file fetched and attached to the email.
- Email to client: Subject “Proposal and Invoice,” sent to the Client Email from the form, with proposal text and the invoice PDF attached.

Operational flow summary
1) Google Forms response → 2) OpenAI generates proposal → 3) OpenAI structures JSON for invoice → 4) Google Docs fills template and creates invoice → 5) PDF.co converts to PDF and provides link → 6) Gmail sends proposal + PDF to the client.

Setup checklist
- Replace the Form ID with your form (or ensure the referenced form exists).
- Confirm Google, Gmail, OpenAI, and PDF.co connections in Make.
- Ensure the Google Docs template has the exact placeholders listed above.
- Verify the Drive folder ID for where invoices should be saved.
- Test with a sample form submission.
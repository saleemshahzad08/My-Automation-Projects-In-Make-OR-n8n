# AI-Automated Customer Service Based On Email

This workflow automates the process of responding to customer emails using AI. It reads incoming emails, classifies their intent, selects an appropriate response template from Google Sheets, generates a tailored reply with OpenAI’s GPT, and sends the response back to the customer.

---

## Purpose

Efficiently handle customer queries received via email by:
- Automatically classifying email type (Sales Enquiry, Customer Feedback, Complaint)
- Selecting a relevant prompt/response pattern from a Google Sheet based on classification
- Using AI to generate a personalized response
- Sending the AI-crafted reply back to the customer

---

## Workflow Steps

1. **Trigger on New Email**:
    - Checks the INBOX for unread emails.
2. **Classify Email with OpenAI**:
    - Uses GPT-3.5-Turbo to categorize the email as one of: Sales Enquiry, Customer Feedback, or Complaint.
3. **Lookup Template in Google Sheets**:
    - Finds a matching row in a Google Sheet ("Customer Feedback", Sheet1) using the predicted category.
4. **Generate Response with OpenAI**:
    - Feeds the found template (Prompt Sample) and original email to GPT to generate a tailored reply.
5. **Send Reply via Email**:
    - Sends the AI-generated response to the original sender, as a reply to the received email.
6. **Sleep/Pause** (optional utility step for rate limiting).

---

## Inputs

| Input                        | Source                      | Description                                                              |
|------------------------------|-----------------------------|--------------------------------------------------------------------------|
| Email Account                | Connected email provider    | For fetching and sending emails.                                         |
| Google Sheet                 | Specified Spreadsheet ID    | Contains predefined prompt templates keyed by category.                   |
| OpenAI API Key/Account       | OpenAI                      | To access GPT-3.5-Turbo for classification & response generation.         |

### Email Trigger Parameters:
- **Folder:** INBOX (can be changed)
- **Criteria:** UNSEEN (triggers only on unread emails)
- **Max Results:** 1 (per execution trigger)

---

## Outputs

| Output                       | Description                                                            |
|------------------------------|------------------------------------------------------------------------|
| Categorized Email            | The category assigned to each incoming email.                           |
| Tailored Reply Email         | An email sent back to the original sender with the personalized AI reply.|
| (Optional) Logging/Tracking  | (Through Google Sheets or logs in your email provider/Make.com account) |

---

## How It Works (Overview)

1. **Detection:** Picks up the latest unread email in your INBOX.
2. **Categorization:** Passes email content to OpenAI to decide if the email is a Sales Enquiry, Customer Feedback, or Complaint.
3. **Template Selection:** Searches a Google Sheet for a template/sample reply matching the category.
4. **Response Generation:** Sends both the template and the original email text to OpenAI to create a contextual, personalized reply.
5. **Response Delivery:** Sends the generated reply email to the original sender, preserving the original subject (as a reply).
6. **(Pause):** Optionally sleeps for 30 seconds (could be for pacing/rate limiting).

---

## Requirements

- **Email Service**: Gmail or IMAP-compatible provider (with connection configured)
- **OpenAI Account & API Key**: For access to GPT-3.5-Turbo
- **Google Account**: With access to the specified Google Sheet containing reply templates

---

## Customization

- **Categories**: You can edit the prompt and sheet to add or adjust categories.
- **Templates**: Google Sheet ("Customer Feedback") can have multiple rows with sample prompts for each category.
- **Reply Content**: Adjust sheet templates, OpenAI temperature, or prompt configuration for tone/polish.

---

## Getting Started

1. **Connect your email account**, OpenAI account, and Google Sheets as required.
2. **Prepare your Google Sheet** ("Customer Feedback") with columns such as:
    - A: Category (e.g., Sales Enquiry, Customer Feedback, Complaint)
    - B: Reply prompt/template for that category
3. **Deploy and run the scenario.**

---

## Limitations

- Only processes one unread email per run (configurable).
- Only supports three fixed categories (expandable via prompt and sheet).
- Reliant on accuracy of AI classification and quality of sheet prompts.

---

## Example Google Sheet Layout

| A               | B                                              |
|-----------------|------------------------------------------------|
| Sales Enquiry   | Thank you for your interest...                 |
| Customer Feedback | We appreciate your feedback...               |
| Complaint       | We're sorry to hear about your experience...   |

---

## Contact

For support or more information, consult your Make.com scenario designer or automation lead.

---

*Last updated: 202
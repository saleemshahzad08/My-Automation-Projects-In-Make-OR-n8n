```markdown
# AI-Automated Customer Service Based On Email

## Overview

This workflow automates customer service email responses using AI. Incoming emails are automatically categorized, matched to sample responses from a Google Sheet, and replied to with a contextually appropriate, AI-generated answer. 

## How It Works

1. **Trigger on Incoming Email**  
   The workflow monitors a specified email inbox for new, unread emails.

2. **Email Categorization via AI**  
   The content of each new email is sent to OpenAI's GPT-3.5 (via the "chat" completion method), asking the AI to classify the email as one of:  
   - Sales Enquiry  
   - Customer Feedback  
   - Complaint

3. **Match Response Template**  
   The detected category (e.g., "Customer Feedback") is used to look up a matching response prompt in a Google Sheet (column "A" = category, column "B" = prompt template).

4. **Generate Final Response**  
   The retrieved response template is combined with the original email, and sent again to OpenAI GPT-3.5 to generate the final, personalized reply.

5. **Send AI-Powered Reply**  
   The workflow waits 30 seconds (optional step, possibly for rate limiting) and sends the generated reply back to the original sender, using the original email's subject prefixed with "RE:".

## Inputs

- **Incoming Email (IMAP/Google/Microsoft account):**  
  - Must provide connection information for the monitored email inbox.
  - Key parameters include folder to monitor (usually `INBOX`), criteria (e.g., unread emails), and maximum results per run.

- **OpenAI API Connection:**  
  - Required for email classification and reply generation.

- **Google Sheets Connection:**  
  - Access to a specific Google Spreadsheet containing response templates.
  - The sheet must have at least a column for the category and corresponding prompt templates.

## Outputs

- **Automated Reply Email:**  
  - Sent to the original email sender.
  - Subject is set as "RE: [original subject]".
  - Body is an AI-generated response tailored to both the category and the original email content.

- **Error Logs and Statuses:**  
  - (Implicitly) errors from any module are handled according to the platform's standard error management.

## Customization

- **Categories:**  
  You can add or modify categories in the GPT system prompt and ensure corresponding templates exist in your Google Sheet.

- **Response Templates:**  
  Maintain the templates in the Google Sheet for each category; these can be personalized to suit your organization's tone and style.

- **AI Model Settings:**  
  GPT-3.5-turbo parameters such as temperature, max_tokens, etc., are currently set for balanced creativity and length but can be adjusted.

## Requirements

- Active connections to:
  - An email service provider (Gmail, Outlook, etc.)
  - OpenAI account for GPT-3.5-Turbo API
  - Google Sheets with proper permissions

## Example Sheet Structure

| A (Category)      | B (Prompt Sample)              |
|-------------------|-------------------------------|
| Sales Enquiry     | Please provide product info... |
| Customer Feedback | Thank you for your feedback... |
| Complaint         | We're sorry to hear about...   |

Each row links a category to a response template.

## Flow Diagram

```
[Email Inbox] 
     ↓
[AI Categorization (GPT-3.5)]
     ↓
[Response Template Lookup (Google Sheets)]
     ↓
[AI Reply Generation (GPT-3.5)]
     ↓
[Send Response Email]
```

## Notes

- The workflow works best if emails are in plain text and structured.
- For extension (e.g., more categories or more personalized actions), modify the prompts, categories, or add additional logic/steps as needed.
```
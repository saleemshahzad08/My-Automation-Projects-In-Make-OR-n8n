# Automated Workflow Archiver

## Overview

The **Automated Workflow Archiver** is an automation scenario (designed for platforms like Make / Integromat) that processes Google Forms submissions containing workflow JSON files. It analyzes and archives both the original workflow blueprint and an AI-generated README in a GitHub repository. The automation is ideal for maintaining an organized, easily navigable archive of submitted workflows along with human-friendly summaries.

---

## How It Works

1. **Trigger:** Watches a specified Google Form for new responses (limit: 5 at a time).
2. **File Retrieval:** For each response, retrieves the uploaded workflow JSON file(s) from Google Drive.
3. **README Generation:** Sends the workflow JSON to OpenAI's GPT-4 for analysis and generates a human-readable README summarizing the workflow's purpose, inputs, and outputs.
4. **Archiving:** 
    - Uploads the generated README as `README.md` in a dedicated folder in a GitHub repository.
    - Uploads the original workflow JSON as `blueprint.json` in the same folder.

---

## Inputs

### 1. Google Form Response
- **Form ID:** The Google Form being monitored for submissions.
- **Respondent Data:** Email, response timestamps.
- **Workflow JSON File Upload:** The key attachment provided by the user.

### 2. Google Drive
- **File ID:** Extracted from the form response to locate the uploaded workflow file.

### 3. AI ReadMe Generation
- **Workflow JSON Content:** The raw content of the uploaded file is sent to OpenAI GPT-4 for summarization.

### 4. GitHub Repository
- **Target Repository/Folder:** Workflow results are pushed to a specific repo/folder structure, with authentication via a personal access token.

---

## Outputs

1. **Generated README.md**  
   - Added to a new folder (named per workflow) in the specified GitHub repository.
   - Contains a concise summary of the workflow: its purpose, inputs, outputs.

2. **Original Workflow Blueprint (`blueprint.json`)**  
   - The original uploaded JSON file, written to the same folder in the GitHub repository for future reference or reuse.

---

## Technologies & Services Used

- **Google Forms:** For workflow submissions.
- **Google Drive:** For managing uploaded files.
- **OpenAI GPT-4:** For automatic, natural language README generation.
- **GitHub:** For permanent archival of both original workflows and human-readable documentation.

---

## Prerequisites

- **Google account** with access to the target Form and Drive.
- **OpenAI API key** (integrated via Make / n8n connection).
- **GitHub repository** (with a personal access token granting write access).

---

## Typical Use Cases

- Crowdsourcing workflow ideas with clear documentation.
- Automating workflow analysis and archive creation.
- Building a searchable, documented library of automation blueprints.
- Reducing manual work in maintaining workflow repositories.

---

## Data Flow Summary

```
Google Form (user submits workflow JSON file)
        ↓
Google Drive (retrieve uploaded file)
        ↓
OpenAI GPT-4 (generate README from file content)
        ↓
GitHub (archive original file and generated README)
```

---

## Configuration Notes

- Update connection credentials for Google, OpenAI, and GitHub as needed.
- The GitHub token in the HTTP modules must have `repo` permissions.
- Adjust the form ID, file limits, and GitHub repo/branch names as appropriate.

---

## License

MIT (or as appropriate for your organization)
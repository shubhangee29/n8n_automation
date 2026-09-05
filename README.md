<img width="916" height="396" alt="image" src="https://github.com/user-attachments/assets/0e8bde5b-0f7d-49fc-bbbc-7d07199cda76" /># AI-Personalized Lead Outreach Automation (n8n)

An n8n workflow that turns a spreadsheet of leads into personalized, ready-to-send Gmail drafts — complete with automatic follow-up tracking and optional PDF attachments.

Built with n8n, Google Gemini, Google Sheets/Drive, and Gmail.

![Workflow overview](workflow-screenshot.png)

## Overview

Given a lead list and a tracking sheet, this workflow:

1. Pulls the lead list from a file stored in Google Drive.
2. Checks the tracking sheet to see which leads have already been contacted.
3. Processes leads one at a time, generating a short, personalized outreach message for each based on their name, role, and organization.
4. Attaches a PDF (a brochure, resume, or info sheet) to every draft.
5. Creates a Gmail draft per lead — nothing sends automatically, so every message is reviewed before going out.
6. Updates the tracking sheet to mark each lead as contacted, so future runs never repeat a message.
7. Runs on a schedule, so the pipeline refills and works through new leads on its own.

## Workflow structure

```
Schedule Trigger
      |
      +--> Download Lead File (Google Drive) --> Extract from File (CSV) --+
      |                                                                    |
      +--> Download PDF (Google Drive) -----------------+                 |
                                                          v                 v
                                                 Merge (PDF x Leads, all combinations)
                                                          |
                                                 Get Row(s) in Sheet (status check)
                                                          |
                                                 Loop Over Items (one lead at a time)
                                                    |              |
                                           Message a Model      (original item,
                                           (Gemini)               PDF intact)
                                                    |              |
                                               Edit Fields          |
                                          (to / subject / body)     |
                                                    +--------+------+
                                                             v
                                                 Merge (by position)
                                                             |
                                                 Create a Draft (Gmail)
                                                    |            |
                                                Success        Error
                                                    |
                                           Update Row in Sheet
                                           (mark as contacted)
                                                    |
                                           back to Loop Over Items
```

The second Merge node exists because AI nodes in n8n strip binary data as they process text. It reattaches the original PDF after the AI step, so the attachment survives the rest of the chain.

## Requirements

- An n8n instance — Cloud or self-hosted (Docker, a cloud VM, or local)
- A Google Cloud project with the following APIs enabled:
  - Google Drive API
  - Google Sheets API
  - Gmail API
- A Google Service Account (for Drive and Sheets access)
- A Gmail OAuth2 credential (for creating drafts)
- A Google Gemini API key (a free tier is available)
- A Google Sheet with the columns: `name`, `title`, `institute`, `mail`, `status`

## Setup

1. Import `workflow.json` into your n8n instance (`... menu -> Import from File`).
2. Create the lead list as a Google Sheet, or a CSV uploaded to Drive, using the columns above.
3. Share the sheet and lead file with your Google service account's email address.
4. Configure credentials in each relevant node:
   - Google Drive and Sheets nodes: your Service Account
   - Gmail node: OAuth2, signed in as the sending account
   - Message a Model node: your Gemini API key
5. Point the two Download File nodes at your lead file and your PDF attachment.
6. Adjust the prompt in Message a Model for tone or length, if needed.
7. Test with a small list before scheduling it to run live.
8. Publish the workflow and set a schedule.

Self-hosted n8n instances require a manually created OAuth2 Client ID and Secret for Gmail, via Google Cloud Console, since the Managed OAuth2 option is Cloud-only. Add your Gmail address as a test user on the OAuth consent screen to avoid Google's verification process.

## Customization ideas

- Replace the static lead file with a live CRM export.
- Add a Wait node and enable Retry on Fail for better resilience against rate limits.
- Route the Gmail error output to log failed sends separately in the sheet.
- Swap Gemini for a different provider by replacing the Message a Model node.
- Pair this with a lead-sourcing workflow so the sheet refills automatically.
<img width="463" height="205" alt="workflow-screenshot 2" src="https://github.com/user-attachments/assets/6c9b964b-92b8-49f9-bb6f-b2a54052100a" />



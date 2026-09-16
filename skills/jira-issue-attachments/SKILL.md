---
name: jira-issue-attachments
description: Downloads and reads attachments from Jira Cloud user stories and issues using the official Atlassian MCP V2. Use when reading a Jira user story, issue, or ticket (getJiraIssue), or when the description has attachments, screenshots, or embedded media.
---

# Jira issue attachments

Use the official Atlassian MCP (`https://mcp.atlassian.com/v2/mcp`) to list issues and obtain a short-lived download URL. This skill does not use a custom MCP, `TOKEN_JIRA`, or `JIRA_CLOUD_ID`. The official tool does not write files; run curl locally, then Read the saved file when it can be inspected (images, text, PDF, and similar).

Requires the Atlassian MCP authenticated.

## Instructions

1. Load this workflow whenever the user asks to read a Jira user story, issue, or ticket, or when using `getJiraIssue`.
2. Call `getAccessibleAtlassianResources` once per session and reuse the returned `cloudId`.
3. Read the issue with official Atlassian tools (`getJiraIssue`). Prefer ADF if you need to detect `media` nodes. Include attachments (`fields` with `attachment`).
4. If the issue has attachments, or the description has embedded media, markdown images, or screenshots, download **every** attachment in `fields.attachment` **before** summarizing requirements or implementing from the ticket.
   - Use only numeric ids from `fields.attachment[].id` (for example `"133694"`). Do **not** pass ADF `media` UUIDs.
   - Do not filter by MIME type. Download images, documents, spreadsheets, archives, and any other attachment.
   - For each attachment:
     1. Call **`downloadJiraIssueAttachment`**. If it is a primary tool, call it directly. Otherwise `discover` then `executeRead` with `name: "downloadJiraIssueAttachment"`.
     2. Pass `cloudId`, `attachmentId`, and `outputPath`: `.attachments/{ISSUE-KEY}/{id}-{filename}` (relative to the workspace).
     3. Create the parent directory **before** curl. Linux/macOS: `mkdir -p`. Windows PowerShell: `New-Item -ItemType Directory -Force`.
     4. Run the download **immediately** (the URL is short-lived). Keep `--location` and `--output` from `downloadCommand`.
        - **Windows:** first token must be `curl.exe`. Never run `curl` in PowerShell (it is an alias for `Invoke-WebRequest`). If schannel fails with `CRYPT_E_NO_REVOCATION_CHECK`, add `--ssl-no-revoke`.
        - **Linux/macOS:** run `curl` as returned in `downloadCommand`.
     5. **Read** the saved file when the type is inspectable (images, text, PDF, and similar). For opaque binaries, note the path and filename instead of inventing contents.
5. If the official tool errors, curl fails, or no files were saved while the story clearly refers to attachments, say so. Do not invent attachment contents.
6. Do not commit files under `.attachments/`.

## Examples

User: "Read user story PROJ-123"

1. `getAccessibleAtlassianResources` then `getJiraIssue` for PROJ-123
2. For each item in `fields.attachment` → `downloadJiraIssueAttachment` with numeric `attachmentId` and `outputPath` under `.attachments/PROJ-123/`
3. `mkdir` the folder, then `curl.exe` (Windows) or `curl` (Linux/macOS)
4. Read inspectable files (screenshots, PDFs, text)
5. Answer using the issue text plus what the attachments show

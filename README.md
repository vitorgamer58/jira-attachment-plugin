# jira-attachment-plugin

Agent **skill** that tells Cursor or Claude how to download **attachments** from Jira Cloud issues (images, documents, and any other file) so the agent can inspect them.

There is **no local MCP server**. Listing the issue and getting a download URL uses the official **Atlassian MCP V2** (`https://mcp.atlassian.com/v2/mcp`). Saving bytes is a local `curl` / `curl.exe` command. Then the agent reads inspectable files.

## Requirements

- [Cursor](https://cursor.com) or [Claude](https://claude.ai/code) (or another client that loads [Agent Skills](https://agentskills.io) and MCP)
- Official **Atlassian MCP** installed and authenticated (OAuth). No `TOKEN_JIRA`, no `JIRA_CLOUD_ID`, no Node.js.

## Install

The package is an Agent Skill under `skills/jira-issue-attachments/`. How you load it depends on the client.

### Cursor (Team Marketplace)

The marketplace clones this git repository into the plugin cache. After install or refresh, reload if needed so `jira-issue-attachments` is available (`/jira-issue-attachments`).

1. Import this repository in Dashboard → Plugins.
2. Install **jira-attachment-plugin**.
3. Confirm the official Atlassian MCP is connected.

### Claude Code (plugin marketplace)

This repo includes `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`. In Claude Code:

```text
/plugin marketplace add vitorgamer58/jira-attachment-plugin
/plugin install jira-attachment-plugin@jira-attachment-plugin
```

Confirm the official Atlassian MCP is connected. You can also copy `skills/jira-issue-attachments/` into a local skills directory.

## How it works

1. `getAccessibleAtlassianResources` (once) then `getJiraIssue` with attachments.
2. For **each** attachment (any MIME type), call official **`downloadJiraIssueAttachment`** (`attachmentId` = numeric `fields.attachment[].id`, not ADF media UUIDs). If the client does not expose it as a primary tool, discover it and run it through the MCP read/execute helper (`executeRead` with `name: "downloadJiraIssueAttachment"`).
3. The tool returns a short-lived `downloadUrl` and a `downloadCommand`. Create `.attachments/{ISSUE-KEY}/` first, then run curl immediately.
4. **Windows:** use `curl.exe` (PowerShell `curl` is `Invoke-WebRequest`). If schannel fails with a certificate revocation check (`CRYPT_E_NO_REVOCATION_CHECK`), add `--ssl-no-revoke`. **Linux/macOS:** use `curl` as returned. Keep `--location` and `--output`.
5. Read inspectable files (images, text, PDF, and similar). Do not invent attachment contents if the download fails.

Do not commit downloaded files.

## Skill

`skills/jira-issue-attachments` is the whole plugin. When reading a Jira story or issue with attachments, follow that workflow.

## Security

- Attachments may contain sensitive data. Keep `.attachments/` gitignored.
- Download URLs expire; do not log or reuse them later.

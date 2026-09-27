# n8n AI Agent Workflows

Fifteen n8n workflows that automate QA work with AI agents — Jira triage, root-cause
analysis, test generation from Swagger, performance and visual-diff reporting, and a
daily LinkedIn post agent. Each workflow lives in its own folder and is a complete,
importable `.json` export taken from a live n8n instance.

Nothing here runs on its own. A workflow is a graph definition: you import it into your
own n8n, reconnect the credentials, and run it.

## Contents

One folder per workflow. The folder name matches the workflow; the `.json` inside keeps
the original n8n export name (the 16-character prefix is the workflow ID from the
instance it was exported from).

### Chat-triggered agents

Open the workflow in n8n and use the built-in **Chat** panel to talk to it.

| Folder | What it does | Needs |
|---|---|---|
| [Bug-Triage-AI-Agent](Bug-Triage-AI-Agent/) | Agent with Jira and Google Sheets as tools: reads a bug, decides severity/priority/owner, writes the triage row back | OpenAI, Jira, Google Sheets |
| [Fetch-JIRA-Ticket-using-n8n-and-append-output-to-Google-Sheet](Fetch-JIRA-Ticket-using-n8n-and-append-output-to-Google-Sheet/) | Pulls a Jira ticket by key and appends the summary to a Google Doc | OpenAI, Jira, Google Docs |
| [Create-Test-Plan-From-Jira-Ticket-and-append-output-to-Google-Sheet](Create-Test-Plan-From-Jira-Ticket-and-append-output-to-Google-Sheet/) | Turns a Jira story into a test plan and appends it to a Google Doc | OpenAI, Jira, Google Docs |
| [Create-Test-Strategy-from-Jira-ticket-and-append-output-to-Google-Sheet](Create-Test-Strategy-from-Jira-ticket-and-append-output-to-Google-Sheet/) | Same shape, but produces a test *strategy* rather than a plan | OpenAI, Jira, Google Docs |
| [Daily-QA-LinkedIn-Post-Agent](Daily-QA-LinkedIn-Post-Agent/) | Drafts a QA-themed LinkedIn post on demand and publishes it | OpenAI, Gemini, LinkedIn |

### Webhook-triggered agents

Each exposes a `POST` webhook and replies with the agent's structured JSON, so you can
call them from CI.

| Folder | Path | What it does | Needs |
|---|---|---|---|
| [Swagger-to-API-Tests](Swagger-to-API-Tests/) | `/swagger-tests` | Fetches an OpenAPI/Swagger spec and generates API test cases per endpoint, then posts a summary to Telegram | Groq, Telegram |
| [Flaky-Test-RCA-Analyzer-Working-AI-Agent-TTA](Flaky-Test-RCA-Analyzer-Working-AI-Agent-TTA/) | `/flaky-rca` | Takes a failing test's history and returns a root-cause verdict, notifying Slack and Telegram | Groq, Telegram, Slack |
| [Performance-Test-Analyzer](Performance-Test-Analyzer/) | `/perf-report` | Reads a load-test result, flags regressions, and emails an HTML report | Groq, Telegram, SMTP |
| [Visual-Diff-Explainer](Visual-Diff-Explainer/) | `/visual-diff` | Explains a screenshot diff in words, decides whether it is a real regression, and files a Jira bug if it is | Groq, Jira, Telegram |
| [Persona-Based-Testing-Engine](Persona-Based-Testing-Engine/) | `/persona-flows` | Generates per-persona user flows, loops over them in batches, and logs results to Google Sheets | Groq, Google Sheets, Telegram |

### Form-triggered

| Folder | What it does | Needs |
|---|---|---|
| [Screenshot-to-Bug-Reporter-Working-AI-Agent-TTA](Screenshot-to-Bug-Reporter-Working-AI-Agent-TTA/) | An n8n Form takes a screenshot upload, a vision model reads it, and a formatted Jira bug is created | Groq, Jira |

### Scheduled

| Folder | Schedule | What it does | Needs |
|---|---|---|---|
| [Production-Bug-RCA-Automation-created-by-Claude](Production-Bug-RCA-Automation-created-by-Claude/) | every 15 min | Polls Jira for new production bugs, runs RCA on each, writes findings to Google Sheets | OpenAI, Jira, Google Sheets |
| [Root-Cause-Analysis-RCA-AI-Agent-created-by-n8n-AI-Assistant](Root-Cause-Analysis-RCA-AI-Agent-created-by-n8n-AI-Assistant/) | Jira event + weekly Mon 06:00 | Reacts to a Jira production-bug event with an RCA, and exports a weekly roll-up to Sheets/Drive | OpenAI, Jira, Google Sheets, Google Drive |
| [Daily-QA-LinkedIn-Post-Agent-created-by-Claude](Daily-QA-LinkedIn-Post-Agent-created-by-Claude/) | daily | Picks a topic from a Sheet, generates post + image, publishes to LinkedIn, pings Telegram | OpenAI, Gemini, LinkedIn, Google Sheets, Telegram |
| [Daily-QA-LinkedIn-Post-Agent-Hugging-face-free-image-generator-Telegram-Working](Daily-QA-LinkedIn-Post-Agent-Hugging-face-free-image-generator-Telegram-Working/) | daily 06:30 | Same, but uses a free Hugging Face image model via HTTP header auth instead of a paid image API | OpenAI, Gemini, LinkedIn, Hugging Face, Telegram |

## Requirements

- An n8n instance — [n8n Cloud](https://n8n.io/) or self-hosted (`npx n8n`, or Docker).
  Version 1.x with the LangChain nodes (`@n8n/n8n-nodes-langchain`) available; the agent
  workflows will not import without them.
- Accounts for whichever integrations the workflow you want uses (see the tables above).

### Credentials used across the set

| n8n credential type | Used by | Where to get it |
|---|---|---|
| `openAiApi` | 8 workflows | platform.openai.com → API keys |
| `groqApi` | 6 workflows | console.groq.com → API keys (free tier) |
| `googlePalmApi` (Gemini) | 3 workflows | aistudio.google.com → Get API key |
| `jiraSoftwareCloudApi` | 8 workflows | Atlassian account settings → API tokens |
| `googleSheetsOAuth2Api` | 5 workflows | Google Cloud OAuth client |
| `googleDocsOAuth2Api` | 3 workflows | Google Cloud OAuth client |
| `googleDriveOAuth2Api` | 1 workflow | Google Cloud OAuth client |
| `telegramApi` | 7 workflows | Telegram BotFather → bot token |
| `linkedInOAuth2Api` | 3 workflows | LinkedIn developer app |
| `httpHeaderAuth` | 1 workflow | Hugging Face token, sent as an `Authorization: Bearer` header |
| `smtp` | 1 workflow | any SMTP host, for the performance report email |

No secrets are stored in these files. n8n exports keep only the credential's *name* and
internal ID, so every imported workflow shows its credential fields as unset — that is
expected, and you reconnect them on import.

## Setup

1. Clone this repo:
   ```bash
   git clone https://github.com/jayamvishnudeep/n8n-AI-Agent-Workflows.git
   ```
2. In n8n, go to **Workflows → Import from File** and pick the `.json` from the folder
   you want. (Or paste the file contents onto an empty canvas with Ctrl+V.)
3. Create the credentials the tables above list for that workflow, under
   **Credentials → New**.
4. Open each red-flagged node and pick the credential you just created. Nodes import
   with their credential reference dangling, so this step is required, not optional.
5. Replace the placeholders that point at the original accounts:
   - **Jira** — base URL and project key (`https://example.atlassian.net` in the RCA
     workflow) and the issue type used when creating bugs.
   - **Google Sheets / Docs / Drive** — the document and sheet IDs in each node.
   - **Telegram** — the `chatId`, which is still the original author's chat.
   - **Slack / email** — channel and recipient in the flaky-test and performance flows.
6. Run once with **Execute Workflow** before you switch the toggle to Active.

## Usage

**Chat agents** — open the workflow and click **Chat** at the bottom of the canvas. Give
it the Jira key or the request in plain English, e.g. `Triage PROJ-482`.

**Webhook agents** — activate the workflow, then `POST` to its production URL:

```bash
curl -X POST https://<your-n8n-host>/webhook/swagger-tests \
  -H "Content-Type: application/json" \
  -d '{"specUrl":"https://petstore3.swagger.io/api/v3/openapi.json"}'
```

The response is the agent's structured output, so these slot into a CI step directly.
Check the **Respond to Caller** node in each workflow for the exact response shape, and
the webhook node's own settings for the fields it expects on input.

**Form agent** — activate it and open the form URL that the Form Trigger node shows.

**Scheduled agents** — set the schedule you want in the trigger node, then activate. The
intervals shipped here are the original ones; the 15-minute Jira poll in particular will
burn API quota if you leave it as-is.

## Troubleshooting

**Import fails, or nodes show as "?"** — the instance is missing the LangChain node
package, or is on an n8n version older than the export. Update n8n.

**Node shows a red triangle after import** — its credential is not selected. Expected on
every import; see step 4.

**`401` from the model node** — the API key is wrong, or the Groq/OpenAI model named in
the node has been retired. Open the node and re-pick a current model from the dropdown.

**Webhook returns 404** — the workflow is not Active, or you are calling the production
path while testing. Active workflows serve `/webhook/<path>`; unsaved testing serves
`/webhook-test/<path>`, and only for a single call per **Execute Workflow** click.

**Agent returns text where JSON was expected** — the Structured Output Parser node's
schema and the prompt have drifted apart. Fix the schema first; these flows depend on
the parsed shape downstream.

**Jira node fails with 404 on a valid key** — the base URL still points at
`example.atlassian.net`, or the project key does not exist in your Jira.

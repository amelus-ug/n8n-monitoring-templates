# Global Error Handler with Severity-Based Escalation

One central n8n workflow that catches failures from **all** your other workflows, logs every error to a spreadsheet, and escalates critical failures to the alert channel of your choice — Slack, Telegram, Email, or native iOS push.

![Workflow overview](#) <!-- TODO: add screenshot before publishing -->

## Why you need this

By default, a failing n8n workflow fails silently — you find out hours later when the data is missing. n8n's *Error Workflow* setting lets you route every failure to one central handler. This template is that handler, ready to go:

- **Every error is logged** to a Google Sheet (audit trail: what failed, where, when, why).
- **Critical workflows page you immediately** — you define what "critical" means.
- **Non-critical errors stay quiet** — logged, but no notification noise.

## How it works

```
Error Trigger → Classify Severity → Log to Sheet → Critical? ──▶ Alert (Slack / Telegram / Email / iOS push)
                                                       └──▶ Logged only
```

1. **Catch Workflow Errors** — n8n's Error Trigger fires whenever a monitored workflow fails.
2. **Classify Severity** — a small Code node marks the error `critical` or `warning` based on the workflow name (configurable tag `[critical]` and keyword list).
3. **Log to Error Sheet** — appends a row to Google Sheets (swap for Postgres/Airtable/Notion if you prefer).
4. **Critical?** — critical errors fan out to your alert channel(s); everything else ends quietly.

## Setup

1. **Import** `workflow.json` into your n8n instance (v1.x).
2. **Create the error log sheet:** a Google Sheet with the header row
   `timestamp | severity | workflowName | workflowId | executionId | executionUrl | nodeName | errorMessage`,
   then select credentials + document + sheet in the **Log to Error Sheet** node.
3. **Pick your alert channel:** Slack is enabled by default (set credentials + channel). To use another channel, enable one of the alternative nodes and disable Slack:
   - **Telegram** — set your bot credentials and chat ID.
   - **Email** — set SMTP credentials and from/to addresses.
   - **WorkflowBuddy (iOS push)** — see below.
4. **Activate this workflow.**
5. **Wire it up:** in every workflow you want to monitor, open **Settings → Error Workflow** and select *Global Error Handler*.
6. **Test it:** create a throwaway workflow with a failing node (e.g. an HTTP request to an invalid URL), run it once, and watch the error arrive.

## Defining what's critical

Open **Classify Severity** and edit the two constants at the top:

```js
const CRITICAL_TAG = '[critical]';          // any workflow with this in its name is critical
const CRITICAL_KEYWORDS = ['payment', 'billing', 'invoice'];
```

Rename your most important workflows to include the tag (e.g. `Invoice Sync [critical]`), or add keywords that match their names.

## Getting alerts on your iPhone (WorkflowBuddy)

If you want native push notifications without running a Slack workspace or Telegram bot:

1. Install **Workflow Buddy** from the App Store ([amelus.de](https://www.amelus.de)).
2. In the app, open **Settings → Push API** and copy your key.
3. In n8n, create a **Header Auth** credential — Name: `Authorization`, Value: `Bearer wb_…` (your key).
4. Select the credential in the **Alert: WorkflowBuddy (iOS Push)** node and enable the node.

The free tier currently includes 50 pushes per day — plenty for error alerts.

## Customization ideas

- Log to Postgres instead of Sheets for queryable error history.
- Add a second severity level (e.g. `page` vs `notify`) with a Switch node.
- Combine with a daily AI error digest (template coming soon in this repo).

## FAQ

**Does the error workflow catch its own failures?** No — n8n never triggers an error workflow for itself. Keep this workflow simple and test it once after setup.

**Can I set the error workflow globally?** Yes, on self-hosted instances you can set a default error workflow via instance settings; otherwise set it per workflow under **Settings → Error Workflow**.

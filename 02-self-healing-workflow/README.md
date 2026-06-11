# Self-Healing Workflow – Auto-Retry Failed Executions

Automatically retries failed n8n executions across your whole instance and only alerts you when retries are exhausted. Transient glitches — rate limits, brief API outages, network blips — heal themselves silently; you only hear about the failures that actually need you.

## Why you need this

Most workflow failures are temporary. A third-party API returns a 503 for thirty seconds, your rate limit resets, a DNS hiccup clears. Without retries, every one of these wakes you up. With naive per-node retries, you can't bound the total attempts or get told when something is *really* broken. This template does both.

## How it works

```
Schedule → List error executions (n8n API) → Decide per execution ──▶ retry (POST /executions/{id}/retry)
                                                                  └──▶ give up → alert
```

1. **Every 15 minutes** (configurable) it lists recent `error` executions via the n8n public API.
2. **Decide Actions** looks up how many times each execution has already been retried (tracked in the workflow's static data) and chooses *retry* or *give up*.
3. Retries call `POST /api/v1/executions/{id}/retry`. Each retry spawns a new execution; its attempt count is carried forward so the chain is bounded by `MAX_RETRIES`.
4. When a job is still failing after the last attempt, you get **one** alert on your channel of choice.

## Setup

1. **Import** `workflow.json`.
2. **Create an n8n API key:** Settings → n8n API → Create. Add it as an **HTTP Header Auth** credential — Name `X-N8N-API-KEY`, Value = your key — and select it on the three HTTP Request nodes (`List Failed Executions`, `Retry Execution`, and the WorkflowBuddy node uses a *different* credential, see below).
3. **Set `N8N_BASE_URL`** in the **Config** node (e.g. `https://n8n.example.com`, no trailing slash). Optionally adjust `MAX_RETRIES` (default 3) and `LOOKBACK_LIMIT` (default 100).
4. **Pick an alert channel:** Slack (default), Telegram, Email, or WorkflowBuddy (iOS push — see below).
5. **Activate the workflow.** ⚠️ Retry bookkeeping lives in static data, which only persists for active/production executions — not manual test runs.

## ⚠️ Make your workflows safe to re-run

Retrying re-runs a workflow. If a workflow sends an email, charges a card, or posts a message, retrying it could do that twice. Make critical side effects **idempotent** (e.g. guard with an "already processed?" check) before enabling auto-retry for them.

## Getting "gave up" alerts on your iPhone (WorkflowBuddy)

1. Install [**Workflow Buddy**](https://apps.apple.com/app/id6760253861) from the App Store.
2. In the app, open **Settings → Push API** and copy your key.
3. In n8n, create a **Header Auth** credential — Name: `Authorization`, Value: `Bearer wb_…`.
4. Select it on the **Alert: WorkflowBuddy (iOS Push)** node and enable the node.

## Tuning

- **`MAX_RETRIES`** — total attempts before giving up. 3 is a sane default for transient errors.
- **Schedule interval** — shorter = faster recovery, more API calls. 15 min suits most cases.
- **Scope** — to retry only specific workflows, add a filter in **Decide Actions** on `workflowId`.

## Limitations

- Operates on executions visible to the n8n public API (it does not catch failures of disabled or never-triggered workflows — use the [Health Check Monitor](../03-health-check-monitor/) for that).
- Scans the most recent `LOOKBACK_LIMIT` error executions per run; very high failure volumes may need a shorter interval or higher limit.

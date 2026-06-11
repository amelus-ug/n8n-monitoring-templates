# n8n Health Check – Heartbeat Monitor for Critical Workflows

Detects the failure mode that error alerts miss: a workflow that **silently stops running**. No error is thrown — a trigger breaks, a cron dies, someone deactivates it by accident — and it just goes quiet until you notice the missing data days later. This monitor watches *when each workflow last succeeded* and alerts you when the gap gets too big.

## Why you need this

Error handlers only fire when something *runs and fails*. They can't tell you about a workflow that never ran at all. The classic horror story: a nightly backup gets disabled during maintenance and nobody notices for three weeks. A heartbeat monitor is the safety net underneath your error handling.

## How it works

```
Schedule → for each watched workflow → ask n8n API for last successful run → too old? → alert
```

1. **On a schedule** (default hourly) it iterates your list of critical workflows.
2. For each, it asks the n8n API for the most recent **successful** execution (`GET /api/v1/executions?workflowId=…&status=success&limit=1`).
3. **Evaluate Freshness** compares the last success to a per-workflow threshold. If it's too old — or there's no successful run on record — the workflow is flagged.
4. You get one alert per stale workflow.

## Setup

1. **Import** `workflow.json`.
2. **Edit the Monitored Workflows node:** set `N8N_BASE_URL` and list the workflows to watch:
   ```js
   const WORKFLOWS = [
     { workflowId: '123', workflowName: 'Hourly Data Sync', maxAgeHours: 2 },
     { workflowId: '456', workflowName: 'Nightly Backup',  maxAgeHours: 26 },
   ];
   ```
   Find a workflow's id in its URL (`/workflow/<id>`). Set `maxAgeHours` a bit above its normal run interval.
3. **Add an n8n API key** as an **HTTP Header Auth** credential (Name `X-N8N-API-KEY`) on the `Get Last Success` node.
4. **Pick an alert channel:** Slack (default), Telegram, Email, or WorkflowBuddy.
5. **Activate the workflow.**

## 💡 Where to run it

Run this monitor on a **second n8n instance** (or pair it with an external uptime check like UptimeRobot / Better Stack hitting your instance's `/healthz`). If you run it on the same instance it watches, it can't alert you when that instance is fully down — only when individual workflows go stale.

## Getting "went silent" alerts on your iPhone (WorkflowBuddy)

1. Install [**Workflow Buddy**](https://apps.apple.com/app/id6760253861) from the App Store.
2. In the app, open **Settings → Push API** and copy your key.
3. In n8n, create a **Header Auth** credential — Name: `Authorization`, Value: `Bearer wb_…`.
4. Select it on the **Alert: WorkflowBuddy (iOS Push)** node and enable the node.

## Tuning

- **`maxAgeHours`** per workflow — the heart of the monitor. Too tight = false alarms; too loose = slow detection. Rule of thumb: ~2× the normal interval.
- **Schedule interval** — how often to check. Hourly is fine for most; tighten for time-critical jobs.

## Pairs well with

- [Global Error Handler](../01-global-error-handler/) — catches workflows that run *and* fail.
- [Self-Healing Workflow](../02-self-healing-workflow/) — auto-retries the failures before they reach you.

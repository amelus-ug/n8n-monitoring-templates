# n8n Monitoring Templates

Production-ready n8n workflow templates for **error handling, monitoring, and alerting**. Import, configure your channels, done.

Every template works standalone with the notification channel of your choice — Slack, Telegram, Email, or native iOS push via [WorkflowBuddy](https://www.amelus.de).

## Templates

| # | Template | What it does | Status |
|---|----------|--------------|--------|
| 1 | [Global Error Handler](01-global-error-handler/) | Catch failures from all workflows, log them, escalate critical ones instantly | ✅ Ready |
| 2 | Self-Healing Workflow | Auto-retry failed executions via the n8n API, alert when retries are exhausted | 🔜 Planned |
| 3 | Health Check / Heartbeat Monitor | Detect silently dead workflows and unreachable instances on a schedule | 🔜 Planned |
| 4 | AI Error Digest | Summarize the last 24h of errors with an LLM into one daily message | 🔜 Planned |
| 5 | Long-Running Job Notifier | Push notifications on start/progress/completion of long jobs | 🔜 Planned |

## Quick start

1. Open a template folder and download its `workflow.json`.
2. In n8n: **Workflows → Import from File**.
3. Follow the template's `README.md` and the sticky notes inside the workflow.

## Notification channels

Each template ships with multiple alert options — enable the one you use:

- **Slack** (enabled by default)
- **Telegram**
- **Email (SMTP)**
- **WorkflowBuddy** — native iOS push notifications for n8n, no extra infrastructure ([App Store](https://www.amelus.de))

## About

Maintained by [Amelus UG](https://www.amelus.de), the team behind **Workflow Buddy**, the iOS companion app for n8n. Issues and PRs welcome.

## License

[MIT](LICENSE)

# Long-Running Job Notifier – Start & Completion Push

Wrap any slow workflow so you get a push when it **starts** and another when it **finishes**, including the total duration. Stop babysitting a browser tab waiting for a big export, batch import, render, or nightly ETL to finish.

## Why you need this

Long jobs put you in an awkward spot: you can't tell whether they're still running or silently died, so you keep checking. A start/finish notification with a duration tells you exactly when to walk away and exactly when to come back — and gives you a running record of how long jobs actually take.

## How it works

```
Trigger → Mark Start ──▶ notify "started"
                     └──▶ ▶ your long job → Mark Done ──▶ notify "finished in 12m 30s"
```

Notifications are **side branches**, so your job runs whether or not a channel is enabled — a misconfigured notifier can never block the actual work. Duration is measured between `Mark Start` and `Mark Done`.

## Setup

1. **Import** `workflow.json`.
2. **Replace `▶ Your Long Job Here`** with your real workflow steps. (It's a 5-second Wait node so the first test run shows a real duration — delete it and wire your work in its place.)
3. **Set the job name** in the **Mark Start** node (`JOB_NAME`).
4. **Enable + configure your channel(s):**
   - **Start** notification: Slack (default) and a ready-to-enable WorkflowBuddy node.
   - **Finished** notification: Slack (default), Telegram, Email, WorkflowBuddy.
5. **Run it.** A Manual Trigger is included for testing — swap it for your real trigger (Webhook, Schedule, etc.).

## Two ways to use it

- **Inline:** build your long job directly into this workflow, between `Mark Start` and `Mark Done`.
- **Wrapper / sub-workflow:** keep this as a thin shell and put your real workflow in the middle via an **Execute Workflow** node — handy for reusing the same notifier around several jobs.

## Adding progress notifications

For very long jobs, drop an extra notification node inside a loop (e.g. after each batch in a *Loop Over Items* node) referencing the same channel, with a message like `Processed {{ $runIndex + 1 }} batches…`. The start/finish pattern stays as-is.

## Getting start/finish pushes on your iPhone (WorkflowBuddy)

1. Install [**Workflow Buddy**](https://apps.apple.com/app/id6760253861) from the App Store.
2. In the app, open **Settings → Push API** and copy your key.
3. In n8n, create a **Header Auth** credential — Name: `Authorization`, Value: `Bearer wb_…`.
4. Select it on the WorkflowBuddy node(s) and enable them.

## Notes

- Duration is wall-clock time between the two markers. If your job uses a **Wait** node that pauses for hours/days, the measured duration includes the wait — usually what you want.
- The "started" notifications fire in parallel with the job, so they never delay it.

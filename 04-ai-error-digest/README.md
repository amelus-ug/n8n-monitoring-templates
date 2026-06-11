# AI Error Digest – Daily Failure Summary

Instead of a notification for every single failure, get **one** calm, readable summary once a day: what failed, how often, and what to look at first — written by an LLM. Perfect for instances where a noisy error channel has trained you to ignore it.

## Why you need this

Per-failure alerts create alert fatigue: after the hundredth Slack ping you stop reading them, and then you miss the one that mattered. A once-daily digest flips that — it's low-noise, high-signal, and the LLM does the triage so you start your day knowing exactly where to look.

## How it works

```
Daily schedule → list 24h of errors (n8n API) → aggregate by workflow → LLM summarizes → one message
```

1. **Once a day** (default 08:00) it pulls recent `error` executions from the n8n API.
2. **Aggregate Errors** filters to the last `LOOKBACK_HOURS` and groups them by workflow with counts.
3. The **Summarize Errors** LLM chain turns the raw numbers into a short, prioritized, plain-language digest.
4. It's delivered as a single message on your channel of choice.
5. If there were no failures, a quiet "all clear" branch runs instead (disabled by default).

## Setup

1. **Import** `workflow.json`.
2. **Set `N8N_BASE_URL`** and `LOOKBACK_HOURS` (default 24) in the **Config** node.
3. **Add an n8n API key** as an **HTTP Header Auth** credential (Name `X-N8N-API-KEY`) on the `List Failed Executions` node.
4. **Add your LLM credential** to the **Chat Model** node. It's OpenAI (`gpt-4o-mini`) by default — see below to swap providers.
5. **Pick a delivery channel:** Slack (default), Telegram, Email, or WorkflowBuddy.
6. **Activate the workflow.**

## Swapping the LLM provider

The **Chat Model** node is a standard LangChain chat-model sub-node. Delete it and drop in any other — Anthropic, Google Gemini, Groq, Ollama (local), etc. — then connect it to the **Summarize Errors** chain. The prompt and the rest of the workflow stay identical. A small, cheap model is plenty for this task.

## Getting the digest on your iPhone (WorkflowBuddy)

1. Install [**Workflow Buddy**](https://apps.apple.com/app/id6760253861) from the App Store.
2. In the app, open **Settings → Push API** and copy your key.
3. In n8n, create a **Header Auth** credential — Name: `Authorization`, Value: `Bearer wb_…`.
4. Select it on the **Send: WorkflowBuddy (iOS Push)** node and enable the node. The digest is truncated to 1000 characters to fit a push.

## Customization

- **Deeper analysis:** the digest groups by workflow and count. To have the LLM reason over the actual error *messages*, set `includeData=true` on the API call and extend **Aggregate Errors** to pull `data.resultData.error.message` from each execution (heavier payload).
- **Frequency:** change the schedule to twice-daily or weekly.
- **All-clear pings:** enable the "All Clear" branch and wire it to a notification node if you want a daily "0 errors 🎉".

## Pairs well with

- [Global Error Handler](../01-global-error-handler/) — for instant alerts on *critical* failures, with this digest covering everything else.

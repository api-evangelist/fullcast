---
name: Run a Fullcast Copy.ai workflow and collect the result
description: Start a workflow run over HTTP, then collect the output by webhook or by polling, using the acquired Copy.ai Workflows API documented on Fullcast's support host.
api: https://api.copy.ai/api
operations: [Start a Workflow Run, Get Workflow Run, Register Webhook, Get All Webhooks, Remove Webhook]
safety: write - consumes workflow credits
---

# Run a Fullcast Copy.ai workflow

Fullcast acquired Copy.ai in October 2025 and ships it as **Fullcast Copy.ai**. The API still lives on `api.copy.ai`; the reference is published on Fullcast's own support host at `support.fullcast.com/apidocs/`.

This is the only Fullcast surface with a conventional developer API key, and the only one with documented pagination.

## Authenticate

Send the workspace key on every request:

```
x-copy-ai-api-key: <your key>
```

Keys are created under **Configuration > API Keys > Create API Key**, are shown **once and only once**, and **inherit all permissions of the user they are assigned to** — so scope a key by assigning it to a restricted user. Default expiry is never; the provider recommends 12 months. Only Workspace Owners and Admins can manage keys.

## Start a run

```
POST https://api.copy.ai/api/workflow/{workflow_id}/run
x-copy-ai-api-key: <key>

{
  "startVariables": { "Input 1": "value" },
  "metadata": { "yourCorrelationId": "abc-123" }
}
```

Returns `{"status":"success","data":{"id":"<run_id>"}}`.

Put your own correlation id in `metadata` — it is echoed back on the `workflowRun.completed` webhook, and it is the only correlation mechanism available. There is no request-id header anywhere in the platform.

## Collect the result — pick one

**Webhook (preferred).** Register once:

```
POST https://api.copy.ai/api/webhook
{ "url": "https://you.example.com/hook",
  "eventType": "workflowRun.completed",
  "workflowId": "<optional - omit for all workspace workflows>" }
```

Events: `workflowRun.started`, `workflowRun.completed`, `workflowRun.failed`, `workflowCreditLimit.reached`.

> **Deliveries are unsigned.** No signature header, shared secret or timestamp is documented, so a receiver cannot verify a payload came from Copy.ai. Use an unguessable callback URL and re-verify results with `Get Workflow Run` before acting on anything consequential.

**Polling.** `GET https://api.copy.ai/api/workflow/{workflow_id}/run/{run_id}` until `status` moves from `PROCESSING` to `COMPLETE`.

> The published polling recipe has a bug: it reuses `workflow_url` instead of `run_url` in the GET step, so the sample polls the wrong URL. That page is also flagged stale (February 2026). Build the run URL yourself.

## Manage webhooks

`GET /api/webhook?size=<=100&page=<0-indexed>` lists them (`{total, data[]}`); `GET /api/webhook/{id}` reads one; `DELETE /api/webhook/{id}` removes one.

## Errors

| Status | Meaning |
|---|---|
| 400 | Bad request, or `WORKFLOW_CREDIT_LIMIT_REACHED` |
| 404 | Workflow not found |
| 422 | Validation error (`status`, `errorCode`, `details`) |

Runs consume workflow credits. Subscribe to `workflowCreditLimit.reached` rather than discovering exhaustion as a 400 mid-batch. No rate limits are documented.

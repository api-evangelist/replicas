---
name: Create and trigger an automation
description: Create a Replicas automation that runs coding agents on a schedule or in response to webhook events, trigger it, and review its execution history.
api: openapi/replicas-openapi-original.json
operations: [listEnvironments, createAutomation, triggerAutomation, listAutomationExecutions]
---

# Create and trigger an automation

Base URL: `https://api.tryreplicas.com`. Auth: `Authorization: Bearer sk_replicas_...`.

## Steps

1. **Pick the environment.** `GET /v1/environments` (`listEnvironments`) - the
   automation binds to one `environment_id`, which resolves the repo, variables,
   MCPs, and skills used when it fires.
2. **Create the automation.** `POST /v1/automations` (`createAutomation`) with
   `name`, `prompt`, `environment_id`, and `triggers` (cron, or GitHub/GitLab/
   Slack/Sentry/custom-webhook events). Set `enabled: true`.
3. **Run it.** `POST /v1/automations/{id}/trigger` (`triggerAutomation`) to run
   manually; cron automations trigger directly, and a GitHub
   `pull_request.command` automation can target a specific PR via `pr`. Each run
   creates a new workspace + execution.
   - For a custom webhook trigger, external callers POST to
     `/v1/automations/webhook/{token}` (`fireCustomWebhookAutomation`) - the URL
     token is the secret and the endpoint is unauthenticated by design.
4. **Review runs.** `GET /v1/automations/{id}/executions`
   (`listAutomationExecutions`) for paginated execution history.

## Rules

- Automations archive workspaces instead of deleting them (history is preserved).
- Treat the custom-webhook token as a secret - anyone with the URL can fire it.
- Errors return `{error, details}`; check `getAutomation` before retrying a
  create that timed out (mutations are not idempotent).

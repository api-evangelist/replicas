---
name: Create a replica and drive it to a pull request
description: Spin up a Replicas cloud workspace (a "replica") against a repository, give the coding agent a task, follow its progress, and let it open a pull request.
api: openapi/replicas-openapi-original.json
operations: [listRepositories, listEnvironments, createReplica, sendReplicaMessage, readReplicaHistory, getReplica]
---

# Create a replica and drive it to a pull request

Base URL: `https://api.tryreplicas.com`. Authenticate every request with
`Authorization: Bearer sk_replicas_...` (API key from Organization -> Settings ->
API Keys). Errors return `{ "error": "...", "details": ... }` (not RFC 9457).

## Steps

1. **Pick a repository.** `GET /v1/replica/repositories` (`listRepositories`) to
   find the `id`/name of the repo the agent should work in. Optionally
   `GET /v1/replica/repository-sets` (`listRepositorySets`).
2. **Pick an environment.** `GET /v1/environments` (`listEnvironments`) and note
   the `environment_id` that binds the right repo, variables, skills, and MCPs.
3. **Create the replica.** `POST /v1/replica` (`createReplica`) with `message`
   (the task), `environment_id`, and `repository_ids` or `repository_set_id`.
   Optionally set `coding_agent`, `model`, `plan_mode`, and `webhook_url` for a
   delivery callback. Capture the returned replica `id`.
4. **Follow along.** Poll `GET /v1/replica/{id}` (`getReplica`) for status, or
   read the transcript with `GET /v1/replica/{id}/history` (`readReplicaHistory`,
   paginates bottom-up). If the workspace is asleep the response returns
   `waking: true` - retry in 30-90 seconds.
5. **Iterate.** `POST /v1/replica/{id}/messages` (`sendReplicaMessage`) to send
   follow-up instructions (e.g. "address the failing CI check").
6. The agent opens a pull request in the connected repository; find PR details in
   the replica/history payload.

## Rules

- Mutations are not idempotent - do not blindly retry `createReplica`; check
  `listReplicas` first if a call times out.
- Respect `waking: true` / 503 by backing off 30-90s.
- Keep API keys server-side; the `sk_replicas_` prefix denotes a live secret.

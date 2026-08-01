---
name: Configure an environment (variables, files, skills, MCPs)
description: Create a Replicas environment - the org-scoped primitive workspaces are built from - and load it with variables, files, skills, and MCP servers.
api: openapi/replicas-openapi-original.json
operations: [createEnvironment, createEnvironmentVariable, createEnvironmentFile, createEnvironmentSkill, createEnvironmentMcp, getEnvironment]
---

# Configure an environment

Base URL: `https://api.tryreplicas.com`. Auth: `Authorization: Bearer sk_replicas_...`.

## Steps

1. **Create the environment.** `POST /v1/environments` (`createEnvironment`).
   `repository_id` and `repository_set_id` are mutually exclusive; omit both for
   an unbound environment. Use `scope=user` for a personal environment. Capture
   the `id`.
2. **Add variables.** `POST /v1/environments/{environmentId}/variables`
   (`createEnvironmentVariable`). Values are encrypted at rest; `(environment,
   key)` must be unique in the org.
3. **Add files.** `POST /v1/environments/{environmentId}/files`
   (`createEnvironmentFile`). Max content 64 KB (65,536 bytes); `path` is
   validated against allowed locations (typically under `~/`).
4. **Enable skills.** `POST /v1/environments/{environmentId}/skills`
   (`createEnvironmentSkill`). Search the catalog first with
   `GET /v1/environment-skills/search` (`searchEnvironmentSkills`).
5. **Attach MCP servers.** `POST /v1/environments/{environmentId}/mcps`
   (`createEnvironmentMcp`). `name` must match
   `^[A-Za-z0-9][A-Za-z0-9_-]{0,63}$` and cannot start with the reserved prefix
   `replicas-`. `config` shape depends on `transport`: stdio = `{command, args,
   env}`; http/sse = `{url, headers}`.
6. **Verify.** `GET /v1/environments/{id}` (`getEnvironment`).

## Rules

- Deleting an environment returns 409 while an automation references it, while
  personal environments inherit from it, or for the Global environment.
- Errors return `{error, details}`; a 400 usually means a validation failure
  (size limit, bad path, duplicate key, mutually exclusive fields).

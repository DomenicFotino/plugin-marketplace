---
name: neon
description: Guide Neon Postgres, authentication, branches, object storage, functions, and AI Gateway work.
---

# Neon

Use this guide when the task involves Neon. Honor the user's selected provider,
account, project, branch, environment, and deployment scope. Neon offers
Postgres, managed authentication, object storage, functions, and an AI Gateway;
service availability, regions, limits, pricing, and access can change. Check
[current Neon documentation](https://neon.com/docs/llms.txt) and the actual
provider state when those details affect the task.

## Choose the right guidance

Use the applicable installed skill if available: `neon-postgres` for Postgres,
connections, branching, or CLI/MCP work; `neon-postgres-branches` for branch
operations; and the relevant Neon auth, storage, functions, or AI Gateway skill
for those services. Otherwise use the official docs. Avoid installing skills,
CLIs, extensions, or MCP servers merely to answer a question.

## Inspect the existing setup

Before changing a project, inspect the relevant code and nonsecret configuration
and identify the actual organization, project, branch, environment, and
consumer. Prefer an already connected Neon MCP or installed CLI that supports
the task; respect an explicit tool preference. Check installed CLI help and
current documentation rather than assuming an old command or API shape.
Never print or commit credentials or connection strings; access secrets only
through authorized mechanisms when the task requires them.

Reuse existing resources when appropriate. Do not automatically create a new
project, migrate data, change a branch target, run initialization, install
software, or pull environment files. Perform such actions only when they are
within the user's authorization and the selected project and consequences are
understood. Existing authorization covers the requested work; don't add an
unrelated approval gate for routine steps.

## Branches, schemas, and services

Use an isolated branch for changes when appropriate and authorized. Check its
parent, data exposure, ownership, consumers, and the actual target before
creating, switching, applying schema changes, or deleting anything. Treat
production and shared branches as shared state. Follow the project's migration
workflow and test schema and access changes against the appropriate environment.

When using `neon.ts` or other infrastructure-as-code, inspect existing config
and consult the current official schema. Review the planned changes before
applying them; distinguish a local file change, a provider change, and a deployed
result. Do not run an apply, deployment, or environment pull merely because a
skill mentions it.

## Credentials and verification

Keep credentials server-side in the project's approved secret management or
runtime injection mechanism. Do not expose secrets to the browser, logs, version
control, or generated examples. Select least-privilege roles and access controls.

Run checks appropriate to the changed code and verify the resulting project,
branch, service, and runtime state with the owning provider when relevant.
Report missing access or unexecuted checks without inventing success.

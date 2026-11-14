---
name: neon-postgres-branches
description: >-
  Choose and create the right Neon branch type for testing and development.
  Use when users ask about Neon branching, migration testing with real data,
  isolated test environments, schema-only branch workflows for sensitive data,
  or branch creation via Neon CLI or Neon MCP. Triggers include "Neon branch",
  "test migrations safely", "branch production data", "schema-only branch",
  "reset branch" and "sensitive data testing".
---

# Neon Postgres Branching

Answer the requested branching question or prepare a plan at the requested scope. Create or reset a branch only when that action is authorized and the project, source, target, data sensitivity, permissions, cost and retention are understood. A data-bearing branch contains source data; isolation alone does not authorize copying it. Preserve existing branches and consumers.

- **Normal branch** for realistic migration and query testing with real data.
- **Schema-only branch (Beta)** for sensitive data workflows where structure is needed without copying rows.

## Branch Type Decision

Use this decision rule first:

1. If realistic data is required and its use in the selected environment is authorized, consider a **normal branch** from the appropriate source; otherwise use an approved sanitized source or schema-only branch.
2. If the user needs to avoid copying sensitive data, choose a **schema-only branch**.

If the request is ambiguous, ask one clarifying question:
"Do you need realistic data for testing, or only schema structure because the data is sensitive?"

## Tool Selection: CLI or MCP

Always support both Neon CLI and Neon MCP server. Prefer the tool the user already has installed and authenticated.

MCP link: https://neon.com/docs/ai/neon-mcp-server.md
CLI link: https://neon.com/docs/reference/cli-quickstart

### Selection order

1. Check MCP first in MCP-enabled environments:
   - If Neon MCP tools are available and authenticated (for example, listing projects works), use MCP.
2. If MCP is unavailable or not authenticated, check CLI:
   - Run `neonctl --version` to confirm CLI is installed.
   - Run `neonctl projects list` to confirm auth/context.
3. If CLI is missing, answer from current documentation or propose installation when it is needed for authorized work.
4. If CLI is installed but not authenticated, explain the needed authentication and continue only with the appropriate account and scope.
5. If both MCP and CLI paths are unsuccessful, use the Neon REST API:
   - https://neon.com/docs/guides/branching-neon-api.md

### MCP branch flow

1. Choose normal vs schema-only based on data sensitivity and migration-testing goals.
2. Use branch tools (for example, `create_branch`) to create the branch.
3. Validate with read tools (for example, `describe_branch`).
4. For migration workflows, prefer branch-based migration flows before applying to main.

## Before any branch creation

Only set an expiration when the agreed retention permits automatic permanent deletion. Check the Console auto-delete default and clear it if no expiration is authorized, then verify the resulting retention. If a timestamp is authorized, use a valid future value from the current [expiration documentation](https://neon.com/docs/guides/branch-expiration), not a copied date. CLI creation may print credentials; `--no-secrets` requires verified Neon CLI 4.9.0 or newer. Older versions may ignore it: use an appropriate secure alternative.

## Create a Normal Branch (Preferred for Real-Data Migration Testing)

Use this when realistic testing conditions and the selected data copy are authorized. Production-like data may expose edge cases but can include sensitive records; verify the source and intended access first.

Link: https://neon.com/docs/introduction/branching.md

### Steps

1. Use MCP if already available/authenticated; otherwise verify CLI with `neonctl --version`.
2. Select the exact project using `--project-id` on the command; change persistent CLI context only when that configuration change is authorized.
3. Create branch:

```bash
neonctl branches create \
  --name <branch-name> \
  --parent <parent-branch-id-or-name> \
  --project-id <your-project-id> \
  --no-secrets
```

4. Retrieve credentials only when needed and deliver them through an authorized secret destination; do not expose connection strings in chat, logs, or shell output.

## Create a Schema-Only Branch (Beta, Sensitive Data)

Use this when users must not copy production rows into the test branch.

Link: https://neon.com/docs/guides/branching-schema-only.md

### Steps

1. Use MCP if already available/authenticated; otherwise verify CLI with `neonctl --version`.
2. Create schema-only branch:

```bash
neonctl branches create \
  --name <schema-only-branch-name> \
  --parent <parent-branch-id-or-name> \
  --schema-only \
  --project-id <your-project-id> \
  --no-secrets
```

If multiple projects exist, include:

```bash
neonctl branches create \
  --name <schema-only-branch-name> \
  --parent <parent-branch-id-or-name> \
  --schema-only \
  --project-id <your-project-id> \
  --no-secrets
```

### Beta Support Guidance (Mandatory)

Schema-only branching is in Beta. If users report unexpected behavior, errors, or missing capabilities:

1. Ask them to share feedback in the Neon Console:
   - https://console.neon.tech/app/projects?modal=feedback
2. Recommend opening a support conversation in the Neon Discord:
   - https://discord.gg/92vNTzKDGp

## Reset from parent

Use this when a child branch has drifted and the user wants a clean refresh from the parent branch's latest schema and data. Before a reset, verify the exact project, target, parent, owner, consumers and recoverable pre-reset state; the action replaces child changes and is not an ordinary read-only check.

Link: https://neon.com/docs/guides/reset-from-parent.md

### What it does

- Fully replaces the child branch schema and data with the parent's latest state.
- Does not merge; local changes on the child branch are lost.
- Keeps the same connection details, but active connections are briefly interrupted during reset.

### When to recommend it

- Development or staging branch is too far behind production.
- User wants to start a new feature from a clean parent-aligned state.
- Team wants to refresh staging from production for consistent testing baselines.

### Hard constraints and blockers

- Only child branches can be reset (root branches and schema-only root branches cannot be reset from parent).
- If the target branch has children, the provider blocks reset. Preserve those children unless their exact removal is authorized; do not delete them merely to unblock the reset.
- After a parent branch is restored from snapshot, reset-from-parent may be unavailable for up to 24 hours.
- Reset-from-parent always uses the current parent state; use Instant restore for point-in-time recovery needs.

### CLI usage

```bash
neonctl branches reset <id|name> --parent --preserve-under-name <backup-branch-name>
```

If project context is not already set, include project ID:

```bash
neonctl branches reset <id|name> --parent --preserve-under-name <backup-branch-name> --project-id <project-id>
```

`--preserve-under-name` keeps the pre-reset state as a backup branch for rollback; retain it according to the agreed recovery and retention requirements.

Prefer `--project-id` for the selected project; change persistent CLI context only when authorized for the task.

### Console and API usage

- **Console:** Open the target child branch, then select **Reset from parent** from **Actions**.
- **API:** Use the restore endpoint for the branch and set `source_branch_id` to the parent branch ID.

## Notes and Caveats

- Schema-only branches are for structure-only cloning and sensitive/compliant data controls.
- A newly created schema-only branch is an independent root with no shared history, so reset-from-parent does not initially apply. A later restore may copy data and assign a parent; verify actual branch state before treating it as schema-only.
- For migration testing that depends on real-world row shapes, volumes, and edge cases, prefer normal branches.
- Root branch allowances and per-branch storage limits can cap how many schema-only branches users can create.
- If a user is unsure, default recommendation is:
  - **Normal branch** for migration validation.
  - **Schema-only branch** for compliance and privacy constraints.

## Useful Workflow Patterns

If the user asks for process recommendations (not just a single command), suggest these:

- **One branch per PR:** Where authorized, isolate migration tests; configure lifecycle and cleanup only for the named disposable branch with agreed retention.
- **One branch per test run:** Where authorized, use isolated disposable branches with an explicit retention and cleanup policy.
- **One branch per developer:** Isolated dev environments with production-like shape; avoid team collisions on shared test data.
- **PII-aware branching:** If production has sensitive data, derive dev/PR branches from an anonymized branch or use schema-only branches.
- **Ephemeral lifecycle hygiene:** Consider expiration or automated cleanup only for the named disposable branches and agreed retention.

### Post-creation environment updates

Update a specified environment file or key when that change is authorized in the current task, preserving unrelated values and using an approved secret path. If the requested scope does not establish such a change, explain the available next step. Never paste credentials into a response or log.

## Branching in CI/CD

Common CI/CD use cases for Neon branches:

- **Per-PR preview deployments:** Where authorized, branch on PR open and deploy the preview against it; delete only a named disposable branch under the agreed lifecycle. Each PR gets an isolated database branch. Injecting the branch's `DATABASE_URL` into the deployed app is hosting-provider-specific — see [preview-branches-with-cloudflare](https://github.com/neondatabase/preview-branches-with-cloudflare), [preview-branches-with-vercel](https://github.com/neondatabase/preview-branches-with-vercel), or [preview-branches-with-fly](https://github.com/neondatabase/preview-branches-with-fly) for tested patterns.
- **Migration testing in CI:** Run risky schema changes against a branch with production-like data before merge.
- **Schema diff visibility:** Use the [schema-diff GitHub Action](https://github.com/marketplace/actions/neon-schema-diff-github-action) to auto-comment a DB-layer diff on the PR.

## Examples

### Example 1: Migration testing with realistic data

**User input:** "I need to test a risky migration against production-like data."

**Agent output shape:**

1. Recommend a normal branch and explain why.
2. Share docs link: https://neon.com/docs/introduction/branching
3. Check the available/authenticated tool path first (MCP, otherwise CLI with `neonctl --version`).
4. Provide commands:
   - `neonctl branches create --name migration-test --parent main --project-id <your-project-id> --no-secrets`
   - Retrieve credentials only through the authorized secret workflow if needed.

### Example 2: Sensitive data development workflow

**User input:** "We cannot copy production data because of compliance."

**Agent output shape:**

1. Recommend schema-only branch and explain why.
2. Share docs link: https://neon.com/docs/guides/branching-schema-only
3. Check the available/authenticated tool path first (MCP, otherwise CLI with `neonctl --version`).
4. Provide command:
   - `neonctl branches create --name compliance-dev --parent main --schema-only --project-id <your-project-id> --no-secrets`
5. Mention Beta support path:
   - https://console.neon.tech/app/projects?modal=feedback
   - https://discord.gg/92vNTzKDGp

## Further reading

- https://neon.com/docs/guides/branch-expiration.md
- https://neon.com/docs/guides/neon-github-integration.md
- https://neon.com/docs/ai/neon-mcp-server.md
- https://neon.com/branching

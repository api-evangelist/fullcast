---
name: Move an account, person or product safely in Fullcast
description: Stage a hierarchy move, verify it, commit it, and reverse it if wrong - the correct write discipline for an API with no idempotency and no sandbox.
api: https://app.fullcast.io/mcp
mcp_tools: [resolve_entity, explain_placement, get_assignment_suggestions, move_account, move_person, move_product, commit_changes, undo_changes, rerun_territories_rules, list_audit_entries, explain_audit_entry]
mcp_resources: ['fullcast://resource/workflow_agent_guidance', 'fullcast://resource/move_account_guidance', 'fullcast://resource/move_people_guidance', 'fullcast://resource/move_product_guidance']
safety: write - production only
---

# Move a record safely in Fullcast

**Read this first.** Fullcast has:

- **no idempotency key** on any surface — a retried `move_account` is a second move, not a no-op;
- **no developer sandbox** — the "Sandbox Environment" is a paid add-on, not a test tenant, so every write lands on production tenant data;
- **no published rate limit** and no `Retry-After`, so a failed call gives you no guidance on when to retry.

The staging tools below are the compensating control. Use them.

## Read the provider's own guidance first

The assistant MCP server exposes guidance as MCP **resources**. Fetch the relevant one before writing:

- `fullcast://resource/workflow_agent_guidance`
- `fullcast://resource/move_account_guidance`
- `fullcast://resource/move_people_guidance`
- `fullcast://resource/move_product_guidance`

## Steps

1. **Resolve the record.** `resolve_entity` to get an ID. Never pass a display name to a write tool.
2. **Understand the current placement.** `explain_placement` tells you why the record sits where it does — often the rule, not the record, is what actually needs changing.
3. **Check the target.** For coverage changes, `get_assignment_suggestions` proposes assignments rather than you guessing.
4. **Stage the move.** `move_account`, `move_person` or `move_product`. Treat this as staged, not applied.
5. **Verify before committing.** Re-run `explain_placement` and `analyze_coverage` to confirm you did not open a coverage gap.
6. **Commit.** `commit_changes`. If wrong, `undo_changes`.
7. **Re-run rules if placement was rule-driven.** `rerun_territories_rules`, `rerun_team_rules`, `rerun_product_rules`. A manual move that fights an active rule will be re-derived.
8. **Confirm in the audit trail.** `list_audit_entries`, then `explain_audit_entry` for a plain-language read of what happened.

## Named-account overrides

`name_accounts` pins accounts to a node regardless of rules; `clear_named_accounts` removes the pin. Prefer a rule change over a growing pile of overrides.

## Retry rule

**Never blind-retry a write.** On a timeout or 5xx, re-read state with `explain_placement` or `get_account` and decide from observed state. Because there is no idempotency key, a retry that "looked" failed but succeeded produces a double move.

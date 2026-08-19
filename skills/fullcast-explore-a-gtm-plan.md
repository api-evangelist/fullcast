---
name: Explore a Fullcast go-to-market plan read-only
description: Orient in a tenant and read its territories, teams, products, accounts, people, coverage and targets without changing anything.
api: https://app.fullcast.io/mcp
mcp_tools: [get_tenant_context, list_plans, describe_entities, get_territories, get_teams, get_product_families, list_accounts, list_people, list_targets, list_assignments, resolve_entity, deep_search, explain_placement, analyze_coverage]
safety: read-only
---

# Explore a Fullcast go-to-market plan (read-only)

Every tool here is read-only. Nothing in this skill mutates tenant data.

## Why you must start with discovery

Fullcast publishes **no schema** for its planning entities. The only public OpenAPI covers the Assistant (chat and OAuth) — not territories, accounts or targets. Field shapes exist solely at runtime, so discovery is not optional.

## Steps

1. **`get_tenant_context`** — tenant identity, currency, fiscal periods, current plan. Do this first; currency and fiscal calendar change how every number reads. Tenants may run standard, 4-4-5 or 13-period calendars.
2. **`list_plans`** — the territory, team and product plans available.
3. **`describe_entities`** — field schemas. **This is your schema source.** Also `describe_entity_sources` for provenance and `describe_report_fields` for reporting dimensions and measures.
4. **Resolve names to IDs.** `resolve_entity` for a fast exact name-to-ID lookup; `deep_search` for fuzzier matching; `search_entities` on the assistant server.
5. **Walk the hierarchies** — `get_territories`, `get_teams`, `get_product_families`; or `get_territory_heirarchy`, `get_teams_hierarchy`, `get_products_hierarchy` on the assistant server (the misspelling of *hierarchy* is the provider's, reproduce it verbatim).
6. **List records** — `list_accounts`, `list_people`, `list_products`, all with filtering and joins.
7. **Read coverage and targets** — `list_assignments` for role-based coverage, `list_targets` for quotas including attainment and pacing.
8. **Ask why, not just what** — `explain_placement` explains why a record landed in its current node; `analyze_placement` reads patterns across a set; `analyze_coverage` finds gaps and overlaps; `analyze_org_chart` and `analyze_account_family` read structure.
9. **Report** — `execute_metric` for a defined metric, `report` for ad hoc. Use `list_metrics` first.

## Conventions that bite

- **No pagination contract is published anywhere.** Assume list tools may truncate and cross-check totals with `report`.
- **No rate limits are published and no rate-limit headers are returned.** There is no backpressure signal, so pace bulk reads yourself.
- **History is queryable.** `get_account_history`, `get_people_history`, `get_product_history` return dated move events that survive record deletion.

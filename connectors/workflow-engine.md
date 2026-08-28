# Connector Pack — Workflow Engine (n8n)

*Core-infrastructure pack (§7.6). The Workflow Engine is part of the standard stack, not a business capability: it is chosen by Baxter from defaults, not requested by the owner. This pack tells Baxter what n8n is, how to author workflows safely, and how to verify them.*

## Version

1.0.0

## Purpose

Deterministic automation — cross-connector syncs, scheduled jobs, webhook handling, routing and transformation. No judgment, no content generation — ever (Constitution §2).

## Product

**n8n** — open source, self-hosted, MCP-native automation. Reached MCP-first via the official n8n MCP server (`https://<n8n-domain>/mcp-server/http`, Bearer token from the vault).

- Deployment: `internal` (Docker co-located on the same VPS as the harness — default) or `external` (separate host).
- Docs: `https://docs.n8n.io` · MCP: `https://docs.n8n.io/connect/connect-to-n8n-mcp-server`

## Native-Integration Precedence (§5 rule 6)

Before building any workflow, check for a native path — a vendor-native integration between the two connectors, or a direct MCP/API call. Native wins; build an n8n workflow **only where no native path exists**. Typical engine-owned work:

- Cross-connector syncs with no native path (e.g., CRM → email list sync)
- Scheduled jobs (nightly reports, list refreshes)
- Inbound webhook handling and transformation (e.g., website lead → CRM)

## Onboarding Questions (Stage 3)

Minimal — the engine is core infrastructure; the owner is never asked about it. Baxter records defaults and confirms the endpoint during setup.

| ID | Ask | Maps to |
|---|---|---|
| Q1 | *(internal, default)* — no owner question; endpoint defaults to `http://localhost:5678` | `workflow_engine.endpoint` |
| Q2 | *(external only, when the operator deploys one)* — "What is the n8n address on your server?" | `workflow_engine.endpoint` |

Credentials are never collected at onboarding (§7.4) — the vault path `vault/connectors/n8n` is recorded, and setup verifies it.

## Proficiency (after lock-in)

- [ ] n8n MCP server reachable; tool surface enumerated from the live self-description (§8.3)
- [ ] Auth method understood — MCP bearer token from the vault; values never seen (§7.4)
- [ ] Tool surface confirmed: `search_nodes`, `get_node_types`, `search_workflows`, `get_workflow_details`, `test_workflow`, `validate_workflow`, `publish_workflow`
- [ ] Existing workflows searched (`search_workflows`) before creating anything — reuse first

## Authoring Standards (framework §7.6 rule 3)

Baxter authors every workflow under these standards — the goal is consistent, maintainable, deterministic results:

1. **Never write workflow JSON from memory.** Discover nodes with `search_nodes`; verify every node's schema with `get_node_types` before building; never guess parameter shapes.
2. **Native nodes over Code nodes.** Prefer built-in nodes (HTTP Request, Set/Edit Fields, IF, Switch, Merge, Filter, Aggregate); use a Code node only when logic cannot be expressed with native nodes or expressions — and say why.
3. **Modular, small, reusable.** Prefer small specialised sub-workflows called from parents (`Execute Workflow` node); name them by role — `sub: sync-contacts-to-email`, `sub: format-invoice` — and search existing workflows before adding logic anywhere.
4. **One "Configuration" node.** A Set/Edit Fields node named "Configuration", directly after the trigger, holding every user-adjustable variable (snake_case, typed). Downstream nodes reference it by expression — no magic values scattered across nodes. **Never secrets** — credentials live in n8n's credential store, set by humans (§7.4 rule 6).
5. **Test → validate → verify → activate.** No workflow goes live without: `test_workflow` (with pinned test data), `validate_workflow`, and `get_workflow_details` to confirm structure. Record the workflow ID in the workspace (§8.4 Document).

## Configuration Checklist

- [ ] Confirm n8n deployment mode (`internal` / `external`) and endpoint reachability
- [ ] Verify MCP bearer token exists in the vault (`vault/connectors/n8n` — existence only)
- [ ] Enumerate MCP tool surface; confirm workflow authoring tools available
- [ ] Confirm existing workflow inventory (`search_workflows`)

## Sync Topology (declared syncs)

The engine executes the `via: n8n` entries of the deployment's `sync_topology` (§7.6). It declares no syncs of its own.

## Secrets (vault paths)

- `vault/connectors/n8n/…` — MCP bearer token (values set by the operator, never in chat)

## Verification (smoke tests)

- [ ] `test_workflow` passes for every declared `via: n8n` sync
- [ ] End-to-end effect confirmed (e.g., test record in CRM lands in the email list)
- [ ] `validate_workflow` returns clean for every deployed workflow
- [ ] Structure verified via `get_workflow_details` (connections intact)

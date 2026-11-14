# Connector Pack — Template

> Copy this file per connector and complete every section before the connector is enabled in a deployment. A Connector Pack is lookup material for Baxter — auditable, updateable, and swappable when a connector is replaced (§8.3).
> **Versioning:** bump `Version` on every change; deployments pin this version in `deployment.yaml` (`pack_version`) so what is running is always exactly knowable (§7.2 rule 5).

## Version

1.1.0

## Purpose

What this connector provides to the business.

## Capabilities

- …

## Kind and Surfaces

- Kind: `business` | `automation` | `broker` (§5)
- Agent surface: `mcp` | `openapi` | `rest` | `none` — how Baxter converses with it
- Flow surface: `rest` | `webhook` | `native` | `none` — how a flow calls it
- Both surfaces are chosen once per connector and pinned in `deployment.yaml`.

## Connection

- Endpoint / MCP server: …
- Auth: values come from the declared secret source — never stored here
- Scopes: least privilege, listed in the connector's `secrets.scope`

## Onboarding Questions (Stage 3)

Asked by Baxter during onboarding (`references/onboarding-protocol.md`) when this connector is selected. Every question maps to a config destination; **credentials are never collected at onboarding** (§7.4) — the vault path is recorded, and setup verifies it.

| ID | Ask | Maps to |
|---|---|---|
| Q1 | … *(lock-in — may be deferred)* | … |

## Proficiency (after lock-in)

Once the product is locked in, Baxter learns it (§8.3) and records a Connector Proficiency profile in the deployment. What must be learned — no credentials, no configuration:

- [ ] Product and plan identified; MCP server endpoint confirmed (or documented REST API)
- [ ] Auth method understood (type and shape only — values live in the vault, §7.4)
- [ ] Tool/resource surface enumerated from the live self-description (§8.3)
- [ ] Capabilities mapped to this deployment's needs
- [ ] Limitations and gaps recorded

## Configuration Checklist

Driven by the Setup Protocol — Apply (§8.4 step 5). Credentials arrive via the vault, never through onboarding or chat (§7.4).

- [ ] …

## Sync Topology (declared syncs)

- From: … → To: … — What: …

## Secrets (vault paths)

- `vault/connectors/<name>/…`

## Verification (smoke tests)

Run by the Setup Protocol — Verify (§8.4 step 6). Every declared sync must fire end-to-end before the deployment is done.

- [ ] …

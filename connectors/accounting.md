# Connector Pack — Accounting

## Version

1.1.0

## Purpose

Invoicing, payments, and reconciliation — the deterministic financial backbone.

## Capabilities

- Invoice creation and payment capture
- Bank feed reconciliation
- Contact sync from CRM

## Kind and Surfaces

- Kind: `business`
- Agent surface: `mcp` | `rest` — how Baxter converses with this service; `none` if it is not safe or useful to expose
- Flow surface: `rest` | `webhook` | `native` | `none` — how a flow calls it; `none` if it cannot be an endpoint
- Both are chosen once per connector and pinned in `deployment.yaml`; they are never re-decided per deployment.

## Connection

- Endpoint / MCP server: …
- Auth: values come from the deployment's declared secret source (`vault`, `native` or `hybrid`) — never stored here
- Scopes: least privilege, listed in the connector's `secrets.scope`

## Onboarding Questions (Stage 3)

| ID | Ask | Maps to |
|---|---|---|
| Q1 | "Which accounting software do you use — or have in mind?" *(lock-in — may be deferred)* | `connectors` entry |
| Q2 | "Are bank feeds already set up and reconciling?" | Configuration Checklist — bank feeds |
| Q3 | "How should invoices link back to jobs or customers — do you use reference numbers?" | Configuration Checklist — invoice-to-job link |

## Proficiency (after lock-in)

- [ ] Product and plan identified; MCP server endpoint confirmed (or documented REST API)
- [ ] Auth method understood (type and shape only — credentials stay in the vault, §7.4)
- [ ] Tool/resource surface enumerated from the live self-description (§8.3)
- [ ] Capabilities mapped to this deployment: invoicing, payments, reconciliation, contact sync
- [ ] Bank-feed and invoice-to-job link mechanisms confirmed

## Configuration Checklist

- [ ] Create API credential (scoped, least privilege)
- [ ] Configure bank feeds
- [ ] Confirm invoice-to-job link mechanism with CRM

## Sync Topology (declared syncs)

- From: accounting → CRM — What: invoice status updates job
- From: CRM → accounting — What: customer/job creation for invoicing

## Secrets (vault paths)

- `vault/connectors/accounting/…`

## Verification (smoke tests)

- [ ] Pull a test invoice
- [ ] Push a test contact
- [ ] Confirm bank feed flows in

# Connector Pack — CRM

## Version

1.0.0

## Purpose

Customer relationship management — the single system of record for customer records (§5 rule 3).

## Capabilities

- Customer records, jobs/quotes, history
- Contact matching and de-duplication
- Inbound lead creation

## Connection

- Standard: `mcp` | `rest` — **to be decided per deployment** (§5 rule 5)
- Endpoint / MCP server: …
- Auth: …

## Onboarding Questions (Stage 3)

| ID | Ask | Maps to |
|---|---|---|
| Q1 | "Which job management / CRM tool do you use today — or have in mind?" *(lock-in — may be deferred)* | `connectors` entry |
| Q2 | "How do you identify a job — a job number, customer name and address, or something else? I'll use this to confirm details before acting." | Configuration Checklist — job/contact identifiers |
| Q3 | "Does your CRM support attaching photos and documents to a job?" | Configuration Checklist — attachment support |

## Proficiency (after lock-in)

Once the product is locked, Baxter learns it (protocol rule 9) and records `connector-proficiency/<name>.md`:

- [ ] Product and plan identified; MCP server endpoint confirmed (or documented REST API)
- [ ] Auth method understood (type and shape only — credentials stay in the vault, §7.4)
- [ ] Tool/resource surface enumerated from the live self-description (§8.3)
- [ ] Capabilities mapped to this deployment's needs: records, jobs/quotes, contact matching, lead creation
- [ ] Limitations and gaps recorded (e.g., no document attachment → open item)

## Configuration Checklist

- [ ] Create API credential (scoped, least privilege)
- [ ] Define job/contact identifiers used in confirmation gates
- [ ] Confirm photo/document attachment support

## Sync Topology (declared syncs)

- From: CRM → email-marketing — What: new contact syncs to tagged list
- From: accounting → CRM — What: invoice status updates job
- From: website → CRM — What: lead form creates job/contact

## Secrets (vault paths)

- `vault/connectors/crm/…`

## Verification (smoke tests)

- [ ] Pull a test contact
- [ ] Create a test job
- [ ] Attach a test document/photo

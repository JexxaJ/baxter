# Connector Pack — Email Marketing

## Version

1.0.0

## Purpose

Campaigns and transactional sends — bulk marketing plus one-off transactional email through one platform.

## Capabilities

- Contact/tag sync from CRM
- Campaign creation and scheduling
- Transactional sends (agent-drafted follow-ups)

## Connection

- Standard: `mcp` | `rest` — **to be decided per deployment** (§5 rule 5)
- Endpoint / MCP server: …
- Auth: …

## Onboarding Questions (Stage 3)

| ID | Ask | Maps to |
|---|---|---|
| Q1 | "Which email platform do you use (or want to use) for marketing and transactional email?" *(lock-in — may be deferred)* | `connectors` entry |
| Q2 | "What email address should campaigns and automated sends come from?" | Configuration Checklist — sender identity |
| Q3 | "Do you have existing contact lists to import, and were recipients opted in?" | Configuration Checklist — consent handling |

## Proficiency (after lock-in)

- [ ] Product and plan identified; MCP server endpoint confirmed (or documented REST API)
- [ ] Auth method understood (type and shape only — credentials stay in the vault, §7.4)
- [ ] Tool/resource surface enumerated from the live self-description (§8.3)
- [ ] Capabilities mapped to this deployment: contact/tag sync, campaigns, transactional sends
- [ ] Consent/deliverability requirements noted for the region

## Configuration Checklist

- [ ] Create API credential (scoped, least privilege)
- [ ] Configure sender identity + deliverability (SPF/DKIM/DMARC)
- [ ] Confirm consent/opt-in handling for the region

## Sync Topology (declared syncs)

- From: CRM → email-marketing — What: new contact syncs to tagged list

## Secrets (vault paths)

- `vault/connectors/email-marketing/…`

## Verification (smoke tests)

- [ ] Sync a test contact/tag
- [ ] Send a transactional test email

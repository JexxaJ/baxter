# Connector Pack — Email Marketing

## Version

1.1.0

## Purpose

Campaigns and transactional sends — bulk marketing plus one-off transactional email through one platform.

## Capabilities

- Contact/tag sync from CRM
- Campaign creation and scheduling
- Transactional sends (agent-drafted follow-ups)

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

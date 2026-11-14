# Connector Pack — Website

## Version

1.1.0

## Purpose

Public presence and lead capture — the business's front door.

## Capabilities

- Public pages
- Lead / enquiry forms
- Customer chat widget (when activated — feeds the Customer Engagement Specialist)

## Kind and Surfaces

- Kind: `business`
- Agent surface: `mcp` | `rest` | `webhook` — how Baxter converses with this service; `none` if it is not safe or useful to expose
- Flow surface: `rest` | `webhook` | `native` | `none` — how a flow calls it; `none` if it cannot be an endpoint
- Both are chosen once per connector and pinned in `deployment.yaml`; they are never re-decided per deployment.

## Connection

- Endpoint / MCP server: …
- Auth: values come from the deployment's declared secret source (`vault`, `native` or `hybrid`) — never stored here
- Scopes: least privilege, listed in the connector's `secrets.scope`

## Onboarding Questions (Stage 3)

| ID | Ask | Maps to |
|---|---|---|
| Q1 | "Where does your website live, and can you (or your web person) edit it?" *(lock-in — may be deferred)* | `connectors` entry |
| Q2 | "What does your enquiry form ask for, and where do submissions go today?" | Sync Topology — lead form destination |
| Q3 | "Should the customer chat widget be switched on from day one, or later?" *(gate-critical — activates the Customer Engagement Specialist)* | `channels.customer_chat`, `roles.customer_engagement` |

## Proficiency (after lock-in)

- [ ] Platform identified; connection method confirmed (webhook / REST / MCP) per §5 rule 5
- [ ] Lead-form event shape documented (what fields a submission carries)
- [ ] Chat-widget capability assessed (feeds `customer_engagement` role activation)
- [ ] Limitations and gaps recorded (e.g., no webhook support → polling workflow)

## Configuration Checklist

- [ ] Wire lead form to the API / Event Gateway
- [ ] Enable/disable customer chat widget (drives `customer_engagement` role activation)
- [ ] Configure spam/abuse handling

## Sync Topology (declared syncs)

- From: website → CRM — What: lead form creates job/contact

## Secrets (vault paths)

- `vault/connectors/website/…`

## Verification (smoke tests)

- [ ] Submit a test lead end-to-end
- [ ] Confirm event reaches the Workflow Engine

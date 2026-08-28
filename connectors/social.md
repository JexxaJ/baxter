# Connector Pack — Social Scheduler

## Version

1.0.0

## Purpose

Social media scheduling and publishing — public-facing by nature, always behind the human approval gate.

## Capabilities

- Draft storage and scheduling
- Publishing to connected channels (e.g., Facebook, Instagram, Google Business Profile)
- Post reporting

## Connection

- Standard: `mcp` | `rest` — **to be decided per deployment** (§5 rule 5)
- Endpoint / MCP server: …
- Auth: …

## Onboarding Questions (Stage 3)

| ID | Ask | Maps to |
|---|---|---|
| Q1 | "Which scheduling tool do you use (or want to use) for social publishing?" *(lock-in — may be deferred)* | `connectors` entry |
| Q2 | "Which social channels does the business run (e.g., Facebook, Instagram, Google Business Profile)?" | Configuration Checklist — connected channels |
| Q3 | "Do you have owner/admin access to all of them, or is something managed by someone else?" | Configuration Checklist — owner access |
| Q4 | "Anything you never want posted — topics, phrases, images?" | `business-context.md` → Persona and tone |

## Proficiency (after lock-in)

- [ ] Product and plan identified; MCP server endpoint confirmed (or documented REST API)
- [ ] Auth method understood (type and shape only — credentials stay in the vault, §7.4)
- [ ] Tool/resource surface enumerated from the live self-description (§8.3)
- [ ] Capabilities mapped to this deployment: scheduling, publishing, reporting per channel
- [ ] Approval-gate fit confirmed: drafts cannot publish without owner sign-off

## Configuration Checklist

- [ ] Create API credential (scoped, least privilege)
- [ ] Connect channels and verify owner access
- [ ] Confirm approval flow: Baxter drafts → owner approves → scheduler publishes

## Sync Topology (declared syncs)

- From: social → (reporting) — What: publish results back to Baxter memory

## Secrets (vault paths)

- `vault/connectors/social/…`

## Verification (smoke tests)

- [ ] Schedule a test post
- [ ] Confirm approval gate blocks an unapproved publish

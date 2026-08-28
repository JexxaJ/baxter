# Baxter — Business AI Assistant (Core Repository)

A repeatable business system: a **fixed, reusable core** plus **swappable connectors**. The core is identical in every deployment; connectors adapt the system to each business. This repository is what gets cloned when starting or deploying a new business.

## Repository layout

| Path | What it holds |
|---|---|
| `docs/` | The architecture framework (authoritative), harness notes (e.g., Hermes walkthrough), and the reference business plan (historical) |
| `core/constitution.md` | The Core Constitution — Baxter's instruction set, identical in every deployment (§8.1) |
| `core/bootstrap.md` | The bootstrap entry point — what a fresh harness loads to become Baxter (§8.1) |
| `core/onboarding-protocol.md` | The Onboarding Protocol — how Baxter gathers a business's needs from its owner (§8.6) |
| `core/business-questionnaire.md` | The core questionnaire — Stages 0, 1, 2, 4 of onboarding (§8.6) |
| `core/business-context.template.md` | The template Baxter completes for `business-context.md` during onboarding |
| `core/deployment.schema.json` | Formal schema for `deployment.yaml` — structure + semantic rules S1–S10 (§8.2) |
| `core/setup-protocol.md` | The Setup Protocol — how Baxter connects and configures what onboarding defined (§8.4) |
| `connectors/` | Connector Packs — what Baxter knows about each product: capabilities, connection, configuration, onboarding questions, sync topology (§8.1) |
| `deployments/` | Per-business deployment configs — created by onboarding, one directory per business (§8.2) |

## Starting a new deployment

Bring up Baxter on any agent harness (Hermes Cloud, VPS, local — same either way). Full walkthrough: `docs/harness-notes.md`.

1. **Deploy the harness** and configure provider + model settings. Create one messaging channel (e.g., a Telegram bot) and leave it open to the owner.
2. **Paste the bootstrap invitation** into the default profile:
   > Clone the Baxter repository `https://github.com/JexxaJ/baxter` into your workspace. When done, read `core/bootstrap.md` from it and follow it exactly.
3. **Baxter takes over** — the default profile becomes Baxter. It detects there is no Deployment Config and starts **onboarding** on that channel: business profile (scraped and confirmed), capability selection, connector lock-in and learning, working rules — ending in a committed, schema-validated config under `deployments/<business>/` (§8.6).
4. **Store secrets** in the password vault during setup — never in this repository, never in chat (§7.4). Onboarding records vault paths, never values.
5. **Setup** runs automatically after onboarding: connections, workflows, verification, documentation (§8.4, `core/setup-protocol.md`).

## Operating rules

- No secrets, credentials, or `.env` values are stored in this repository.
- All per-business configuration is versioned here (§7.2): changes flow repository → deploy, never the reverse.
- The core never changes per business — connectors are chosen per business.
- Releases are tagged (semantic versioning); every deployment pins `core_version` and each `pack_version` in its `deployment.yaml` (§7.2 rule 5).

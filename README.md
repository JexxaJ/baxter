# Baxter — Business AI Assistant

A portable business system: a **fixed blueprint** plus **swappable hosts, services and automation**. Install the skill on a skills-compatible agent — or paste the brief on a host that has no skill loader — and the agent *becomes* Baxter: it onboards a business, learns its tools, configures the connections, and operates the deployment. Per-business configuration lives in a git workspace the agent owns.

Baxter talks to every enrolled service directly (MCP first), and builds durable **flows** between them through an optional automation provider such as n8n or Zapier. Once a flow runs, the services talk to each other; Baxter checks results through the run ledger.

## Install and run

```bash
npx skills add JexxaJ/baxter
```

Then tell your agent: **"Become Baxter."** It will ask what you want to call it — that name becomes the deployment identity.

1. **Choose a host.** Read `docs/<harness>-requirements.md` first — it says which ports the platform supplies and how the blueprint is delivered (skill, brief or operator runbook).
2. **Install the blueprint** as a skill where supported, or paste `references/brief.md` into the agent's instructions.
3. **Say "become Baxter."** Baxter creates the workspace, runs onboarding, and commits a validated config.
4. **Choose your backup** and your secret source (external vault, host-native store, or hybrid). Values never enter chat or the repository.
5. **Run setup:** connections, optional automation provider, flows, verification, documentation.

## How work happens

| Mode | What it is | Example |
|---|---|---|
| **Conversation** | Baxter uses a service's tools | "Which jobs are late?", "Draft a reply" |
| **Flow** | A durable route between services, built by a provider | New enquiry → CRM record → tagged email list |
| **Gate** | A human approval before anything public, financial or destructive | Publish a post, send a campaign, refund |

A deployment may have **zero** automation providers and zero flows; providers are added when a flow needs one, not before.

## Repository layout

| Path | What it holds |
|---|---|
| `START.md` | Agent-facing entry point: clone, read the blueprint, run onboarding |
| `SKILL.md` | Bootstrap entry point — identity, state machine, workspace rules, guardrails |
| `references/brief.md` | Portable short form of the constitution, for hosts without a skill loader |
| `references/constitution.md` | The immutable behaviour of Baxter |
| `references/framework.md` | The architecture framework (authoritative) |
| `references/onboarding-protocol.md` | How Baxter gathers a business's needs |
| `references/business-questionnaire.md` | The core questionnaire (Stages 0–4) |
| `references/business-context.template.md` | The template for `business-context.md` |
| `references/deployment.schema.json` | Schema v2 + semantic rules S1–S20 |
| `references/setup-protocol.md` | How Baxter connects, builds flows and documents |
| `connectors/` | Connector Packs — business services, the automation-provider contract, n8n and Zapier |
| `docs/harness-requirements-template.md` | How a platform is assessed |
| `docs/<harness>-requirements.md` | Per-platform assessment and runbook (Hermes, Rakazo, Grok Bot, Nod, OpenMausBot) |
| `docs/harness-notes.md` | Platform index and assessment rules |
| `docs/architecture.html` | The blueprint diagram (interactive; spec in `docs/architecture.spec.json`) |
| `VERSION` | Blueprint release (semver); deployments pin it as `core_version` |

## Versioning

`VERSION` is the source of truth — currently **0.2.3**, which adds the owner-chosen assistant identity to the 0.2 line (schema v2, host adapters, the automation-provider contract and native secret sources). Released versions are tagged; every deployment pins `core_version`, each pack pins `pack_version`, and each host pins `adapter_version`. Updates are explicit (`npx skills update`), never silent.

## Operating rules

- No secrets, credentials or `.env` values in this repository or any config — references only.
- Per-business configuration lives in the agent's git **workspace**; changes flow repository → deploy, never the reverse.
- The blueprint never changes per business — hosts, services, providers and the assistant's name do.
- **Baxter is the blueprint codename.** The deployment's assistant name is chosen by the owner at onboarding and is the only identity used at runtime.
- One system of record for customer data; flows never generate content; judgment stays with Baxter.
- Every gated action is approved by a verified human, and the kill switch reaches flows.

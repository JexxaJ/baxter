# Baxter — Business AI Assistant

A repeatable business system: a **fixed, reusable core** plus **swappable connector packs**, distributed as an installable agent skill. Install the skill on any skills-compatible agent and the agent *becomes* Baxter — it onboards a business by questionnaire, learns its tools, configures the connectors, and runs the deployment. The core is identical in every deployment; only the per-business configuration differs, and that lives in a git-versioned workspace the agent owns. The standard stack is **agent harness + n8n co-located on one VPS** — n8n is the default Workflow Engine for the deterministic flows, with vendor-native integrations taking precedence wherever they exist (§7.6).

## Install and run

```bash
npx skills add JexxaJ/baxter
```

Then tell your agent: **"Become Baxter."** That's the whole setup.

1. **Install the skill** on your agent harness (Hermes Cloud, VPS, local — same either way). Configure provider + model settings and one messaging channel (e.g., a Telegram bot) as usual.
2. **Say "become Baxter"** (or "run Baxter bootstrap", or describe onboarding a business). Baxter detects there is no config workspace, creates one (`git init`), and starts **onboarding** on the current channel: business profile (scraped from your website and confirmed with you), capability selection, connector lock-in and learning, working rules — ending in a committed, schema-validated config (§8.6).
3. **Choose your backup**: Baxter offers to push the workspace to a private git repository you control; if you decline, keep your agent-level backups current — the config is your deployment (§7.2).
4. **Store secrets in your password vault** when setup asks — never in chat, never in any repository (§7.4). Onboarding and setup reference vault paths only.
5. **Setup** follows automatically: connections, workflows, verification, documentation (§8.4). Every irreversible step sits behind an approval gate addressed to you.

A worked Hermes walkthrough is in `docs/harness-notes.md`.

## Repository layout

This repository **is** the skill (`npx skills add JexxaJ/baxter`):

| Path | What it holds |
|---|---|
| `SKILL.md` | The bootstrap entry point — identity, state machine, workspace rules, guardrails (§8.1) |
| `references/constitution.md` | The Core Constitution — Baxter's instruction set, identical in every deployment (§8.1) |
| `references/onboarding-protocol.md` | The Onboarding Protocol — how Baxter gathers a business's needs from its owner (§8.6) |
| `references/business-questionnaire.md` | The core questionnaire — Stages 0, 1, 2, 4 of onboarding (§8.6) |
| `references/business-context.template.md` | The template Baxter completes for `business-context.md` |
| `references/deployment.schema.json` | Formal schema for `deployment.yaml` — structure + semantic rules S1–S11 (§8.2) |
| `references/setup-protocol.md` | The Setup Protocol — how Baxter connects and configures what onboarding defined (§8.4) |
| `references/framework.md` | The architecture framework (authoritative) |
| `connectors/` | Connector Packs — what Baxter knows about each product: capabilities, onboarding questions, proficiency checklist, configuration, sync topology (§8.1) |
| `docs/harness-notes.md` | Operator notes — the Hermes walkthrough |

## Operating rules

- No secrets, credentials, or `.env` values in this repository or any config — vault paths only (§7.4).
- Per-business configuration lives in the agent's git-versioned **config workspace**; changes flow repository → deploy, never the reverse (§7.2).
- The core never changes per business — connectors are chosen per business.
- Core updates are explicit skill updates (semantic version tags); every deployment pins its `core_version` (§7.2 rule 5).

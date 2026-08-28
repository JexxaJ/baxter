# Harness Notes — Bringing Up Baxter on an Agent Harness

*Status: reference notes, not core architecture. The framework (§8) is harness-agnostic; these notes walk through one concrete harness — **Hermes** (Cloud or self-hosted VPS, same behaviour) — end to end. If you use a different harness, map the same four phases onto its equivalents.*

## 1. The Four Phases

| Phase | Who | What happens |
|---|---|---|
| **1. Harness** | Operator | Deploy Hermes (Cloud, VPS, or local server). Configure provider + model settings. No Baxter-specific setup yet. |
| **2. Bootstrap** | Operator → default profile | Clone the repo, load the bootstrap instruction (below). The **default profile becomes Baxter**. |
| **3. Onboard** | Baxter + owner | Baxter detects no Deployment Config, enters onboarding mode, runs the questionnaire on this channel. Ends with a committed `deployments/<business>/` config. |
| **4. Setup** | Baxter + owner | Baxter detects no setup record, runs the Setup Protocol (§8.4): credentials into the vault, connections, workflows, verification, documentation. |

## 2. Phase 1–2 — The bootstrap invitation

In the **default profile** (fresh, after provider/model setup), paste:

```text
Clone the Baxter repository https://github.com/JexxaJ/baxter into your
workspace. When done, read core/bootstrap.md from it and follow it exactly.
```

That is the entire operator-side configuration. Everything else — identity, constitution, state detection, protocols — is retrieved from the repo by the agent itself (§7.3 retrieve-don't-stuff). Re-running the same line on a wiped harness resumes from repo state.

**Notes:**
- The default profile *is* the bootstrap profile *is* the future Coordinator. There is no separate "Baxter profile" to create — the default profile becomes Baxter by reading `core/bootstrap.md`.
- Constitution is loaded **by reference** (`core/constitution.md` from the cloned repo), not inlined into a prompt — this keeps the paste-able instruction stable while the core versions under git (§7.2 rule 5).

## 3. Phase 3 — Onboarding on the channel

The conversation that carries the bootstrap (Telegram bot, or the Hermes UI itself) is the **Messaging Channel** during onboarding. Baxter:

1. Determines state: no `deployments/<business>/deployment.yaml` → onboarding mode.
2. Runs Stage 0 on that channel: introduction + owner identity verification.
3. Proceeds through the questionnaire — including scraping-and-confirming the business profile from a supplied website, capability selection, connector lock-in & learning (no credentials), and working rules.
4. Writes `deployments/<business>/deployment.yaml` + `business-context.md`, validates against `core/deployment.schema.json` (S1–S10), presents the full config, and commits after owner approval.

## 4. Phase 4 — Setup, credentials, and profiles

Setup consumes the committed config (`core/setup-protocol.md`). Hermes-specific notes:

- **Credentials:** humans add them to the vault at the paths the config expects (§7.4); Baxter verifies existence only. Hermes never sees values.
- **Roles → profiles:** the default profile remains the **Coordinator** permanently. Specialist roles (`content`, `operations`, `customer_engagement`) are realized as **additional Hermes profiles created at Apply**, each derived from the same core artifacts:
  - system prompt = the role's scope from `deployment.yaml` (`roles.<name>.scopes`) + the Constitution's role rules (§4.1) + `business-context.md` persona section;
  - tool access limited to the scoped connectors' MCP servers;
  - memory namespace per role (§7.5).
- **If profile creation can't be automated** in a given harness version, Apply instead produces a **manual checklist** in the setup record (exact prompt text, scopes, tool list per profile) and the operator creates the profiles; Verify then confirms each profile holds only its scoped tools.
- Until a specialist profile exists, the Coordinator performs that role's work itself — a default deployment runs Baxter alone (§4.1 rule 6: design the seams now, activate later).

## 5. Recovery / re-deployment

Wipe the harness, deploy fresh, paste the bootstrap invitation again. State detection (bootstrap §2) resumes from the repository: committed config present → skips onboarding; setup record missing → re-runs Setup from the repository (apply from repo + verify, §7.1 rule 4). Memory lives in the core Memory Service, not the harness (§7.5), so nothing conversational is lost.

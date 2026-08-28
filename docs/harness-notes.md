# Harness Notes — Bringing Up Baxter on an Agent Harness

*Status: operator reference, not core architecture. The framework (§8) is harness-agnostic; these notes walk through one concrete harness — **Hermes** (Cloud or self-hosted VPS, same behaviour) — end to end. If you use a different harness, map the same four phases onto its equivalents.*

## 1. The Four Phases

| Phase | Who | What happens |
|---|---|---|
| **1. Harness** | Operator | Deploy Hermes (Cloud, VPS, or local server). Configure provider + model settings. No Baxter-specific setup yet. |
| **2. Install** | Operator | Install the skill once: `npx skills add JexxaJ/baxter`. |
| **3. Onboard** | Baxter + owner | Say "become Baxter". Baxter detects no config workspace, creates one, enters onboarding mode, runs the questionnaire on this channel. Ends with a committed config in the workspace. |
| **4. Setup** | Baxter + owner | Baxter detects no setup record, runs the Setup Protocol (§8.4): credentials into the vault, connections, workflows, verification, documentation. |

## 2. Hermes test checklist — onboarding-only dry run

Onboarding deliberately requires **no credentials, no vault, no connectors** — that's what makes it a clean first test. Full test:

1. In Hermes Cloud, open the default profile (provider/model already configured).
2. Install the skill — via Hermes' skill-install mechanism or by running `npx skills add JexxaJ/baxter` in the agent's environment. Confirm `baxter` appears in the profile's available skills.
3. Message the profile (Hermes UI or the Telegram bot wired to it): **"Become Baxter."**
4. Expect, in order:
   - an introduction + identity verification (Stage 0) — it treats this human as the owner;
   - a request for the business's **website or a descriptive document**, followed by extracted findings to confirm;
   - the plain-language capability menu (customer records, invoicing, email marketing, social, website, customer chat);
   - per selected capability, a "which tool?" question — answer, or defer ("not sure yet");
   - working rules: who approves what, sync confirmations, and the **backup question** (private remote vs local-only + warning);
   - a full config draft + summary, ending in an approval gate.
5. Approve. Baxter commits `baxter-workspace/deployments/<business>/` (deployment.yaml, business-context.md, connector-proficiency/) to the local git workspace.
6. Verify without leaving the channel: ask Baxter to `git log` the workspace, or check the private remote if you provided one.

**Stopping here is a valid test outcome** — onboarding → committed config is the complete understanding phase. Setup (credentials, connections) is a separate session with the vault reachable.

## 3. Notes for Hermes specifically

- **The default profile *is* Baxter.** There is no separate "Baxter profile" to create — the skill's SKILL.md makes whatever profile loads it into Baxter, and that profile remains the permanent **Coordinator** (§4.1).
- **Constitution is loaded by reference** from the installed skill's files (`references/constitution.md`), not inlined into a prompt — the skill stays small and updates flow through `npx skills update` (§7.2 rule 5).
- **Roles → profiles:** specialist roles (`content`, `operations`, `customer_engagement`) are realized as **additional Hermes profiles created at Apply** (§8.4 step 5), each derived from the same core artifacts: the role's scopes from `deployment.yaml` (`roles.<name>.scopes`), the Constitution's role rules (§4.1), and the business-context persona. Tool access limited to the scoped connectors' MCP servers; memory namespace per role (§7.5).
- **If profile creation can't be automated** in a given Hermes version, Apply instead produces a **manual checklist** in the setup record (exact prompt text, scopes, tool list per profile) and the operator creates the profiles; Verify then confirms each profile holds only its scoped tools.
- Until a specialist profile exists, the Coordinator performs that role's work itself — a default deployment runs Baxter alone (§4.1 rule 6: design the seams now, activate later).
- **Workspace location:** Baxter creates `baxter-workspace/` in its working directory. On Hermes Cloud, confirm persistence across restarts (or attach a volume); if the workspace could be wiped, use the private-remote backup so the config survives.

## 4. Recovery / re-deployment

Wipe the harness, reinstall the skill, say "become Baxter" again. State detection (SKILL.md §2) resumes from workspace + remote: committed config present → skips onboarding; setup record missing → re-runs Setup from the repository (apply from repo + verify, §7.1 rule 4). Memory lives in the core Memory Service, not the harness (§7.5), so nothing conversational is lost.

# Harness Notes — Bringing Up Baxter on an Agent Harness

*Status: operator reference, not core architecture. The framework (§8) is harness-agnostic; these notes walk through one concrete harness — **Hermes** — end to end. If you use a different harness, map the same phases onto its equivalents.*

## 0. Hosting requirement — VPS, not Hermes Cloud

The standard stack is **Hermes + n8n co-located on one VPS** (the Workflow Engine, §7.6). **Hermes Cloud does not support Docker sidecars and cannot host n8n** — it can only pair with an `external` n8n on a separate host. For the standard, single-instance deployment: **deploy Hermes on a VPS** (self-hosted or a one-click Hermes WebUI Docker template), then add n8n to the same host.

```text
one VPS
├── Hermes (agent harness) ──── talks to n8n via the official n8n MCP server
├── n8n (Docker) — the Workflow Engine (§7.6) — internal deployment
└── baxter-workspace/ — agent-local config repository (§7.2)
```

The `external` option (n8n on a separate VPS) exists for isolation or scale — same MCP wiring, different endpoint.

## 1. The Phases

| Phase | Who | What happens |
|---|---|---|
| **1. Harness** | Operator | Deploy Hermes on a VPS + n8n in Docker on the same host. Configure provider + model settings. Create the n8n MCP bearer token; store it in the vault at `vault/connectors/n8n`. |
| **2. Install** | Operator | Install the skill once: `npx skills add JexxaJ/baxter`. Point the harness's MCP configuration at the n8n MCP server (URL + token) so Baxter can drive it. |
| **3. Onboard** | Baxter + owner | Say "become Baxter". Baxter detects no config workspace, creates one, enters onboarding mode, runs the questionnaire on this channel. Ends with a committed config in the workspace. |
| **4. Setup** | Baxter + owner | Baxter detects no setup record, runs the Setup Protocol (§8.4): verifies the engine is reachable, builds only the workflows that have no native path (§5 rule 6), verifies everything, documents. |

**n8n configuration is Baxter's job, not the operator's.** All flow-building happens through Baxter's setup protocol using the n8n MCP tools — the operator only stands up the service and its token.

## 2. Hermes test checklist — onboarding-only dry run

Onboarding deliberately requires **no credentials, no vault, no connectors** — that's what makes it a clean first test. Full test:

1. On the **VPS**, open the default profile (provider/model already configured).
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
- **Workspace location:** Baxter creates `baxter-workspace/` in its working directory. Confirm persistence across restarts (or attach a volume); if the workspace could be wiped, use the private-remote backup so the config survives.
- **n8n wiring:** add the n8n MCP server to the harness's MCP configuration — URL `http://localhost:5678/mcp-server/http` (internal default) and `Authorization: Bearer <token>`; the token itself lives in the vault (`vault/connectors/n8n`). Baxter enumerates the tool surface itself at setup (n8n pack, Proficiency).

## 4. Recovery / re-deployment

Wipe the harness, reinstall the skill, say "become Baxter" again. State detection (SKILL.md §2) resumes from workspace + remote: committed config present → skips onboarding; setup record missing → re-runs Setup from the repository (apply from repo + verify, §7.1 rule 4). Memory lives in the core Memory Service, not the harness (§7.5), so nothing conversational is lost.

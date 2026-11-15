# Hermes — Harness Requirements

*Assessment for the Baxter blueprint v0.2.2. Paper assessment from product documentation and existing operator notes; the conformance test in §12 is run at first install and its results recorded here.*

## 1. Identity

- Product: Hermes Agent (Nous Research)
- Hosting: self-hosted (VPS or local); hosted variant exists
- Licence: open source
- Docs: hermes-agent.nousresearch.com
- Assessed: blueprint 0.2.2

## 2. Topology

- A **profile** is a full agent: its own model, memory, skills, connectors, sessions and gateway bot.
- State lives per profile on the host filesystem.
- One process can run several profiles with separate Telegram bots.

## 3. Ports

| Port | Grade | Mechanism | Notes / fallback |
|---|---|---|---|
| Channel | native | Telegram bot, Web UI | Staff can be separate users or topics |
| Runtime | native | Agent loop with tools | |
| Instructions | native | Skills directory loaded per profile | Skill mode works |
| Roles | partial | Separate profiles per role | Profiles must be created by the operator; scopes are per profile, not enforced per connector |
| Tools | native | MCP servers, shell, HTTP | MCP per profile |
| Memory | partial | Profile memory | Host-native: not portable, not role-scoped by a service; declare degraded |
| Scheduling | partial | Cron / scheduled prompts | Verify on the installed version |
| Approvals | partial | Chat confirmation | No blocking approval primitive; gated flows rely on n8n pause |
| Files | native | Host filesystem | Git workspace works |
| Secrets | partial | Profile env/config | External vault needs a plugin/service; native by default |
| Audit | degraded | Logs only | Core run ledger compensates |

**Verdict:** full host. `delivery: skill`. Declare `memory` (if host-native) and `audit` as degraded.

## 4. Delivery mode

- **skill** — install `SKILL.md` + `references/` into the profile's skills directory.
- Bootstrap: install skill → set `docs/hermes-requirements.md` in the workspace → message "Become my assistant" (the owner names it at Stage 0).
- Keep the skill read-only; configuration lives in `baxter-workspace/`.

## 5. Secrets

- Default: `native` (profile environment/config).
- `hybrid` possible: vault reachable through an MCP secrets service for core-owned keys; profile env for connector tokens.
- Per-role scoping: only as good as profile isolation. Record `tool-scoping` as degraded if profiles share credentials.

## 6. Services and providers

- MCP servers per profile; HTTP tools available.
- Native automation: scheduled prompts/cron; **not** a general flow engine.
- Recommended automation provider: external n8n via its MCP server (see `connectors/n8n.md`), not n8n co-located unless the host supports Docker sidecars.
- Hermes Cloud cannot host Docker sidecars: use an external provider or host-native routines.

## 7. Roles mapping

| Blueprint role | Construct | Isolation | Notes |
|---|---|---|---|
| Coordinator | default profile | native | Own memory/model/skills |
| Content | second profile, content scopes | partial | Same host, separate credentials |
| Operations | third profile, CRM/accounting scopes | partial | |
| Customer Engagement | profile with customer channel | partial | Only if a customer chat channel exists |

Profile creation is an Apply step with a manual checklist if it cannot be automated.

## 8. Workspace and persistence

- Filesystem: yes. `baxter-workspace/` in the agent's working directory.
- Git: yes. Private remote optional; agent-level backup otherwise.
- Persistence across container restarts depends on the mounted volume — confirm at install, or use the private remote.

## 9. Approvals and kill switch

- Approvals: chat confirmation, recorded by the assistant; no external enforcement hook.
- A running flow is stopped only through the provider (n8n API) or by cancelling the prompt.
- Kill switch: stop accepting messages, stop memory writes, pause provider runs where the API allows. Record what cannot be stopped.

## 10. Constraints

- No Docker sidecars on the hosted variant.
- Profiles are operator-managed; role isolation is procedural, not credential-enforced.
- No native run ledger; the workspace ledger is authoritative.

## 11. Config mapping

```yaml
harness:
  platform: hermes
  delivery: skill
  adapter_version: 1.0.0
  ports: { channel: native, runtime: native, instructions: native,
           roles: partial, tools: native, memory: partial,
           scheduling: partial, approvals: partial, files: native,
           secrets: partial, audit: degraded }
  degraded: [memory, audit]
secrets:
  source: native
  native_store: hermes-profile-env
```

## 12. Conformance test

| Check | Result |
|---|---|
| Creates workspace and starts Stage 0 | to verify |
| Refuses a pasted credential | to verify |
| Refuses a service added in conversation | to verify |
| Public action pauses | to verify |
| Provisional flow appears in ledger | to verify |
| Kill switch stops agent work | to verify |

## 13. Known gaps

- No native approval-enforcement point; gated flows depend on the provider's pause.
- Memory portability requires a core memory service; otherwise declare `host-native`.
- Conformance test pending first install.

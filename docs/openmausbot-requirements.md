# OpenMausBot — Harness Requirements

*Assessment for the Baxter blueprint v0.2.2. Desk assessment from the repository (`milind-soni/OpenMausBot`, commit `8471a36`, Apache-2.0); run the conformance test at first install and record the result here.*

## 1. Identity

- Product: OpenMausBot (formerly OpenGrokBot; also "MausBot"), open source, Apache-2.0
- Hosting: local desktop (macOS/Windows/Linux), self-hosted server, or team cloud
- Engines: Claude Code / Codex / Grok CLIs, other CLIs, or API engines; **agent tools require a CLI engine** — API-key connections are chat-only
- State: `~/.openmausbot` (SQLite, transcripts, keys, workspaces)
- Assessed: blueprint v0.2.2

## 2. Topology

- One **bot per chat** — a real agent with its own personality (`SOUL.md`), model, memory, computer and connected apps.
- A small harness server on `127.0.0.1` owns every agent process; clients are the desktop app, browser, and paired phones.
- Teams add a Chief of Staff, channels (Work/Personal/project), shared instructions, departments and guests.
- Work between bots uses a job runtime (`Assignment`) with approval levels, resumption and an audit trail in SQLite.
- Long-running: the server must stay up; closing the browser does not stop it, stopping the server stops all bots.

## 3. Ports

| Port | Grade | Mechanism | Notes / fallback |
|---|---|---|---|
| Channel | native | In-app chat (desktop/web/phone), team channels | No external IM bridge; staff use the app |
| Runtime | native | Harness server + agent CLIs/API engines | CLI engine required for tools |
| Instructions | partial | `SOUL.md` standing instructions; skills are `SKILL.md` **text only** | No supporting files in a skill; use the brief in `SOUL.md` + a repo checkout on the computer |
| Roles | partial | One bot per role, own model/apps/memory; teams, Chief, rooms | Isolation is per bot; credential scoping is per connected app, not a vault policy |
| Tools | native | Custom MCP servers (stdio/HTTP), Composio 500+ apps, computer/browser | Strongest tool plane of the assessed hosts |
| Memory | partial | `MEMORY.md` (first 200 lines / 24 KB per turn) + topic files + daily logs | File-based, auditable, but host-native and size-bounded; declare degraded |
| Scheduling | native | Routines (cron, webhooks), proactive checks | Routines run **agent turns** — they are not deterministic flows |
| Approvals | native | Ask / Auto-accept edits / Approve for me / Full access; approval cards; Sending-on-your-behalf | Strongest approval model of the four; default is Full access, change it |
| Files | native | Bot workspace `~/.openmausbot/workspaces/<botId>/`, project folders, cloud/Local VM computers | Git works |
| Secrets | partial | Local `~/.openmausbot` store; provider CLI logins; Composio OAuth; MCP headers in plaintext | No external vault; per-bot app scoping is the control |
| Audit | partial | SQLite job runtime, transcripts, approval journal | No structured export known; the workspace ledger complements it |

**Verdict:** full host with `delivery: brief`. Declare `instructions`, `memory` and `secrets` degraded. Strongest approvals; weakest portable memory.

## 4. Delivery mode

- **brief** — put `references/brief.md` in the bot's `SOUL.md` / standing instructions, and keep this repo on the bot's computer so it can read the protocols by path.
- Skills: only `SKILL.md` text travels; `references/` and `connectors/` do not. A skill import is therefore not a substitute for the files — use the filesystem.
- Bootstrap: create the bot **named by the owner** (the platform requires a name at creation), put the brief in `SOUL.md`, set the project folder to the repo/workspace, then paste the prompt in §11.

## 5. Secrets

- **native only.** Keys, MCP header values and provider credentials live under `~/.openmausbot`; the docs state MCP header values are plaintext.
- Provider logins come from the installed CLIs — no new accounts, but the credential is shared with anything else using that CLI login.
- Composio holds app OAuth; per-bot app connections are the scoping mechanism.
- There is no external vault. Declare `secrets` degraded and compensate: one bot per role, only that role's apps connected, approval level Ask, and `Sending on your behalf: Ask every time`.
- Never paste credentials into chat; use the platform's own card or settings flow.

## 6. Services and providers

- MCP: add remote (Streamable HTTP/SSE) or local stdio servers; tools appear to compatible bots on their next task.
- Apps: Composio marketplace (Gmail, Slack, GitHub, Notion, …) — a **broker** connector in blueprint terms.
- Native automation: routines (cron/webhook) and proactive checks run the bot; they are judgment runs, not deterministic flows.
- For a **flow** the blueprint expects a deterministic provider: connect an external n8n via MCP/API (`connectors/n8n.md`), or use vendor-native integrations. Do not record a routine as a `via: n8n`-equivalent flow; it is an agent turn.
- OpenMausBot also exposes its own stdio MCP server to external clients — useful for driving it from another agent, not for flows.

## 7. Roles mapping

| Blueprint role | Construct | Isolation | Notes |
|---|---|---|---|
| Coordinator | the named assistant bot | native | Own `SOUL.md`, model, memory, apps |
| Content | a second bot | partial | Only content apps; Ask level |
| Operations | a third bot | partial | CRM/accounting apps only |
| Customer Engagement | a separate bot + channel | partial | Customer-facing; never holds finance apps |

Use one bot per role, never a bot per task. Task work belongs in assignments/rooms with no new credentials.

## 8. Workspace and persistence

- Each bot has a private workspace folder; a project folder can point at a git checkout.
- Recommended: `baxter-workspace/` inside the coordinator's workspace, committed; private remote optional.
- State survives server restarts in `~/.openmausbot`; a deleted bot deletes its schedules and webhooks.
- Team sharing strips skills' supporting files, memory and credentials — the workspace remains the source of truth.

## 9. Approvals and kill switch

- Approval levels are per bot/turn: **Ask**, Auto-accept edits, Approve for me, Full access. Default is Full access — set **Ask** for the coordinator and **Ask every time** for sending on your behalf before onboarding.
- Approval cards pause a turn and resume it; `needs_input` is part of the job runtime.
- Kill switch: stop the bot / all bots and pause routines. In-flight computer actions may finish; record that.
- The local API counts local calls as the owner — never expose the loopback port.

## 10. Constraints

- CLI engine required for tool use; an API-key engine is chat-only.
- Skill packages carry `SKILL.md` text only; no scripts or reference files.
- Memory is capped at 200 lines / 24 KB per turn.
- Routines are agent turns; they cannot replace a deterministic flow provider.
- Full access is the default; that is the opposite of the blueprint's gate posture.
- Hosted team defaults differ; verify operator policy.

## 11. Config mapping

```yaml
harness:
  platform: openmausbot
  delivery: brief
  adapter_version: 1.0.0
  ports: { channel: native, runtime: native, instructions: partial,
           roles: partial, tools: native, memory: partial,
           scheduling: native, approvals: native, files: native,
           secrets: partial, audit: partial }
  degraded: [instructions, memory, secrets]
secrets:
  source: native
  native_store: openmausbot-local
```

## 12. Conformance test

| Check | Result |
|---|---|
| Owner names the bot; it confirms the name and creates the workspace | to verify |
| Refuses a pasted credential | to verify |
| Refuses a service added in conversation | to verify |
| Public send pauses (Sending on your behalf = Ask every time) | to verify |
| Provisional flow recorded in `runs/ledger.md` | to verify |
| Kill switch stops the bot and pauses routines | to verify |

## 13. Known gaps

- No external vault; native secrets only.
- Skills cannot carry the blueprint's references; the filesystem is the carrier.
- No deterministic flow engine; an external n8n is required for true flows.
- Conformance test pending first install.
- The default Full access level is a configuration hazard for a business deployment.

# Rakazo — Harness Requirements

*Assessment for the Baxter blueprint v0.2.3. Paper assessment from product documentation; verify the conformance test at first install. Rakazo is a bot platform with its own computer, routines, memory and app brokers, and imports agents from other platforms.*

## 1. Identity

- Product: Rakazo (rakazo.com), open source, Apache-2.0
- Hosting: self-hosted via Docker (Postgres + worker + app) or hosted cloud (beta)
- Docs: rakazo.com/docs, github.com/elie222/rakazo
- Assessed: blueprint 0.2.3

## 2. Topology

- A **bot** is a persistent teammate: its own conversations, memory, routines, apps and (optionally) its own computer.
- Bots can share a team computer, delegate to each other, and work in group chats.
- State: Postgres + bot home folders + browser profiles + audit log on the host.
- Clients: web, desktop, iOS, Android — one API.
- Mature enough to import agents from OpenClaw/Hermes, which makes it a realistic migration target.

## 3. Ports

| Port | Grade | Mechanism | Notes / fallback |
|---|---|---|---|
| Channel | native | Slack, WhatsApp, Telegram, iMessage/SMS, in-app | Self-host required for some channels |
| Runtime | native | Per-bot agent loop with computer and tools | |
| Instructions | native | Skills install from a git URL with `SKILL.md` + references beside it; per-bot instructions | Skills are task-matched and script-free; the Coordinator needs its condensed brief in bot instructions (20k-char cap), with the skill loaded by reference |
| Roles | partial | Separate bots + delegation | No per-connector credential scoping between bots by default |
| Tools | native | MCP, OpenAPI, Composio/Pipedream apps | Strongest connector reach of the four hosts |
| Memory | native | Per-bot memory + Postgres | Host-native: not portable; declare degraded |
| Scheduling | native | Routines: schedule, webhook, email, GitHub event | Also a candidate automation provider |
| Approvals | native | Product-level approvals ("ask before acting") | Verify that a routine can pause and resume |
| Files | native | Persistent Debian home with git, node, python3, sqlite3; shared computer | Filesystem exists on the host |
| Secrets | native | Encrypted connector credentials on the server; env for server secrets | No first-class external vault; `hybrid` only for core keys |
| Audit | partial | Audit log present | Export/retention to verify |

**Channel options:** native web/desktop/mobile app; external Slack, WhatsApp, Telegram, iMessage/SMS — some channels are self-host only.

**Verdict:** full host with `delivery: skill` (brief fallback). Declare `memory` degraded for portability, `roles` partial, and name the bot-instruction limits.

## 4. Delivery mode

- **skill** (recommended) — install the skill from the repo so `SKILL.md` and its references sit beside it, and paste the condensed `references/brief.md` into the assistant bot's instructions (20k-char cap) so it is always in character.
- **brief** fallback when the skill cannot be installed: paste the brief and hand over documents stage by stage.
- **operator** alternative when bot creation and app connection are to be done by a person with the runbook.
- Bootstrap: install the skill; create the assistant bot with the brief in its instructions (the owner names it at first contact; record the name) → connect apps → ask it to begin onboarding.

## 5. Secrets

- Default: `native` — connector OAuth and credentials are encrypted on the server; model keys in env per bot.
- `hybrid` is the recommended declaration: vault (or host env) for core/root secrets, native for connector grants.
- Scoping: per bot/connection, not per blueprint role. Declare `tool-scoping` degraded and compensate by giving each bot only its own apps.

## 6. Services and providers

- **Broker** connectors are first-class here: Composio/Pipedream front 1,400+ apps, so a single connector entry may represent a broker plus named apps.
- MCP servers and OpenAPI tools are supported directly.
- **Routines can be the automation provider** (`provider: host-routines`) for simple flows. Declare capabilities honestly: pause/resume in routines must be verified; run history is visible in the product.
- External n8n remains an option when flow logic outgrows routines.

## 7. Roles mapping

| Blueprint role | Construct | Isolation | Notes |
|---|---|---|---|
| Coordinator | assistant bot (owner-named) | partial | Own memory, apps |
| Content | "Content" bot | partial | Only content apps connected |
| Operations | "Operations" bot | partial | CRM/accounting apps only |
| Customer Engagement | "Customer chat" bot | partial | Customer channel only |

One bot per role; never a bot per task. Task bots are ephemeral subagents with no credentials.

## 8. Workspace and persistence

- Files: each bot has a home folder; a shared computer is possible.
- Git: available on the host; keep `baxter-workspace/` as a repo and let the assistant bot write to it.
- Backups: Postgres + bot files belong to the deployment; the workspace still pushes to a private remote when chosen.
- The platform's own settings are **not** the source of truth; the workspace is.

## 9. Approvals and kill switch

- Product approvals can gate actions per bot. Confirm whether an approval can pause a *routine* mid-run and resume it; if not, gated flows must stay with the assistant or move to a provider that can pause.
- Kill switch: disable the bot / stop routines; record that in-flight computer sessions may continue until cancelled.

## 10. Constraints

- Skills install but are task-matched and cannot run scripts; the Coordinator's brief must also live in bot instructions.
- No Docker or sudo on a bot computer: n8n must be external (or use routines).
- Hosted cloud is beta; some channels and local MCP are self-host only.
- Memory and audit are host-owned; portability requires the workspace + exported summaries.
- Per-bot credential scoping is weaker than a vault-backed service.

## 11. Config mapping

```yaml
harness:
  platform: rakazo
  delivery: skill
  adapter_version: 1.0.0
  ports: { channel: native, runtime: native, instructions: partial,
           roles: partial, tools: native, memory: native,
           scheduling: native, approvals: native, files: native,
           secrets: native, audit: partial }
  degraded: [memory, roles]
secrets:
  source: hybrid
  vault_path: vault/
  native_store: rakazo-connectors
```

## 12. Conformance test

| Check | Result |
|---|---|
| Creates workspace and starts Stage 0 from the brief | to verify |
| Refuses a pasted credential | to verify |
| Refuses a service added in conversation | to verify |
| Public action pauses | to verify |
| Provisional flow recorded in ledger | to verify |
| Kill switch stops the bot and its routines | to verify |

## 13. Known gaps

- Brief delivery means the blueprint content is a copy in bot instructions; keep it version-stamped and re-paste on blueprint upgrades.
- Routine pause semantics unverified — a gated flow must not rely on them until proven.
- Workspace/git discipline is operator-enforced.

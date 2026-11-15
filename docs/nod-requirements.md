# Nod — Harness Requirements

*Assessment for the Baxter blueprint v0.2.3. Paper assessment from Nod's documentation (n8n Labs research preview); verify at first install. Nod is the closest of the four to the blueprint's own assumptions: a filesystem workspace, a skills reader, MCP tools, native approvals and schedules.*

## 1. Identity

- Product: Nod (n8n Labs), open source, research preview
- Hosting: Mac, Windows, Linux, or a server; your own model provider
- Docs: nod.n8n.io/docs, github.com/n8n-io/nod
- Assessed: blueprint 0.2.3

## 2. Topology

- A **personal agent** with a workspace container and its own browser; one owner, multiple threads and saved tasks.
- Runtime: Pi sessions; one turn at a time per thread, subagents in parallel.
- State: workspace files, thread transcripts, memory, learned skills, task sessions, Vault.
- Channels: web, Telegram, WhatsApp.
- Designed as a single-owner agent; not multi-tenant.

## 3. Ports

| Port | Grade | Mechanism | Notes / fallback |
|---|---|---|---|
| Channel | native | Web, Telegram, WhatsApp | Business staff = separate Nod instances or a shared chat |
| Runtime | native | Pi sessions with turn guard | |
| Instructions | partial | `skills` tool reads guides and learned skills | A SKILL.md must be adapted to a guide; `brief` is the fallback |
| Roles | partial | Threads, tasks, subagents | No persistent specialist profiles; role isolation is by instance or prompt |
| Tools | native | Apps, MCP server tools, codemode, browser | Deferred tool loading keeps context small |
| Memory | native | Nod memory + learned skills | Host-native; declare degraded for portability |
| Scheduling | native | Schedules, proactive checks, events | |
| Approvals | native | Trust checks, `ask_user`, action review | Confirm a scheduled flow can wait for approval |
| Files | native | Workspace container, bash, git | Best-in-class for the blueprint |
| Secrets | native | Vault, private entry, sign-in handoff | No external vault option |
| Audit | partial | Turn events, model usage, receipts | Verify export/retention |

**Channel options:** native web/desktop app; external Telegram and WhatsApp. No Slack or SMS bridge.

**Verdict:** full host, `delivery: skill` (guide) with `brief` fallback. Declare `memory` degraded for portability and `roles` partial.

## 4. Delivery mode

- **skill** — convert `SKILL.md` + `references/` into a Nod guide/skill entry the `skills` tool can read, or paste `references/brief.md` if the guide route is awkward.
- Bootstrap: new agent with the guide → workspace path → begin onboarding; the owner names it at Stage 0.
- Nod already reads core guides on demand, which matches the blueprint's "retrieve, don't stuff" rule.

## 5. Secrets

- `native` — Nod Vault holds logins; app actions pass a trust check; private entry keeps values out of the model.
- `hybrid` possible by keeping core/root secrets in the operator's vault and giving Nod only scoped app connections.
- Scoping: per app connection and per instance; no per-role service. Record `tool-scoping` as degraded if specialists share one instance.

## 6. Services and providers

- Apps via Composio-style connections, MCP servers, OpenAPI, plus the computer/browser for services with no connector.
- **n8n integration:** Nod connects to n8n to build, test and run workflows — a natural pairing of host and automation provider.
- Host schedules can serve as `provider: host-routines` for simple flows; n8n is the better choice when a flow needs structure, pause or run history.

## 7. Roles mapping

| Blueprint role | Construct | Isolation | Notes |
|---|---|---|---|
| Coordinator | the Nod agent | native | Owner-facing |
| Content | second Nod instance | partial | Separate model/memory/apps |
| Operations | second/third instance | partial | |
| Customer Engagement | separate instance + channel | partial | Customer chat needs its own channel and sandbox |

Because Nod is single-owner, a multi-role deployment means one instance per role on the same host, each with its own workspace and Vault scope. Record the operational cost.

## 8. Workspace and persistence

- Full workspace container with `bash`, `git`, files.
- `baxter-workspace/` is a real directory; private remote backup works.
- Persistence: verify the workspace volume survives restarts and upgrades.

## 9. Approvals and kill switch

- Every tool call passes a turn guard; app actions and payments pass a trust check and can ask the owner.
- Scheduled runs and tasks: verify they wait for approval rather than acting after a timeout.
- Kill switch: stop tasks/all background work; the agent refuses new calls. In-flight browser steps need checking.

## 10. Constraints

- Research preview: APIs and behaviour may change; re-assess at each release.
- Single owner; no native teams or per-user roles.
- No external vault integration; native Vault only.
- Guide format differs from a skill; conversion is required.

## 11. Config mapping

```yaml
harness:
  platform: nod
  delivery: skill
  adapter_version: 1.0.0
  ports: { channel: native, runtime: native, instructions: partial,
           roles: partial, tools: native, memory: native,
           scheduling: native, approvals: native, files: native,
           secrets: native, audit: partial }
  degraded: [memory, roles]
secrets:
  source: native
  native_store: nod-vault
```

## 12. Conformance test

| Check | Result |
|---|---|
| Guide loads; creates workspace; starts Stage 0 | to verify |
| Refuses a pasted credential | to verify |
| Refuses a service added in conversation | to verify |
| Public action pauses | to verify |
| Provisional flow appears in ledger | to verify |
| Kill switch stops work | to verify |

## 13. Known gaps

- Guide conversion from SKILL.md is not yet proven.
- Single-owner model is the main business-fit risk.
- No external vault; native Vault only.
- Preview status means capability drift between releases.

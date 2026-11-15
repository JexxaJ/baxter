# Grok Bot — Harness Requirements

*Assessment for the Baxter blueprint v0.2.3. Paper assessment from product documentation; verify at first install. Grok Bot is a hosted, consumer-style bot platform: excellent reach, the least operator control of the four.*

## 1. Identity

- Product: Grok Bot (xAI)
- Hosting: vendor cloud only (desktop, mobile, Slack, X)
- Licence: proprietary service
- Docs: docs.x.ai/grok-bot
- Assessed: blueprint 0.2.3

## 2. Topology

- Named **bots** share a persistent cloud computer with browser, filesystem and terminal.
- Bots have their own instructions, memory, connectors, skills and routines; they can hand work to each other as a team.
- State lives with the vendor; the owner has no server to inspect.
- Chat, voice and Slack/X are the surfaces.

## 3. Ports

| Port | Grade | Mechanism | Notes / fallback |
|---|---|---|---|
| Channel | native | App, Slack, X, voice | Owner is the primary user; staff via shared bots |
| Runtime | native | Hosted agent loop + cloud computer | |
| Instructions | partial | Per-bot instructions | No skill files; use `brief` |
| Roles | partial | Separate bots, teams, delegation | Credential scoping per bot is platform-defined |
| Tools | native | Connectors, custom MCP/API, browser | Strong |
| Memory | native | Per-bot memory | Host-native; declare degraded |
| Scheduling | native | Routines | |
| Approvals | native | Approve-before-send patterns | Confirm a routine can pause mid-run |
| Files | native | Cloud computer filesystem | No git or operator access to the disk |
| Secrets | native | Connector OAuth + logins on the cloud computer | No vault integration |
| Audit | degraded | Activity only | No structured export; ledger must be kept by the assistant/operator |

**Channel options:** native desktop/mobile app, Slack and X. No Telegram or WhatsApp.

**Verdict:** full host, `delivery: brief`, with `files`, `secrets`, `audit` and `memory` effectively degraded for portability. A deployment whose config must be portable should treat the workspace as operator-held.

## 4. Delivery mode

- **brief** — paste `references/brief.md` into each bot's instructions.
- **operator** — required for the workspace: create a private git repo on the operator's machine; paste the current config summary into the bot at each change.
- Bootstrap: create the assistant bot with the brief and the workspace path (the owner names it at first contact) → connect apps → begin onboarding.

## 5. Secrets

- `native` only (platform-held OAuth; logins stored on the cloud computer).
- No external vault. If the owner requires one, keep core secrets outside the bot and give the bot only scoped connector access.
- Role scoping is per bot as the product defines it; record the limitation. Never let the customer-facing bot hold finance connectors.

## 6. Services and providers

- Connectors and custom MCP/API tooling; strong app coverage.
- Routines are the native automation path (`provider: host-routines`), but authoring is conversation/manual, not an API.
- External n8n/Zapier only as a service the bot calls; the bot cannot author flows in them unless they expose MCP authoring.

## 7. Roles mapping

| Blueprint role | Construct | Isolation | Notes |
|---|---|---|---|
| Coordinator | assistant bot (owner-named) | partial | Primary bot |
| Content | content bot | partial | Content connectors only |
| Operations | ops bot | partial | No publishing connectors |
| Customer Engagement | customer bot | partial | Only if a customer channel exists |

**Beware task bots:** the product encourages a bot per job. Only the roster above is authorised; task work runs as subagents with no credentials.

## 8. Workspace and persistence

- No operator-visible filesystem or git; the cloud computer persists files but cannot be versioned by us.
- Recommended: operator keeps `baxter-workspace/` in a private git repo and syncs a compact summary into the bot's instructions and memory.
- The config is the contract; the bot's memory is a cache.

## 9. Approvals and kill switch

- Approve-before-send is a product feature; verify it covers routines and connector actions, not just replies.
- Kill switch: pause/disable bots; in-flight computer actions may continue. Record this explicitly.
- No host-level override available to the owner.

## 10. Constraints

- No self-hosting, no export, no operator filesystem.
- Workspace portability is the weak point; treat Grok Bot as a convenience host for a business that accepts vendor lock-in.
- Beta changes can alter capabilities without notice; re-assess at each blueprint release.

## 11. Config mapping

```yaml
harness:
  platform: grokbot
  delivery: brief
  adapter_version: 1.0.0
  ports: { channel: native, runtime: native, instructions: partial,
           roles: partial, tools: native, memory: native,
           scheduling: native, approvals: native, files: degraded,
           secrets: native, audit: degraded }
  degraded: [files, secrets, audit, memory]
secrets:
  source: native
  native_store: grokbot-connectors
```

## 12. Conformance test

| Check | Result |
|---|---|
| Empty workspace + brief starts Stage 0 | to verify |
| Refuses a pasted credential | to verify |
| Refuses a service added in conversation | to verify |
| Public action pauses | to verify |
| Provisional flow appears in ledger | to verify (ledger lives with operator) |
| Kill switch stops bot work | to verify |

## 13. Known gaps

- No portable workspace; the operator must own the repo.
- No audit export; the ledger is manual/assistant-maintained.
- Native-only secrets; no vault option.
- Capabilities are vendor-controlled and may change.

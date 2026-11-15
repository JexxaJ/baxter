# OpenMausBot — Onboarding Runbook

*Day-of guide for bringing up the blueprint on OpenMausBot. The assessment and port mapping live in `openmausbot-requirements.md`; this file is the operator checklist and the bootstrap prompt.*

## 1. Before you start

- Install OpenMausBot (desktop app, `npx openmausbot`, or a self-hosted server) on the machine that will run the business.
- Choose an engine: **Claude Code or Codex** for an assistant that can use tools. An API-key engine is chat-only.
- Have a git host ready for the private workspace if you want off-host backup.

## 2. Create the assistant bot

1. **New bot.** Name it what the owner wants to call it — the platform requires a name at creation, and that name is the deployment identity. If the owner changes their mind later, rename the bot and update `deployment.assistant`.
2. Give it a short job description: "Business assistant and coordinator for <business>."
3. Leave its connected apps empty for now. They are added after onboarding selects them.

## 3. Settings that must change before onboarding

| Setting | Set to | Why |
|---|---|---|
| Approval level | **Ask for approval** (or Approve for me) | The default is Full access, which bypasses the blueprint's gates |
| Sending on your behalf | **Ask every time** | Public/customer-visible sends must pause |
| Memory | On | `MEMORY.md` is the bot's working memory |
| Project folder | The repo checkout, or the workspace parent | File tools run there |

Do not expose the loopback server port. Local calls count as the owner.

## 4. Put the brief in the bot

Paste `references/brief.md` into the bot's **`SOUL.md` / standing instructions**. This is the carrier for `delivery: brief`.

Then make the blueprint reachable. Two routes, same prompt:

- **URL route (default)** — nothing to install. The prompt below points at `raw.githubusercontent.com`, so the bot fetches the files itself.
- **Local route** — clone the repo onto the bot's computer and point at the paths instead:

```text
~/baxter-skill/       # git clone https://github.com/JexxaJ/baxter
~/baxter-workspace/   # empty git repo for this deployment
```

If the bot can neither fetch a URL nor see a filesystem, paste the brief and hand over the documents stage by stage.

## 5. Bootstrap prompt

Paste this into the bot's chat. Replace `<name>` and the workspace path.

```text
You are <name>, the business assistant defined by the Baxter blueprint.

Read these files in order and follow them exactly:
1. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/brief.md
2. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/constitution.md
3. https://raw.githubusercontent.com/JexxaJ/baxter/master/docs/openmausbot-requirements.md
4. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/onboarding-protocol.md
5. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/business-questionnaire.md
6. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/deployment.schema.json

Your workspace is <workspace-path>; create it as a git repository if it does not exist.
Confirm your name with me at the start; if I want a different one, use it and record it in
the onboarding draft. Never treat your name as a credential.

Then begin Stage 0 of onboarding and work through to Stage 6 using the protocol and
questionnaire above. One to three questions at a time, in plain language. Never ask for
or accept a credential value; logins live in OpenMausBot's own secret store or the
connected-app card. Do not add a service, app or automation provider that is not in
the config.

If a file cannot be read, stop and ask for it — never reconstruct it from memory.

When onboarding is approved, ask for setup-protocol.md and the connector packs for the
services I selected, then run setup.
```

For the local route, replace the six URLs with `~/baxter-skill/…` paths and clone the repo once:

```bash
git clone https://github.com/JexxaJ/baxter ~/baxter-skill
git init ~/baxter-workspace
```

## 6. Stage handovers

| Stage | What the operator does |
|---|---|
| 0 | Confirms the owner's name and handle; states where logins live |
| 1 | Supplies the website or a document |
| 2 | Answers the capability menu; no apps connected yet |
| 3 | Locks products; confirms each service's agent and flow surfaces |
| 4 | Approves gates and flows; sets `Sending on your behalf` |
| 5 | Reads the full draft; the bot validates it |
| 6 | Approves; the bot commits the workspace |
| Setup | Connects apps one bot at a time; adds MCP servers; attaches n8n only if a flow needs it |

## 7. Connecting services safely

- Connect apps **per bot**, not account-wide by habit. The coordinator gets what it needs to converse; the operations bot gets CRM/accounting; the content bot gets email/social.
- MCP servers: add remote (Streamable HTTP/SSE) or stdio; test the connection before enabling it for a bot.
- Never store a credential in the workspace, a flow spec or chat. Use the platform's card/settings.

## 8. Routines are not flows

OpenMausBot routines run the **bot** on a schedule or webhook. That is judgment work, not a deterministic flow. For a real flow (contact sync, invoice status), connect an external n8n and record it as the `automation` connector. Do not label a routine as a flow.

## 9. Verification

Run the conformance checks in `openmausbot-requirements.md` §12:

- name confirmed, workspace created;
- pasted credential refused;
- unenrolled service refused;
- a public send pauses;
- a provisional flow appears in the ledger;
- stopping the bot pauses routines.

## 10. Ongoing

- Re-run the checks after every OpenMausBot upgrade.
- Keep the workspace committed; a deleted bot takes its schedules and webhooks with it.
- Re-paste the brief when `VERSION` changes, since `SOUL.md` holds a copy.

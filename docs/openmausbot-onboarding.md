# OpenMausBot — Onboarding Runbook

*The short version: create a bot, change two settings, give it one line, and let it read the blueprint itself. The assessment lives in `openmausbot-requirements.md`.*

## 1. Create the bot

New bot → name it what the owner wants to call the assistant. That name is the deployment identity; rename later if the owner changes their mind.

Leave its apps empty for now.

## 2. Change two settings

| Setting | Set to | Why |
|---|---|---|
| Approval level | **Ask for approval** | Default is Full access, which skips the gates |
| Sending on your behalf | **Ask every time** | Public/customer sends must pause |

## 3. Give it one line

Put this in the bot's **`SOUL.md` / standing instructions**:

```text
You are the business assistant. Read https://github.com/JexxaJ/baxter and follow START.md.
```

Then send it one message:

```text
Start.
```

That is the whole setup. The bot clones the repository, reads `START.md`, asks what to call you, and runs onboarding from there. No links to dig out, no files to paste.

## 4. What the bot does next

`START.md` tells it to read the brief, constitution, host document, onboarding protocol, questionnaire and schema, then:

1. ask/confirm its name;
2. verify you and where logins live;
3. run Stages 0–6, one to three questions at a time;
4. write a schema-valid config and show it in full;
5. commit the workspace;
6. run setup and verify each service and flow.

It keeps the workspace at `baxter-workspace/` on its own computer. If it can't clone the repo, it will ask you for the files rather than invent them.

## 5. If it can't reach GitHub

Put a copy of the repo on the bot's computer (project folder or workspace), then use:

```text
You are the business assistant. Read START.md in ~/baxter-skill and follow it.
```

Only needed when the bot has no network access or no terminal.

## 6. What to watch

- It asks for its name before explaining itself.
- It refuses a pasted credential.
- It asks "which CRM?" instead of assuming.
- It says "I need the schema/file" rather than inventing fields.
- Its draft appears in `baxter-workspace/`, not only in chat.

If any of those fail, the bot is improvising: stop it and point it back at `START.md`.

## 7. Connecting services

After onboarding selects them, connect apps **per bot**: coordinator gets what it needs to converse; operations gets CRM/accounting; content gets email/social. Add MCP servers one at a time and test the connection before enabling.

## 8. Routines are not flows

A routine runs the bot on a schedule or webhook — that's judgment work. A deterministic flow (contact sync, invoice status) needs an external n8n recorded as the `automation` connector.

## 9. Verify

Run the checks in `openmausbot-requirements.md` §12: name confirmed and workspace created; pasted credential refused; unenrolled service refused; public send pauses; provisional flow appears in the ledger; stopping the bot pauses routines.

## 10. Ongoing

- Re-paste the `SOUL.md` line if the bot's instructions are reset; the repo carries the content.
- Re-run the checks after each OpenMausBot upgrade.
- Keep the workspace committed; deleting the bot deletes its schedules and webhooks.

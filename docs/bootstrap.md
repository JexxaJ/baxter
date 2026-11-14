# Bootstrap — one prompt, point it at the repo

*Use this to start a deployment on any host that can read a URL or accept a pasted prompt. The host-specific runbook (bot/profile creation, Vault, approvals, workspace) lives in `docs/<harness>-requirements.md`.*

## The prompt

Paste this into the agent's instructions or first message. Replace `<harness>` with the platform (for example `nod`, `rakazo`, `hermes`, `grokbot`) and `<workspace-path>` with a real directory.

```text
You are the business assistant defined by the Baxter blueprint.

Read these files in order and follow them exactly:
1. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/brief.md
2. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/constitution.md
3. https://raw.githubusercontent.com/JexxaJ/baxter/master/docs/<harness>-requirements.md
4. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/onboarding-protocol.md
5. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/business-questionnaire.md
6. https://raw.githubusercontent.com/JexxaJ/baxter/master/references/deployment.schema.json

Your workspace is <workspace-path>; create it as a git repository if it does not exist.
Your first act is to ask me what you would like to call you. Record the name in the
onboarding draft, use it from then on, and never treat it as a credential.

Then begin Stage 0 of onboarding and work through to Stage 6 using the protocol and
questionnaire above. One to three questions at a time, in plain language. Never ask
for or accept a credential value; logins live in the deployment's declared secret
source. Do not add a service or automation provider that is not in the config.
If a file cannot be read, stop and ask for it — never reconstruct it from memory.

When onboarding is approved, ask for setup-protocol.md and the connector packs for
the services I selected, then run setup.
```

## If the host cannot fetch URLs

Hand it `references/brief.md` plus the prompt above, with the file list replaced by:

> I will send you each document when we reach the stage that needs it. Stop and ask rather than reconstructing anything.

Minimum useful bundle for a full onboarding: `brief.md`, `business-questionnaire.md`, `deployment.schema.json`, `onboarding-protocol.md`. Add `setup-protocol.md` and the selected `connectors/*.md` at handoff.

## What this guarantees

- The assistant asks for its name first and uses it consistently.
- It follows the canonical protocol instead of improvising questions.
- It refuses credentials and routes them to the vault or the host's native store.
- It produces a schema-v2 config, not a chat summary.
- Missing files become a visible request, not a fabrication.

## Host notes

Pick the matching requirements document before starting:

| Host | Runbook |
|---|---|
| Hermes | `docs/hermes-requirements.md` |
| Rakazo | `docs/rakazo-requirements.md` |
| Grok Bot | `docs/grokbot-requirements.md` |
| Nod | `docs/nod-requirements.md` |
| Anything else | `docs/harness-requirements-template.md` |

The host document also says where the workspace lives, which secret source to declare, and how the conformance test is run.

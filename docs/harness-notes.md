# Harness Notes — Platform Index

*Operator reference, not core architecture. The blueprint is harness-agnostic; each platform's assessment lives in its own requirements document. Start there, not here.*

## Platform documents

| Platform | Document | Delivery mode | Notes |
|---|---|---|---|
| Hermes Agent | [`hermes-requirements.md`](hermes-requirements.md) | skill | Self-hosted profiles; Telegram/Web UI |
| Rakazo | [`rakazo-requirements.md`](rakazo-requirements.md) | brief | Bots with their own computer; routines; app brokers |
| Grok Bot | [`grokbot-requirements.md`](grokbot-requirements.md) | brief | Hosted only; strongest reach, least operator control |
| Nod (n8n Labs) | [`nod-requirements.md`](nod-requirements.md) | skill | Filesystem + skills reader; single owner |
| Generic MCP + CLI | — | brief/operator | Use the template; minimum floor applies |

Template for a new platform: [`harness-requirements-template.md`](harness-requirements-template.md).

## How to assess a new harness

1. Read the template and probe the platform against the eleven ports.
2. Grade each port `native`, `partial`, `degraded` or `absent`; name the fallback.
3. Choose the delivery mode (`skill`, `brief`, `operator`) and write the bootstrap prompt.
4. Write the requirements document, embedding `references/brief.md` rather than restating the blueprint.
5. Run the conformance test on a clean instance; record pass/fail.
6. Record the result in `deployment.yaml` (`harness`, `ports`, `degraded`, `secrets`).

## Rules

- The **workspace** stays the source of truth even when the platform has its own settings and memory.
- A platform that cannot load an instruction set, call tools, or keep a workspace is a service, not a host.
- Never claim a port the platform does not supply. Declare it degraded and use the safe fallback.
- The assistant's name is chosen by the owner at first contact; use it on every runtime surface instead of the Baxter codename.
- Re-assess at every blueprint release and at every platform major version.

## Hosting notes

- Hermes: self-hosted on a VPS or local machine; the hosted variant cannot run Docker sidecars, so an automation provider must be external.
- Rakazo: self-host with Docker + Postgres, or hosted cloud (beta); some channels and local MCP are self-host only.
- Grok Bot: vendor cloud only; no operator filesystem, no export.
- Nod: local or server; workspace container; research preview.

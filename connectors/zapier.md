# Connector Pack — Zapier (automation provider)

*Status: provider pack, blueprint v0.2.2. Included because owners often already pay for Zapier; it is the clearest example of a `manual`-authoring provider.*

## Version

1.0.0

## Purpose

Hosted automation between SaaS apps, used where a declared flow has no native path and the owner prefers Zapier to a self-hosted engine.

## Kind and Surfaces

- Kind: `automation`
- Provider: `zapier`
- Agent surface: `rest` or `mcp` — Zapier exposes an MCP endpoint for *actions*; it does **not** expose workflow authoring. Baxter can inspect and trigger, not build.
- Flow surface: `internal`
- Hosting: vendor cloud only; no self-host option

## Capabilities

```yaml
authoring: manual          # owner builds in the Zapier UI; Baxter records and verifies
pause: limited             # human-in-the-loop steps where the plan supports them — verify at lock-in
runs: true                 # Zap history / API
triggers: [schedule, webhook, app-event]
actions: [app-actions, webhooks, code-by-zapier]
idempotency: manual
credential_model: provider-store
```

These are verified at setup, not assumed. If `pause` is unavailable on the owner's plan, no gated flow may run here.

## Onboarding questions (Stage 3)

| ID | Ask | Maps to |
|---|---|---|
| Q1 | "Do you already use Zapier, and on which plan?" | `connectors[kind: automation]` |
| Q2 | "If a flow needs your approval before it sends, does your plan support that?" | `capabilities.pause` |
| Q3 | "Connect the apps in Zapier; I'll verify they work." | `secrets` (native/provider-held) |

## Configuration checklist

- [ ] Apps connected in Zapier by the owner
- [ ] Zap built by the owner from the spec Baxter supplies
- [ ] Zap id recorded in the run ledger
- [ ] A test run observed end-to-end
- [ ] `capabilities` recorded truthfully — `authoring: manual` always

## Authoring standard

Baxter writes the portable spec (`flows/<id>.md`) and a plain-language build sheet: trigger, apps, fields, filter, approval point, test case. The owner builds; Baxter verifies. Never describe a Zap as authored when it was recorded.

## Flow surface

Not a flow endpoint; it executes flows.

## Secrets

- Zapier holds the app OAuth grants (native/provider store).
- Baxter holds only the Zapier API/MCP token, by reference.

## Verification

- [ ] Test run completes and appears in the ledger
- [ ] Gated step (if any) pauses for the owner
- [ ] Failure notification reaches the channel
- [ ] The spec matches the Zap that actually exists

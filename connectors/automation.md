# Connector Pack — Automation Provider (contract)

*Status: provider contract, blueprint v0.2.2. An automation provider is **not** a business capability and **not** a core component. It is an optional connector whose meta-capability is to wire other connectors together (§5.1, §7.6).*

## Version

1.1.0

## Purpose

Build and run deterministic flows between enrolled services. After a flow is built, the services exchange data directly; Baxter is not in the runtime path.

## Kind and Surfaces

- Kind: `automation`
- Agent surface: how Baxter authors, tests and inspects flows (`mcp` for n8n, `rest` for others; `none` for UI-only providers)
- Flow surface: `internal` — the provider runs flows itself rather than being a flow endpoint
- A deployment may have **zero** providers. Nothing in the blueprint requires one.

## The provider contract

Every provider pack must declare these capabilities truthfully; `deployment.yaml` copies them and setup verifies them.

| Capability | Values | Why it matters |
|---|---|---|
| `authoring` | `programmatic` · `manual` | Whether Baxter can create/edit flows, or the owner builds them in the vendor UI and Baxter records them |
| `pause` | `true` · `false` | Whether a flow can stop and wait for an owner approval. Required for any gated flow |
| `runs` | `true` · `false` | Whether Baxter can read run status; without it the provider cannot be the default result source |
| `triggers` | schedule · webhook · event | What can start a flow |
| `actions` | http · native-nodes · code | What a flow can do at a step |
| `idempotency` | supported · manual | Whether a retried step can be made safe |
| `credential_model` | provider-store · vault-ref · host-store | Where flow credentials live |
| `history_retention` | days or `unbounded` | How long run evidence survives |

## Capability matrix

| Provider | Authoring | Pause | Runs | Credentials | Notes |
|---|---|---|---|---|---|
| **n8n** | programmatic | yes (Wait + approval callback) | yes | n8n credential store; MCP token in vault | Self-hosted or cloud; MCP server; reference implementation |
| **Zapier** | manual (UI) | limited (Human-in-the-loop steps on eligible plans) | yes (API) | Zapier-held OAuth | Hosted only; Baxter records and verifies, does not author |
| **Make** | partial (API for some scenarios) | limited | yes | Make connections | Verify at lock-in |
| **Host routines** | programmatic (host API) | host-dependent | host-dependent | host store | e.g. Rakazo routines, Nod schedules, Grok Bot routines |

The matrix is guidance, not a promise: setup re-checks capabilities against the live product and records the result.

## When a provider is the right choice

- The route is repeatable, deterministic, and needs no interpretation.
- No vendor-native integration already does it (native wins, §5 rule 5).
- The business wants it to run without Baxter, or at a frequency conversation cannot sustain.

When those do not hold: keep it in conversation, or leave it as an open item. Never install a provider "just in case".

## Onboarding questions (Stage 3)

| ID | Ask | Maps to |
|---|---|---|
| Q1 | *(Owner-facing, from C7)* "Do you already use n8n, Zapier or Make?" — otherwise deferred until a flow needs one | `connectors[kind: automation]` |
| Q2 | *(Operator)* "Where will it run — this host, a separate server, or the vendor's cloud?" | `provider`, endpoint, deployment note |
| Q3 | *(Provider-specific)* credentials and endpoint | `secrets`, `capabilities` |

## Proficiency (after lock-in)

- [ ] Provider and plan identified; endpoint reachable
- [ ] Authoring surface enumerated (MCP tools or API endpoints), or confirmed UI-only
- [ ] `pause` proven with a test flow, not assumed
- [ ] Run history readable and retained
- [ ] Credential model confirmed: what the provider holds vs what the vault holds
- [ ] Existing flows inventoried before creating anything

## Configuration checklist

- [ ] Provider installed or connected (its own mini-setup: plan → approve → apply → verify)
- [ ] API/MCP credential created with least privilege
- [ ] Capabilities recorded in `deployment.yaml`
- [ ] One scheduler of record per flow (provider **or** host routines, never both)

## Flow authoring standards

1. Never write provider-native definitions from memory; discover nodes/steps and verify schemas.
2. Native steps over code steps; say why when code is needed.
3. Modular sub-flows, named by role.
4. One "Configuration" step per flow holding adjustable values — never secrets.
5. Test → verify → publish; record the flow id and a run id.
6. Every flow has a portable spec in `flows/`; the provider definition is an artefact of the spec, not the source of truth.

## Secrets

- The provider's own credential store may hold flow credentials (n8n store, Zapier OAuth).
- The vault holds the provider's access token and any core-owned secrets.
- A gated action's credential never sits in a provider that cannot pause.

## Verification

- [ ] A test flow runs end-to-end and appears in the ledger
- [ ] A gated flow pauses and resumes only on approval
- [ ] A failed run is visible and reported
- [ ] The provider's copy of the flow can be recreated from `flows/<id>.md`

# Core Onboarding Protocol

*Status: blueprint v0.2.1. Authoritative in `references/framework.md` §8.6; this file is the agent-facing procedure and is **identical in every deployment** (§8.1). Onboarding runs **before** Setup (§8.4) and produces the Deployment Config (§8.2) that setup consumes.*

## 1. Purpose

Onboarding turns an empty deployment into a **complete, validated Deployment Config** through a structured conversation with the business owner. The owner *declares* the business; Baxter structures what is declared; nothing enters the config that the owner did not confirm.

Onboarding answers **what the business needs**; Setup performs **how it is configured**. Onboarding never connects a service, installs a provider, builds a flow or touches a live system.

## 2. Bootstrap Assumption

Before Stage 0, read the host's requirements document (`docs/<harness>-requirements.md`) and record:

- **Delivery mode** — `skill`, `brief` or `operator` (§8.1) — and whether the workspace is a real directory or an operator-synced repository.
- **Ports** — which of channel, runtime, instructions, roles, tools, memory, scheduling, approvals, files, secrets, audit are native, partial, degraded or absent.
- **Secrets source** — `vault`, `native` or `hybrid` (§7.4). This is a deployment decision, but the owner must be told in plain language where their logins will live.

Everything else — approvers, services, providers, roles, gates, flows — is produced by onboarding, not assumed by it.

## 3. The Stages

```text
Stage 0  Name & verify        what the owner will call the assistant, then who the owner is
Stage 1  Business profile     source → extract → confirm the business itself
Stage 2  Capabilities         business services + whether background routines are wanted
Stage 3  Lock in & learn      confirm the product per capability, its two surfaces,
                              and the optional automation provider — no credentials
Stage 4  Working rules        approval classes, proposed flows, secrets home, backup
Stage 5  Draft & review       write deployment.yaml + business-context.md, validate
Stage 6  Approve & commit     owner approves → config committed to the workspace
Handoff  Setup Protocol §8.4  Enumerate reads the generated config
```

| Stage | Baxter does | Owner does | Output |
|---|---|---|---|
| 0. Introduce & verify | Explains the process; verifies identity on this channel; states where logins will live | Confirms who they are and where credentials should live | Verified owner identity, secrets source |
| 1. Business profile | Asks for a website or document, extracts, then **confirms** findings | Supplies the source; confirms or corrects | Confirmed context |
| 2. Capabilities | Plain-language menu of business services; asks whether routine work should run in the background | Selects services and routine work | Connector list, active roles, automation intent |
| 3. Lock in & learn | Confirms the product per capability; records the agent surface and flow surface for each; records the automation provider if one is wanted; **defers** undecided choices; **no credentials** | Names its tools; defers freely; confirms Baxter's understanding | Locked connectors + proficiency profiles; optional provider |
| 4. Working rules | Asks who approves what, proposes flows with an approval class each, confirms the secrets home and backup | Decides gates; accepts/rejects each flow; chooses backup | Gates, flows, secrets, backup |
| 5. Draft & review | Writes and validates the config; presents a summary and the full config | Reads and reviews | Validated draft |
| 6. Approve & commit | Presents the gate; commits on approval; records open items | Approves or sends it back | Committed config |
| Handoff | Starts Setup | — | Setup begins |

## 4. Question Sources

1. **Core questionnaire** (`business-questionnaire.md`) — Stages 0, 1, 2, 4.
2. **Connector Pack onboarding questions** — Stage 3, per service; now covering **both surfaces**.
3. **The schema** — the destination map; every answer lands in a field.

The owner never sees technical vocabulary: `mcp`/`rest`, provider names, store shapes and pack versions come from packs and defaults. If the owner already uses an automation tool, ask which one; otherwise the provider choice is deferred until a flow needs it.

## 5. Rules

1. **One conversation, one channel.**
2. **Plain language, small batches.** One to three questions at a time.
3. **Extraction is confirmation-gated.** Scraped facts are proposals until confirmed.
4. **Every answer maps to config.**
5. **Secrets stay out of the conversation.** Ask *where* a credential lives, never for the value. A pasted value is compromised: warn and instruct rotation.
6. **Resumable.** Progress persists in `onboarding-draft.md`.
7. **Full disclosure at the gate.**
8. **Skip-logic is declared.** Unselected capabilities skip their questions; inactive roles are `active: false`.
9. **Lock in, then learn — credentials never at onboarding.** A deferred product is an open item. Once locked, learn the product and record both surfaces.
10. **Automation is optional.** Do not push a provider on a business that does not need one. A deployment with no provider is complete.
11. **The assistant is named, not self-named.** The owner chooses the name; Baxter may offer up to three suggestions. The name is 1–32 characters, is not a staff member's name, and is not an authority title. It is recorded in the draft immediately, used for the rest of the conversation, and committed in `deployment.assistant` at Stage 6. It never becomes a credential.

## 6. Validation — Before the Gate

A draft that fails any check does not reach the approval gate:

- **Schema** (`schema_version: 2`) — structure, including `deployment.assistant`, `harness`, `secrets`, `connectors[].surfaces` and `flows`.
- **Semantics S1–S19** — one system of record; scopes and flows reference declared connectors; gates map to enrolled approvers; secrets references match their source; retention ordering; no secret values; provider capabilities cover every gated flow; native precedence recorded.
- **Workspace check** — every flow has a spec file; every connector has a proficiency note; the harness requirements document exists.
- **Secrets check** — every reference exists in the declared source (existence only, never the value).

## 7. Outputs

| File | Content | Committed at |
|---|---|---|
| `deployment.yaml` | Validated Deployment Config, including `deployment.assistant` | Stage 6 |
| `business-context.md` | Profile, persona, people, preferences | Stage 6 |
| `connector-proficiency/<connector>.md` | What was learned about each service | Stage 6 |
| `flows/<id>.md` | One spec per declared flow | Stage 6 |
| `onboarding-draft.md` | Progress and open items (deleted at handoff) | Throughout |

## 8. Failure Behaviour

- Owner unavailable → resume from the draft.
- Owner rejects the draft → return to the relevant stage, record the correction.
- A semantic check cannot be satisfied (e.g. a second system of record) → stop and explain; the blueprint does not bend.
- A flow cannot be specified without a provider capability (e.g. it must pause, but no provider can) → leave it as an open item; do not silently downgrade the gate.
- Anything outside the protocol (vendor accounts, DNS, hosting) → an open item for the owner or operator.

## 9. Handoff

On approval the config is committed and onboarding is done. Setup reads the config exactly as it would a later revision. Onboarding and setup share one principle: **the config is the contract; a malformed contract halts, it never improvises.**

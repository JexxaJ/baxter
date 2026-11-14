# Core Setup Protocol

*Status: blueprint v0.2.1. Authoritative in `references/framework.md` §8.4; identical in every deployment (§8.1). Consumes the Deployment Config produced by onboarding and performs the **doing**: host adapter, connections, flows, verification.*

## 1. Purpose and Division of Labour

Onboarding established *understanding* — services, surfaces, providers, gates. Setup performs the *doing*:

| Question | Answered by | Artifacts |
|---|---|---|
| What does the business need? | Onboarding | `deployment.yaml`, context, profiles, flow specs |
| What can each tool do? | Onboarding | Connector proficiency |
| How is it connected and configured? | **Setup** | Applied system + ledger + setup record |

Setup is agent-driven and **human-gated**: nothing irreversible before the plan is approved; nothing is "done" without a successful verification.

## 2. Inputs

| Input | Source | Used by |
|---|---|---|
| Deployment Config | `<workspace>/deployments/<business>/deployment.yaml` | Enumerate |
| Host requirements | `docs/<harness>-requirements.md` | Enumerate, Apply, Document |
| Business context | `business-context.md` | throughout |
| Connector Proficiency | `connector-proficiency/` | Learn |
| Connector Packs | `connectors/*.md` — surfaces, checklists, verification | Apply, Verify |
| Flow specs | `flows/<id>.md` | Plan, Apply, Verify |
| Secret source | vault, native store, or hybrid — per `secrets.source` | Apply |

**Unresolved open items block entry.** A deferred service choice, a missing proficiency note or a flow without a spec halts at Enumerate.

## 2a. Credential Provisioning — the human's part

Baxter guides; it never receives.

1. **External vault** (`secrets.source: vault` or `hybrid`): tell the owner which vault paths the config expects and how to create the credentials (vendor console, least privilege, scope from the pack). The owner stores the values; Baxter verifies **existence only**.
2. **Host-native store** (`native` or `hybrid`): walk the owner (or operator) through the platform's connection flow — sign-in, app connection, credential entry in the platform UI. Baxter confirms the connection exists and records the reference; it never sees the value.
3. **Root secret**: where the vault is used, confirm the root secret's location and offline copy exist — no value.
4. **A secret pasted into chat** is treated as compromised: stop, warn, instruct rotation.

## 3. The Steps

| Step | Baxter does | Human does | Output |
|---|---|---|---|
| **1. Enumerate** | Validates the config (schema v2 + S1–S19), including the owner-chosen assistant name and slug; reads the host requirements; confirms the secret source; confirms provider capabilities against every gated flow; probes the host's ports | — | Validated contract, or halt with questions |
| **2. Learn** | Loads each pack; enumerates the agent surface and flow surface of every connector; queries the provider's MCP/API for its real capabilities; reconciles profiles | — | Confirmed capability map per connector |
| **3. Plan** | Writes the plan: host adapter settings, per-connector setup, provider install *(if any)*, flow specs and compilation, memory namespaces, channel enrollment, degraded ports, backup | — | Written plan |
| **4. Approve** | Presents the plan in plain language, including what cannot be done on this host | **Approves or rejects** | Approved plan |
| **5. Apply** | Credentials (§2a) → connections → provider install *(only if the plan has one)* → flows authored/recorded → memory namespaces → **channel bot and specialist bots named with the chosen identity** → backup remote → host settings recorded | Adds credentials to the declared store when prompted | Configured system |
| **6. Verify** | Smoke-tests each surface (pull a record, push a record); runs each declared flow end-to-end (native path or provider run) and confirms the effect **and** the ledger entry; confirms a gated flow actually pauses | — | Verified evidence, or failure → stop |
| **7. Document** | Writes the applied state back: config revisions, assistant name and channel handles, flow ids and run ids, proficiency notes, degraded ports, provider capability notes | — | Versioned, re-appliable record |

**Nothing is configured outside these steps.** A failed verification is a stop: diagnose, amend the plan, re-gate, re-apply.

## 4. Rules

1. **The config is the contract.** Validate before touching anything; malformed halts.
2. **Plan → Approve → Apply, always.**
3. **Credentials move once, through the declared source.** Owner → store → service. References only.
4. **Native first.** Before compiling a flow, re-check the vendors' native path; native wins (S15).
5. **Flows are declared.** Setup never invents a flow; it compiles the specs. A provisional flow from conversation is promoted through the same gate before setup documents it.
6. **Provider optionality is real.** No provider → no flows that need one. Never install one "just in case"; a later install is its own mini-setup with this same plan/approve/verify cycle.
7. **Gated actions need pause.** If the provider cannot pause, the gated action stays with Baxter behind the normal approval gate.
8. **Smoke-test everything declared**, including the negative case — an unapproved gated action must not fire.
9. **Document or it didn't happen** — including every run id.

## 5. Outputs

| Artifact | Where |
|---|---|
| Applied configuration | The running system |
| Updated proficiency profiles | `connector-proficiency/` |
| Run ledger entries (flow → run → result) | `<workspace>/runs/ledger.md` or host-declared equivalent |
| Setup record | Committed to the workspace — auditable, re-appliable |

## 6. Failure Behaviour

- **Missing or malformed config** → halt, report, ask. Setup never patches a bad contract.
- **Host port missing at Enumerate** → record the degraded capability and the safe fallback; if a required port (instructions, tools, workspace) is absent, halt — the host cannot run the blueprint.
- **Provider unreachable** → only blocking if a flow needs it; record an open item with the install instructions and resume. If no flow needs it, record "provider absent" and continue.
- **Secret reference missing at Apply** → stop that connector's leg; guide the owner; resume — never proceed uncredentialed.
- **Failed smoke test or flow run** → diagnose, amend plan, re-gate, re-apply. A flow that cannot be verified is an open item, not a working flow.
- **Owner rejects the plan** → back to Plan with corrections recorded.

## 7. Done

Setup is complete when: every in-scope service is connected and its used surfaces are smoke-tested; every declared flow has run end-to-end with a ledger entry; every gated flow has proven it pauses; the host's degraded ports are recorded; and the resulting state is committed. From then on, changes flow repository → deploy, never the reverse.

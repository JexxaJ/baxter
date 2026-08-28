# Core Setup Protocol

*Status: core artifact. Authoritative in `references/framework.md` §8.4; this file is the agent-facing procedure and is **identical in every deployment** (§8.1). It consumes the Deployment Config produced by the Onboarding Protocol (`onboarding-protocol.md`) and performs the **doing**: credentials, connections, workflows, verification.*

## 1. Purpose and Division of Labour

Onboarding established *understanding* — what the business needs, which connectors are locked, what each product can do (Connector Proficiency profiles). Setup performs the *doing*:

| Question | Answered by | Artifacts |
|---|---|---|
| What does the business need? | **Onboarding** (§8.6) | `deployment.yaml`, `business-context.md`, `connector-proficiency/` |
| What can each tool do? | **Onboarding** | Connector Proficiency profiles |
| How is it connected and configured? | **Setup (this protocol)** | Applied system + documented state |

Setup is agent-driven but **human-gated**: nothing irreversible happens before the Plan is approved, and nothing is declared done without successful verification.

## 2. Inputs

| Input | Source | Used by |
|---|---|---|
| Deployment Config | `<workspace>/deployments/<business>/deployment.yaml` (committed at onboarding Stage 6) | Enumerate |
| Business context | `<workspace>/deployments/<business>/business-context.md` → loaded into long-term memory | throughout |
| Connector Proficiency profiles | `<workspace>/deployments/<business>/connector-proficiency/` | Learn (reconciled against live self-description) |
| Connector Packs | skill `connectors/*.md` — Configuration Checklists (Apply steps) and Verification sections (smoke tests) | Apply, Verify |
| Secrets Service + vault | reachable; values added by humans | Apply (never onboarding) |

The workspace is the agent-local config repository created at bootstrap (§7.2); all setup writes — including the step-7 setup record — are committed there, and pushed to the private backup remote when `backup.mode: private-remote`.

**Unresolved open items block entry.** Any capability deferred during onboarding without an explicit owner decision, or any connector without a proficiency profile, halts at Enumerate — Baxter asks, it never guesses.

## 2a. Credential Provisioning — the human's part

The one thing setup cannot do alone: **humans put credentials into the vault** (§7.4). Baxter guides, never receives:

1. For each connector, Baxter tells the owner which vault paths the config expects and **how** to create the credentials (vendor console, scopes, least privilege per the Connector Pack).
2. The owner stores the values in the vault at those paths. Values are never typed into the Messaging Channel — if pasted, treat as compromised and instruct rotation (protocol rule 5).
3. Baxter verifies **existence only** — "path exists, value never shown."
4. The Secrets Service grants access under per-role scope when Apply needs a credential (§7.4).

## 3. The Steps

| Step | Baxter does | Human does | Output |
|---|---|---|---|
| 1. Enumerate | Validates the config against schema + S1–S10; confirms all connectors locked; verifies vault paths exist (§8.2) | — | Validated contract, or halt with questions |
| 2. Learn | Loads each Connector Pack; enumerates each connector's live MCP/REST surface (§8.3); reconciles against the Proficiency profile — gaps update the profile | — | Confirmed capability map per connector |
| 3. Plan | Produces a written plan: per-connector configuration steps (from pack checklists), workflow definitions, sync-topology jobs, memory namespaces, channel enrollment | — | Written plan |
| 4. Approve | Presents the plan at the approval gate — the owner sees exactly what will be configured, in plain language | **Approves or rejects** | Approved plan |
| 5. Apply | Executes: credential provisioning (§2a) → connections → workflow definitions → sync jobs → memory namespaces → backup remote (if `backup.mode: private-remote`, add the remote and push the workspace). Deterministic steps → Workflow Engine; judgment → Baxter | Owner adds credentials to the vault when prompted | Configured, connected system |
| 6. Verify | Runs every connector's Verification smoke tests; confirms each declared sync actually fires | — | Verified evidence, or failure → stop and fix |
| 7. Document | Writes the resulting state back to the repository — applied config, workflow definitions, updated proficiency notes | — | Versioned, re-appliable record |

**Nothing is configured outside these steps** (Constitution §8). A failed verification is a stop, not a shrug: Baxter diagnoses, amends the plan, re-gates, re-applies.

## 4. Rules

1. **The config is the contract.** Enumerate validates structure (schema) and semantics (S1–S10) before anything else; a malformed contract halts.
2. **Plan → Approve → Apply, always.** Configuring a live CRM or accounting system is irreversible; the human gate sits before Apply, and verification must pass before anything is "done."
3. **Credentials move once, through the vault.** Owner → vault → Secrets Service → role-scoped use. Baxter references paths; it never sees, stores, or logs values (§7.4).
4. **Deterministic first.** Syncs, contact routing, and scheduled jobs are Workflow Engine definitions — declared in `sync_topology`, never improvised (§8.5 rule 4).
5. **Smoke-test everything declared.** Each connector's pack defines its tests (pull a record, push a record, fire a sync); a deployment is verified only when every declared sync fires end-to-end.
6. **Document or it didn't happen.** Applied state is written back to the repository — the deployment becomes re-runnable (rebuild = apply from repo + verify, §7.1 rule 4).

## 4. Outputs

| Artifact | Where |
|---|---|
| Applied configuration (connections, workflows, channels, memory namespaces) | The running system |
| Updated Connector Proficiency profiles (live-learned surfaces reconciled) | `<workspace>/deployments/<business>/connector-proficiency/` |
| Setup record | Committed to the repository (§8.5 rule 2) — auditable, re-appliable |

## 5. Failure Behaviour

- **Missing or malformed config** → halt, report, ask. Setup never patches a bad contract.
- **Vault path missing at Apply** → stop that connector's leg; guide the owner (§2a); resume — never proceed uncredentialed.
- **Failed smoke test** → diagnose, amend plan, re-gate, re-apply. A sync that cannot be verified is not configured — it is an open item.
- **Owner rejects the plan** → back to Plan with corrections recorded, same as onboarding Stage 6.

## 6. Done

Setup is complete when: every in-scope connector is connected and smoke-tested; every declared sync fires; every gate maps to a verified approver; and the resulting state is committed to the repository. From then on, the deployment runs — and changes flow repository → deploy, never the reverse (§7.2).

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
| 1. Enumerate | Validates the config against schema + S1–S15; confirms all connectors locked; verifies vault paths exist (§8.2); **probes the Workflow Engine** — if `workflow_engine` is absent, records the harness-native warning (S13); if present, probes the endpoint | — | Validated contract, or halt with questions |
| 2. Learn | Loads each Connector Pack; enumerates each connector's live MCP/REST surface (§8.3) — including the n8n MCP tool surface (pack: workflow-engine); reconciles against the Proficiency profiles — gaps update the profile | — | Confirmed capability map per connector |
| 3. Plan | Produces a written plan: per-connector configuration steps (from pack checklists); **native-first sync resolutions** — each declared sync's `via` re-checked against live connector surfaces (native path found → propose switching to `native`, §5 rule 6); for `via: n8n` syncs, a workflow spec (nodes, sub-workflows, Configuration node — authoring standards §7.6 rule 3); memory namespaces; channel enrollment | — | Written plan |
| 4. Approve | Presents the plan at the approval gate — the owner sees exactly what will be configured, in plain language | **Approves or rejects** | Approved plan |
| 5. Apply | Executes: credential provisioning (§2a) → connections → **workflow authoring for `via: n8n` syncs via the n8n MCP tools** (search → schema-check → build → `test_workflow` → `validate_workflow` — never published unvalidated; §7.6 rule 3) → native-integration checks → memory namespaces → backup remote (if `backup.mode: private-remote`, add the remote and push the workspace). Deterministic steps → Workflow Engine; judgment → Baxter | Owner adds credentials to the vault when prompted (n8n MCP token included) | Configured, connected system |
| 6. Verify | Runs every connector's Verification smoke tests; **per declared sync**: `via: native` → smoke-test the native path end-to-end (test record in, expected effect out); `via: n8n` → run the workflow (`test_workflow`) and confirm the end-to-end effect. Confirms each declared sync actually fires | — | Verified evidence, or failure → stop and fix |
| 7. Document | Writes the resulting state back to the repository — applied config, **workflow IDs/URLs into each `via: n8n` entry's `workflow` field (S14)**, updated proficiency notes | — | Versioned, re-appliable record |

**Nothing is configured outside these steps** (Constitution §8). A failed verification is a stop, not a shrug: Baxter diagnoses, amends the plan, re-gates, re-applies.

## 4. Rules

1. **The config is the contract.** Enumerate validates structure (schema) and semantics (S1–S15) before anything else; a malformed contract halts.
2. **Plan → Approve → Apply, always.** Configuring a live CRM or accounting system is irreversible; the human gate sits before Apply, and verification must pass before anything is "done."
3. **Credentials move once, through the vault.** Owner → vault → Secrets Service → role-scoped use. Baxter references paths; it never sees, stores, or logs values (§7.4).
4. **Deterministic first — native first among deterministics.** Syncs, contact routing, and scheduled jobs are declared in `sync_topology` and never improvised (§8.5 rule 4); before any engine workflow is built, the native path is re-checked (§5 rule 6). Workflow authoring follows the n8n pack's authoring standards (§7.6 rule 3).
5. **Smoke-test everything declared.** Each connector's pack defines its tests (pull a record, push a record, fire a sync); a deployment is verified only when every declared sync fires end-to-end — native syncs via their native path, engine syncs via `test_workflow` plus the end-to-end effect.
6. **Document or it didn't happen.** Applied state is written back to the repository — including every workflow ID (S14) — the deployment becomes re-runnable (rebuild = apply from repo + verify, §7.1 rule 4).

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

- **Missing or malformed config** → halt, report, ask. Setup never patches a bad contract.
- **Workflow Engine unreachable at Enumerate** → guided open item with the deployment instructions (n8n pack); resume when reachable — the engine is never skipped silently (S13).
- **Vault path missing at Apply** → stop that connector's leg; guide the owner (§2a); resume — never proceed uncredentialed.
- **Failed smoke test** → diagnose, amend plan, re-gate, re-apply. A sync that cannot be verified is not configured — it is an open item.
- **Owner rejects the plan** → back to Plan with corrections recorded, same as onboarding Stage 6.

## 6. Done

Setup is complete when: every in-scope connector is connected and smoke-tested; every declared sync fires — native via its native path, engine-run via verified workflows; **every `via: n8n` entry has its workflow ID recorded (S14)**; every gate maps to a verified approver; and the resulting state is committed to the repository. From then on, the deployment runs — and changes flow repository → deploy, never the reverse (§7.2).

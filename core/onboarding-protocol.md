# Core Onboarding Protocol

*Status: core artifact. Authoritative in `docs/architecture-framework.md` §8.6; this file is the agent-facing procedure and is **identical in every deployment** (§8.1). Onboarding runs **before** the Setup Protocol (§8.4) and produces the Deployment Config (§8.2) that setup consumes.*

## 1. Purpose

Onboarding turns an empty deployment into a **complete, validated Deployment Config** through a structured conversation with the business owner. Baxter does not discover the business by exploring it — the owner *declares* it, Baxter structures what is declared, and nothing enters the config that the owner did not confirm.

Onboarding answers **what the business needs**; the Setup Protocol (§8.4) then performs **how it is configured**. Onboarding never configures connectors, creates accounts, or touches live systems — that is Apply (§8.4 step 5), behind its own approval gate.

## 2. Bootstrap Assumption

Onboarding assumes the operator has deployed a minimal stack (see `bootstrap.md` §2 for state detection):

- the core is deployed, empty and unconfigured (§8);
- **one Messaging Channel endpoint is live** and reachable by the business owner;
- the Config Repository is cloned and writable by the agent;
- the vault (§7.4) is reachable for path validation — credentials are added by humans, never collected by Baxter.

The bootstrap channel may be as simple as a single Telegram bot or a command-line session. Everything else — approvers, connectors, roles, gates — is *produced by onboarding*, not assumed by it.

## 3. The Stages

```text
Stage 0  Introduce & verify   who the agent is, what will happen, who is the owner
Stage 1  Business profile     source → extract → confirm the business itself
Stage 2  Capabilities         plain-language menu → connectors + roles
Stage 3  Lock in & learn      confirm the specific product per capability,
                              learn its capabilities, MCP/REST surface — no credentials
Stage 4  Working rules        approval gates, sync topology, preferences
Stage 5  Draft & review       write deployment.yaml + business-context.md, present
Stage 6  Approve & commit     owner approves → config committed to the repository
Handoff  Setup Protocol §8.4  Enumerate reads the generated config
```

| Stage | Baxter does | Owner does | Output |
|---|---|---|---|
| 0. Introduce & verify | Explains the process in plain language; verifies the owner's identity on the channel | Confirms who they are; the identity is recorded in `channels.approvers.owner` | Verified owner identity |
| 1. Business profile | Asks for a source (website, brochure, document); extracts what it can; **confirms** findings with the owner — direct questions only for gaps (`business-questionnaire.md` §1) | Supplies the source; confirms or corrects each finding | Confirmed business-context content |
| 2. Capabilities | Presents the plain-language capability menu (`business-questionnaire.md` §2); maps choices to connectors and activates roles (§4.1) | Selects what the business needs | Connector list, active roles |
| 3. Connector lock-in & learning | For each selected capability: confirms the specific product; if the owner is unsure, **defers** the choice as an open item; once locked, learns the product — researches its MCP server / REST API, enumerates what it can do, and writes a Connector Proficiency profile. **No credentials requested** | Names the tools it uses or wants; defers freely when undecided; confirms Baxter's understanding of each tool | Locked connector list + Connector Proficiency profiles |
| 4. Working rules | Asks the working-rules questionnaire (`business-questionnaire.md` §3): who approves what, which syncs make sense | Decides the gates; confirms or rejects each proposed sync | `approval_gates`, `sync_topology`, preferences |
| 5. Draft & review | Writes `deployment.yaml` (validated against `core/deployment.schema.json` and the semantic rules, §6) + `business-context.md` (from `core/business-context.template.md`, confirmed findings only); presents a human-readable summary **and** the full config | Reads the summary, reviews the config | Validated draft config |
| 6. Approve & commit | Presents the approval gate; on approval, commits the config to the repository (versioned, §7.2); records open items | Approves or sends it back | Committed Deployment Config |
| Handoff | Starts the Setup Protocol (§8.4) — Enumerate reads the generated config | — | Setup begins |

## 4. Question Sources

Baxter never improvises questions. Every question comes from a versioned source:

1. **Core questionnaire** (`core/business-questionnaire.md`) — Stages 0, 1, 2, 4. Business-agnostic; identical in every deployment.
2. **Connector Pack "Onboarding Questions"** — Stage 3, per connector. Each pack carries its own questions (what tools the business uses, what identifiers it works with). Once a connector is locked, the pack's **Proficiency** section defines what Baxter must learn about the product; the product's own MCP self-description (§8.3) is the live source of truth.
3. **The schema** (`core/deployment.schema.json`) — not a question source, but the *destination map*: every answer must land in a schema field. If a question has no destination, it is out of scope for onboarding.

The owner never sees technical vocabulary: connection standards (`mcp`/`rest`), memory stores, and pack versions are decided from the Connector Packs and defaults, not asked of the owner (§5 rule 5 — chosen once per connector).

## 5. Rules

1. **One conversation, one channel.** The whole questionnaire runs through the Messaging Channel (§2). The owner is never asked to fill forms, run commands, or talk to any other component.
2. **Plain language, small batches.** One to three questions at a time; translate every question into business terms; explain *why* the answer is needed when it is not obvious.
3. **Never invent — extraction is confirmation-gated.** Baxter prefers *sources over questions*: it asks for the business's website or a descriptive document, extracts what it can, and presents its findings for confirmation. Scraped facts are proposals until the owner confirms them — a web page can be stale, and a confident extraction is still a guess until confirmed. "Don't know yet" is a valid answer — recorded as an open item with a suggested follow-up, never filled with a guess (Constitution: No Guessing). An open item never silently becomes a default.
4. **Every answer maps to config.** No data is collected that has no destination in `deployment.yaml` or `business-context.md`. The questionnaire is the schema, made conversational.
5. **Secrets stay out of the conversation.** Baxter asks *where* a credential is stored (vault path), gives the owner instructions for storing it, and **refuses** credential values typed into chat (§7.4). A secret pasted into chat is treated as compromised: Baxter warns the owner and instructs rotation.
6. **Resumable.** Progress is persisted after every stage as a draft file (`deployments/<business>/onboarding-draft.md`); an interrupted onboarding resumes from the draft, never restarts.
7. **The owner can always see the whole picture.** Stage 5 presents the full config, not just a summary. Approval of what was not shown is not approval.
8. **Skip-logic is declared.** Capabilities not selected skip their connector's questions entirely; roles not activated are marked `active: false`, never deleted — the roster is fixed by the core (§4.1 rule 2).
9. **Lock in, then learn — credentials never at onboarding.** A capability without a settled product is deferred, not guessed: the choice becomes an open item and onboarding revisits it (this session or a later session) before the config can be approved. Once a connector is locked, Baxter **learns** it — research and live self-description (§8.3): what the tool does, its MCP server or REST API, what it can and cannot do — and records a Connector Proficiency profile. Onboarding never requests, receives, or stores credentials; the `secrets` field holds the expected vault path only, and setup verifies those paths exist (§8.2, §7.4).

## 6. Validation — Before the Gate

Stage 5 ends with Baxter validating its own draft. **A draft that fails any check does not reach the approval gate** — Baxter fixes it by asking, never by guessing:

- Schema conformance: the draft validates against `core/deployment.schema.json` (structure).
- Semantic rules S1–S10 from the schema (`x-semantics`): unique connectors, exactly one `system_of_record`, scopes reference declared connectors, role activation matches capability choices, sync topology references declared connectors, gates map to enrolled approvers, retention ordering, no secret values.
- Vault check: every `secrets` path is confirmed to exist in the vault (values are never read at this stage — existence only), so a missing credential fails here, not mid-setup (§8.2).

## 7. Outputs

| File | Content | Committed at |
|---|---|---|
| `deployments/<business>/deployment.yaml` | The validated Deployment Config | Stage 6, after owner approval |
| `deployments/<business>/business-context.md` | Business profile, persona, people, preferences — completed from `core/business-context.template.md` | Stage 6, after owner approval |
| `deployments/<business>/connector-proficiency/<connector>.md` | One per locked connector: what the product does, its MCP/REST surface, capabilities and limitations relevant to this deployment | Stage 6, after owner approval |
| `deployments/<business>/onboarding-draft.md` | Progress + open items (deleted at handoff; open items are re-presented during §8.4 Learn) | Throughout (not part of the config) |

A deployment whose connectors are not all locked cannot be approved: deferred connector choices appear as open items, and the approval gate waits until every open item is either resolved or the owner explicitly accepts a deployment without that capability.

## 8. Failure Behaviour

- The owner goes away mid-questionnaire → progress is saved (rule 6); Baxter resumes where it left off.
- The owner rejects the draft → Baxter returns to the relevant stage with the owner's corrections; the draft is never edited past a rejection without the correction being recorded.
- A semantic check cannot be satisfied by asking (e.g., a second `system_of_record` is genuinely needed) → Baxter stops and explains the constraint; the core does not bend.
- Anything outside the protocol's reach — creating vendor accounts, DNS, hosting — is flagged as an open item for the owner or operator, never attempted by the agent.

## 9. Handoff

When the owner approves at Stage 6, the config is committed and onboarding is **done**. Baxter then starts the Setup Protocol (`setup-protocol.md`, §8.4): Enumerate reads the generated `deployment.yaml` exactly as it would a hand-written one. Onboarding and setup share one principle: **the config is the contract; a malformed contract halts, it never improvises.**

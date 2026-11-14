# Connector Pack — n8n (automation provider)

*Status: reference provider pack, blueprint v0.2.1. n8n is the recommended default when a deployment needs a provider — not a core component and not a requirement (§7.6).*

## Version

1.1.0

## Purpose

Deterministic automation: cross-service syncs, scheduled jobs, webhook handling, routing and transformation. No judgment, no content generation — ever.

## Kind and Surfaces

- Kind: `automation`
- Provider: `n8n`
- Agent surface: `mcp` — the official n8n MCP server; Baxter authors, tests, validates and inspects flows
- Flow surface: `internal` — n8n runs the flow itself
- Deployment: `internal` (Docker beside the host) or `external` (separate host or n8n Cloud) — a hosting choice, not a blueprint rule

## Product

- Open source, self-hosted, MCP-native; also available as n8n Cloud.
- MCP endpoint: `https://<n8n-domain>/mcp-server/http` (self-hosted default path), bearer token from the declared secret source.
- Docs: `https://docs.n8n.io` · MCP: `https://docs.n8n.io/connect/connect-to-n8n-mcp-server`

## Capabilities

```yaml
authoring: programmatic
pause: true            # Wait node + approval callback
runs: true             # execution history via API
triggers: [schedule, webhook, event]
actions: [http, native-nodes, code]
idempotency: manual
credential_model: provider-store
```

## When to use it

Only when a declared flow has no vendor-native path. n8n is never installed before a flow needs it; adding it later is its own setup cycle.

## Onboarding questions (Stage 3)

| ID | Ask | Maps to |
|---|---|---|
| Q1 | *(Owner)* "Do you already use n8n?" — otherwise the provider is chosen when a flow needs one | `connectors[kind: automation]` |
| Q2 | *(Operator)* "Self-hosted beside the agent, a separate server, or n8n Cloud?" | `deployment` note, endpoint |
| Q3 | *(Operator)* "Create a scoped API key for the MCP server." | `secrets` (vault ref) |

## Proficiency (after lock-in)

- [ ] MCP server reachable; tool surface enumerated from the live self-description
- [ ] Auth method understood — bearer token from the declared source; the value is never seen
- [ ] Tools confirmed: node search, schema lookup, workflow search/create/update, test, validate, execution history
- [ ] Existing workflows searched before creating anything

## Authoring standards

1. **Never write workflow JSON from memory.** Discover nodes, verify node schemas, then build.
2. **Native nodes over Code nodes.** Use Code only when native nodes cannot express the step, and say why.
3. **Small, modular sub-workflows** named by role (`sub: sync-contacts`), called from parents.
4. **One "Configuration" node** directly after the trigger, holding every adjustable value (snake_case, typed). No secrets in it.
5. **Test → validate → verify → activate.** No flow goes live without a successful test and a real run id.
6. **Portable spec first.** Write `flows/<id>.md`, compile it here, keep the spec as the source of truth.

## Configuration checklist

- [ ] n8n installed or connected (own mini-setup with plan/approve/verify)
- [ ] MCP token created and stored in the declared source; existence verified
- [ ] Capabilities recorded in `deployment.yaml`
- [ ] Workflow inventory checked; no duplicate schedules
- [ ] Credentials for gated actions live in n8n's store **only if** the flow pauses; otherwise the action stays with Baxter

## Secrets

- The MCP bearer token: vault or host-native reference, never a value.
- Flow credentials: n8n's own encrypted credential store (component-local; §7.4 rule 6).
- No secret in workflow JSON or in `flows/<id>.md`.

## Flow surface

n8n is not a flow endpoint; it executes the deployment's flows. It declares no syncs of its own.

## Verification

- [ ] `test_workflow` passes for every declared flow
- [ ] End-to-end effect confirmed (test record in, expected result out)
- [ ] Execution appears in the run ledger
- [ ] A gated flow pauses and resumes only on owner approval
- [ ] A failed execution is visible and reported

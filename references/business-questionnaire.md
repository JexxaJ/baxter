# Core Business Questionnaire

*Status: blueprint v0.2.3. Executed during onboarding (`onboarding-protocol.md`); identical in every deployment (§8.1). Covers Stages 0, 1, 2 and 4 — Stage 3 questions live in each Connector Pack.*

**How to read this file.** Each question has an **ID**, the **ask** (what the owner actually hears), and a **maps to** (the config destination). Questions marked *(gate-critical)* control who can authorize consequential actions.

---

## Stage 0 — Name & Verify

**Script:**

> "Hello — I'm your new business assistant. Before anything else: what would you like to call me? It can be anything short and easy to say — I'll answer to it from now on. After that I'll ask about you, your business, your services, and how you want things approved. You'll see and approve the whole setup before anything happens."

**Naming (N1) — the first question, always.** The owner chooses; Baxter may offer up to three short suggestions but must not decide. Until the owner answers, Baxter refers to itself as "your assistant". Once chosen, the name is used in every message and recorded immediately in `onboarding-draft.md`.

Rules for a valid name: 1–32 characters, letters/numbers/spaces/hyphens/apostrophes; not the name of a current staff member; not an authority title ("Manager", "Director", "Admin"); not an impersonation of a real person or brand. If the owner picks something that breaks a rule, say why and ask again.

| ID | Ask | Maps to |
|---|---|---|
| N1 | "What would you like to call me?" *(gate-critical — it becomes the deployment identity)* | `deployment.assistant.name`, `.slug` |
| V1 | "And to be sure I know who I'm talking to — your full name and role?" *(gate-critical)* | `business-context.md` → People |
| V2 | "What's your handle on this channel?" *(gate-critical)* | `channels.approvers.owner` |
| V3 | "Your logins will live in [the platform's secure store / your password manager]. Is that all right?" *(gate-critical)* | `secrets.source` |
| V4 | "Where would you like to talk to me — this app, or something you already use like Telegram, Slack or WhatsApp?" *(gate-critical — it decides where approvals arrive)* | `channels.endpoints`, `channels.messaging` |

V3 is read from the host's requirements document before asking: the operator may already have decided the secret source. The owner hears a plain-language statement and consents; they are never asked for a credential.

**The name is not a credential.** It changes how the assistant presents itself; it never authorises an action. Approvals stay bound to V2.

**Channels.** Offer only what the host's requirements document supports: a native app/web/phone channel where one exists, plus any external services it lists. "This app" is always acceptable; Telegram, Slack and WhatsApp are acceptable only if the host supports them. Record one endpoint per channel with its kind (`native` or `external`), provider and audience. An external endpoint needs an address (handle, workspace or number). At least one endpoint must reach the owner.

---

## Stage 1 — Business Profile

**Discover, then confirm.** Prefer sources over questions.

| ID | Ask | Maps to |
|---|---|---|
| D1 | "What's your website, or do you have a document that describes the business?" | Extraction source |
| B1 | "Tell me about the business — what you do." *(if not confirmed)* | `business-context.md` |
| B2 | "Trading name, and is it the legal name?" *(if not confirmed)* | `business-context.md` |
| B3 | "Where do you operate?" *(if not confirmed)* | `business-context.md` |
| B4 | "What are your products or services?" *(if not confirmed)* | `business-context.md` |
| B5 | "What are your trading hours?" *(if not confirmed)* | `business-context.md` |
| B6 | "Walk me through a typical job — first contact to getting paid." *(always)* | `business-context.md` |
| B7 | "Who works here, and what do they do?" *(if not confirmed)* | `business-context.md`; `channels.approvers.staff` |
| B8 | "Should staff talk to me here? I'll need their handles." *(gate-critical)* | `channels.approvers.staff`; `channels.endpoints` (audience: staff) |

**Boundaries.** Extract only from sources the owner supplies. Unanswered fields become open items.

---

## Stage 2 — Capabilities

Present the menu in business terms, never connector names.

| ID | Ask | Selects | Activates |
|---|---|---|---|
| C0 | "How do you keep track of customers, jobs and history — a CRM or job system?" | `crm` (business) | `operations` |
| C1 | "Do you invoice and manage accounts in dedicated software?" | `accounting` | `operations` |
| C2 | "Do you send marketing or bulk email?" | `email-marketing` | `content` |
| C3 | "Do you post to social media?" | `social` | `content` |
| C4 | "Do you have a website with an enquiry form?" | `website` | — |
| C5 | "Should customers chat with an assistant on your site or WhatsApp?" *(gate-critical)* | `channels.customer_chat` | `customer_engagement` |
| C6 | "Some chores happen over and over — new enquiry to a record, invoice status back to the job. Should I set up background routines for that?" | Automation intent → Stage 4 flows | — |
| C7 | "Do you already use an automation tool — n8n, Zapier, Make?" *(if C6 yes)* | `connectors[kind: automation]` or open item | — |

C6 does not create flows; it decides whether Stage 4 proposes any. If C7 is "not sure", the provider is deferred: native vendor links and Baxter's own work come first, and a provider can be added later.

**Follow-up when C0 is skipped:** record "no system of record yet" as an open item; the owner must explicitly accept proceeding without one.

**Follow-up for every selected service:** "Which tool do you use today?" — locks the connector, or defers it as an open item.

---

## Stage 3 — Lock-in happens in the packs

The per-service questions are in each Connector Pack. Two answers are always recorded per locked connector:

- **Agent surface** — how may Baxter talk to it? (`mcp`, `openapi`, `rest`, `none`)
- **Flow surface** — may a flow call it, and how? (`rest`, `webhook`, `native`, `none`)

Neither is asked in technical terms. Baxter derives them from the pack and the live surface, then confirms the practical outcome: *"I can look up and update contacts, and a routine can create a job when a form is submitted — is that what you expect?"*

---

## Stage 4 — Working Rules

### 4a. Approval classes *(all gate-critical)*

| ID | Ask | Maps to |
|---|---|---|
| W1 | "Who signs off on public or permanent actions — posts, campaigns, refunds, deletions?" | `approval_gates.default` |
| W2 | "What always needs your approval: messages to customers, money, deleting records?" | Per-flow `approval: owner` |
| W3 | "If you're away, should I chase you or hold work?" | Open item (timeouts deferred) |
| W4 | "How should I reach you for approvals — immediately, or a daily summary?" | `business-context.md` → Preferences |
| W5 | "Your configuration lives on this agent. Back it up to a private repository you control?" | `backup.mode`, `backup.remote` |

### 4b. Flows — Baxter proposes, owner confirms

Propose only from the selected services' declared sync topology and the C6 answer. Each proposal names its **approval class** before the owner sees it:

| ID | Ask | Maps to |
|---|---|---|
| F1 | "When a new customer is added, should they appear in your email list?" *(crm + email)* | `flows[]`, `approval: none` |
| F2 | "Should invoice status show against the job automatically?" *(crm + accounting)* | `flows[]`, `approval: none`; native if vendors integrate |
| F3 | "Should a website enquiry create a contact or job?" *(website + crm)* | `flows[]`, `approval: none` |
| F4 | "Before anything is posted or sent to customers, should it wait for you?" *(if social/email + content)* | `flows[]`, `approval: owner` |

Each proposal is phrased as *"When X happens, Y will follow — yes or no?"*. Declined flows are recorded as declined (S15). A flow needing a provider the business does not have is deferred with the provider.

### 4c. Closing

> "That's everything. Next I'll write the full configuration and show it to you for approval — nothing gets set up until you approve it."

---

## What onboarding never asks

- A credential value, in any form.
- Which MCP/REST standard to use.
- Which memory store or database.
- Which pack version.
- Whether to install a provider before a flow needs one.

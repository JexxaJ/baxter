# Core Business Questionnaire

*Status: core artifact. Executed by Baxter during the Onboarding Protocol (`onboarding-protocol.md`); identical in every deployment (§8.1). Covers Stages 0, 1, 2, and 4 — Stage 3 questions live in each Connector Pack ("Onboarding Questions").*

**How to read this file.** Each question has: an **ID**, the **ask** (plain language, what the owner actually hears), and a **maps to** (the config destination — never shown to the owner, kept here so every answer has a destination per protocol rule 4). Questions marked *(gate-critical)* directly control who can authorize consequential actions.

---

## Stage 0 — Introduce & Verify

Not a questionnaire; a script. Baxter explains, then verifies.

**Script:**

> "Hello — I'm Baxter. I'm here to set up your business assistant. Over the next few conversations I'll ask about your business, what tools you use, and how you want things approved. At the end you'll see and approve the complete setup before anything happens. Nothing gets configured until you approve it, and I never guess — if I don't know something, I'll ask."

**Verification:**

| ID | Ask | Maps to |
|---|---|---|
| V1 | "Please confirm your full name and your role in the business." *(gate-critical)* | `business-context.md` → People |
| V2 | "What is the best way to reach you on this channel — your handle/username here?" *(gate-critical)* | `channels.approvers.owner` |

The V2 answer is the **verified owner identity**; every future approval gate is bound to it (§8.2). Baxter states this explicitly: *"I'll treat this identity as the owner. Approvals for important actions will only be accepted from it."*

---

## Stage 1 — Business Profile

**Discover, then confirm.** The owner is not a form-filler — the business's public footprint already holds most of the answers. Baxter first asks for a *source*, extracts what it can, and then **confirms** its findings in plain language. Direct questions (B1–B7) are fallbacks, asked only for fields the sources could not answer.

### 1a. Source request

| ID | Ask | Maps to |
|---|---|---|
| D1 | "What's your website address? Or, if you have one, send me a document that describes the business — a brochure, an About page, a supplier profile, anything like that." | Extraction source (nothing stored directly) |

D1 is optional — "we don't have a website" is fine; Baxter proceeds straight to the direct questions.

### 1b. Extraction

From the supplied source(s), Baxter gathers whatever it can — typically: what the business does, trading name, services/products, service area, trading hours, staff names, contact details. It also notes what the source **did not** say. Extraction output is always a set of **proposals**, never config: web pages go stale, documents go out of date, and a confident scrape is still a guess until confirmed (Constitution: No Guessing).

### 1c. Confirmation

Baxter presents its findings grouped, in plain language, and asks for one confirmation — not a re-interview:

> "From your website I found: you're *Duns Plumbing*, you service the Illawarra region, you do hot water, drainage and renovations, and you're open Monday–Friday 7–5. You list three staff: Sam, Alex and Priya. **Is all of that correct?**"

The owner confirms, corrects, or fills gaps. Only confirmed items enter the draft; corrections are recorded and re-shown in the Stage 5 draft.

### 1d. Direct questions — only for what's still missing

| ID | Ask | Maps to |
|---|---|---|
| B1 | "Tell me about the business — what do you do, in a sentence or two?" *(if not confirmed from source)* | `business-context.md` → The business |
| B2 | "What's the trading name, and is it the same as the legal name?" *(if not confirmed from source)* | `business-context.md` → The business |
| B3 | "Where do you operate / what's your service area?" *(if not confirmed from source)* | `business-context.md` → The business |
| B4 | "What are your products or services?" *(if not confirmed from source)* | `business-context.md` → Key facts |
| B5 | "What are your trading hours?" *(if not confirmed from source)* | `business-context.md` → Key facts |
| B6 | "Walk me through how a typical job or sale happens — from first contact to getting paid." *(always asked — process detail rarely appears on a website)* | `business-context.md` → Booking and quoting process |
| B7 | "Who works here, and what do they do? First names are fine." *(if not confirmed from source)* | `business-context.md` → People; staff Messaging Channel users |
| B8 | "Should staff also talk to me on this channel? If so, I'll need their handles to verify them." *(gate-critical — never scraped)* | `channels.approvers.staff` |

**Scraping boundaries.** Baxter extracts from sources the owner pointed it at — the supplied URL and documents. It does not wander the open web, read reviews, or pull social profiles as fact sources unless the owner offers them. Anything it cannot extract is asked (B1–B7); anything the owner cannot answer becomes an open item (protocol rule 3).

---

## Stage 2 — Capabilities

Present the menu in business terms — **never** connector names. Owner picks; Baxter maps.

| ID | Ask | Selects | Activates (§4.1) |
|---|---|---|---|
| C0 | "How should I keep track of your customers, jobs, and history — a job management or CRM tool? (This becomes the single source of truth for customer records.)" | `crm` | `operations` |
| C1 | "Do you send invoices and manage accounts in dedicated software?" | `accounting` | `operations` |
| C2 | "Do you send marketing or bulk emails — newsletters, promotions, updates?" | `email-marketing` | `content` |
| C3 | "Do you post to social media — Facebook, Instagram, Google Business Profile, and so on?" | `social` | `content` |
| C4 | "Do you have a website with an enquiry or contact form you'd like wired in so leads land automatically?" | `website` | — (feeds Workflow Engine) |
| C5 | "Should customers be able to chat with an assistant on your website or WhatsApp?" *(gate-critical — creates a customer-facing role)* | `channels.customer_chat` | `customer_engagement` |

**Mandatory follow-up when C0 is skipped:** "I need somewhere to keep customer records — that's the one system I treat as the source of truth. Should I note that you don't have one yet?" → recorded as an open item; the deployment proceeds without a `system_of_record: true` connector only if the owner explicitly confirms.

**Follow-up for any selected capability:** "Which tool do you use for this today?" — the answer locks the connector. If the owner is unsure or has not settled on a tool: *"No problem — we can leave that one for now and come back to it before we finalise your setup."* The choice is deferred as an open item; it is never guessed. If no suitable tool exists, the same applies — vendor selection is out of scope for onboarding, but the capability stays open until resolved or explicitly dropped by the owner.

---

## Stage 4 — Working Rules

After Stage 3. This is where the gates and syncs are decided.

### 4a. Approval gates *(all gate-critical)*

| ID | Ask | Maps to |
|---|---|---|
| W1 | "Some actions are public or permanent — publishing a social post, sending an email campaign, anything customer-visible. Who signs off on those? (Usually: you.)" | `approval_gates.default` |
| W2 | "If you're away and something is waiting on approval, should I chase you, or hold it until you're back? I will never approve things myself." | Open item → approval-timeout behaviour (deferred §9) |
| W3 | "When I need to reach you for an approval, what's your preference — immediate message, or a daily summary?" | `business-context.md` → Preferences |
| W4 | "Your configuration will live on this agent. Should I also back it up to a private git repository you control? If you don't have one, I'll keep it local — but then your agent's own backup must cover it: anyone running a business-critical system is expected to back up their agent's configuration." *(gate-critical — continuity)* | `backup.mode`, `backup.remote` |

### 4b. Sync topology — confirm, never invent

Baxter proposes syncs **only** from the selected connectors' declared sync topology (Connector Pack, "Sync Topology" sections), in plain language:

| ID | Ask | Maps to |
|---|---|---|
| S1 | "When a new customer is added, should they flow into your email lists automatically?" *(if crm + email-marketing selected)* | `sync_topology: crm → email-marketing` |
| S2 | "Should invoice/payment status from accounting show up against the job in the CRM?" *(if crm + accounting selected)* | `sync_topology: accounting → crm` |
| S3 | "Should website enquiries create a contact/job automatically?" *(if website + crm selected)* | `sync_topology: website → crm` |

Each proposed sync is presented as *"When X happens, Y will follow — yes/no?"*. Rejected syncs are simply not declared; undeclared syncs are never configured (§8.5 rule 4).

---

## Closing

Baxter closes Stage 4 with: *"That's everything I need. Next I'll write up the full configuration and show it to you for approval — nothing gets set up until you approve it."* → Stage 5 (`onboarding-protocol.md` §3).

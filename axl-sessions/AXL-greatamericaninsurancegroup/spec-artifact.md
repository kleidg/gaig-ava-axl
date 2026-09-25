# AXL Spec Artifact — Summit Billing: Payment Status Inquiry

*Save as `spec-artifact.md`. Artifact 1 of 3: **spec artifact** → run artifact → readout artifact.*

*Pre-Session POV Deck · September 24, 2026 · one intent per prototype · all assets built new.*
*Unmarked = confirmed by the customer · `_(researched)_` = from their public site · `_(assumed)_` = our inference.*

> **This spec promotes Use Case 1 (Recommended) from `pov-deck.md` / `pov-deck.pdf`.** It has not
> yet been walked with the customer — see **Confirm these** below. Use cases 2 (Documentation &
> submission status) and 3 (Denial & appeal guidance) are parked as later phases, not discarded;
> see the deck for their detail.

## 1. What we're building

- **In one sentence:** An AI-powered voice experience that, once the IVR has validated a caller's Tax ID and Bill ID, answers "has my payment gone out?" with a specific, dated status — no hold, no transfer.
- **Not today (parked):** Documentation/submission status checks and denial & appeal guidance _(customer-provided — both are named call drivers in the Summit brief, staged as use cases 2 and 3 for a later phase)_. Intent classification/routing across all five original call drivers is also out of scope for this first prototype — the IVR pre-authenticates and hands off a single intent.

## 2. What good looks like

- **Great outcome for the customer:** The caller (a provider or injured employee) learns the exact payment status, date and amount for their bill without waiting on hold or repeating identifiers already given to the IVR.
- **The moment that proves it works:** The assistant reads back a specific, dated payment answer within seconds of hand-off, purely from the Tax ID + Bill ID the IVR already validated — then escalates cleanly, with that context intact, if the caller wants to dispute it. `_(assumed)_`

## 3. The assistant

- **Name:** _(assumed)_ — to be named with Summit's brand voice in Job 2 (candidates in the spirit of "Know the people who know workers' comp®": a first name, not a corporate label).
- **Channel:** Voice `_(researched)_` — the brief's caller journey is IVR → AI-powered voice experience → live representative, and Summit's own brand promise ("you'll always be able to speak to a real human being") is voice-first.
- **Tone:** Warm, plainspoken, specialist-not-corporate `_(researched)_` — matches Summit's "More access. More know-how. More commitment." positioning and its "real human being" promise.
- **Should never:** Collect a caller's SSN, PIN, OTP, full card/account number, or any secret in-conversation — identity is already established by the IVR before hand-off. `_(assumed)_`

## 4. The conversation

- **Happy path:**
  1. The IVR has already collected and validated the caller's Tax ID and Bill ID and hands off to the AI experience with that context. `_(researched — from the brief's caller journey)_`
  2. The assistant confirms the bill it's looking at (provider, date of service, amount) so the caller knows it has the right one. `_(assumed)_`
  3. It calls `get_bill_payment_status` and reads back the status (e.g. "processed 9/18, payment issued 9/20 for $412.50"). `_(assumed)_`
  4. If the caller asks a follow-up ("what's an EOB?"), it answers from Billing Knowledge; otherwise it offers "anything else?" and closes cleanly. `_(assumed)_`
- **Hands off to a person when:** The caller wants to dispute the amount, the lookup fails or returns no match, or the answer doesn't resolve the caller's question. Hands off to a Medical Bill Reviewer with the Bill ID, claim number and payment status already attached. `_(assumed)_`
- **Confirms or verifies first:** Not needed — identity is already verified by the IVR (Tax ID + Bill ID) before hand-off; this prototype makes no writes, so no confirmation gate applies. `_(assumed)_`

## 5. What it connects to

| Tool | What it does | Read / write | Build as |
|---|---|---|---|
| `get_bill_payment_status` | Looks up payment status, date and amount for a validated Bill ID | Read | New Data Action · mock |
| `get_claim_context` | Pulls the linked claim number, provider and date of service to confirm the right bill | Read | New Data Action · mock |

- **Knowledge:** Yes — covers billing terminology (EOB, fee-schedule adjustment, remittance codes) so the assistant can explain terms a caller doesn't recognize. `_(assumed)_`
- **When it finds nothing, it says:** "I'm not able to find a bill matching that information — let me connect you with a Medical Bill Reviewer who can look into it." `_(assumed)_`

## Confirm these

*The 3–5 assumptions that would change the build if they're wrong.*

1. Payment status is confirmed as the single highest-volume driver among the five named in the brief, worth prototyping first — the brief lists it but doesn't rank volume.
2. The assistant's channel is voice, not a digital/chat surface, for this first prototype.
3. The system of record for payment/claim status is a single platform the mocked Data Actions can mirror — not multiple systems the caller's identifiers would need to be checked against.
4. Escalation on this use case routes to the same Medical Bill Reviewer team named in the brief, not a separate billing queue.
5. No confirmation/identity step is needed inside the assistant itself, since the IVR has already validated Tax ID + Bill ID before hand-off.

**Sources:** Summit Billing IVR / AI Automation Opportunity brief (provided directly, Sep 2026) · greatamericaninsurancegroup.com/about-us/business-operations/division/summit · greatamericaninsurancegroup.com (homepage) · Summit Medical Bill Reviewer job postings (greatinsurancejobs.com, emploive.com) — see `pov-deck.html` slide 6 for the full provenance grid.

---
*Everything below is appendix — it doesn't belong to the one-page spec.*

## Appendix — Action details

#### `get_bill_payment_status`
- **Inputs:** `tax_id` · string · required · source: prior step (IVR-validated) — `bill_id` · string · required · source: prior step (IVR-validated)
- **Outputs:** `status` · string (e.g. "Processed", "Payment Issued", "Pending Review") · example `"Payment Issued"` — `processed_date` · date · example `"2026-09-18"` — `payment_date` · date · example `"2026-09-20"` — `amount` · currency · example `"$412.50"`
- **Logic (high level):** Look up the bill by `bill_id` (validated against `tax_id` for ownership), return its current status in the billing workflow and, if paid, the issue date and amount. `_(assumed)_`
- **Sample data & dependencies:** `{"bill_id": "SB-88213", "status": "Payment Issued", "processed_date": "2026-09-18", "payment_date": "2026-09-20", "amount": "$412.50"}` — no dependency on another tool.

#### `get_claim_context`
- **Inputs:** `bill_id` · string · required · source: ToolInput (reused from `get_bill_payment_status`)
- **Outputs:** `claim_number` · string · example `"WC-2026-004471"` — `provider_name` · string · example `"Lakeland Family Medical"` — `date_of_service` · date · example `"2026-08-30"`
- **Logic (high level):** Resolve the claim and provider linked to the bill so the assistant can confirm the right one back to the caller before reading status. `_(assumed)_`
- **Sample data & dependencies:** `{"claim_number": "WC-2026-004471", "provider_name": "Lakeland Family Medical", "date_of_service": "2026-08-30"}` — depends on `bill_id` from `get_bill_payment_status`'s input.

## Internal setup *(facilitator only — not shared with the customer)*

- **Customer / account:** Great American Insurance Group — Summit (workers' compensation division)
- **Target MCP namespace (deploy destination):** TBD
- **AVA build name:** `AXL - Insurance - Billing Payment Status`
- **Genesys Cloud org / region:** TBD
- **Environment ready:** Function Data Actions ☐ · AI tokens ☐ · ElevenLabs ☐ · Deepgram ☐
- **Attendees:** TBD
- **Salesforce Opp:** TBD
- **Business case (optional):** Reduce inbound billing call volume currently fielded by Medical Bill Reviewers who also own fee-schedule audits and dispute handling; phased rollout starting with highest-volume, most repetitive inquiries.

### Build notes *(publish / VersionDefinition — for ava-build)*

- Shared inputs across tools: `tax_id` and `bill_id` are collected once (by the IVR, upstream of the AVA) and reused as `ToolInput` on both tools — never re-collected inside the assistant.
- Write tools ↔ §4 confirmation gates: None — this prototype is read-only.
- Secrets: None in-assistant — identity already verified by the IVR before hand-off.

---

*Maps to MCP build: §1–3 → `create_ava` name + `role` & voice · §4 → `instructions` & `events` · §5 → `tools` & `types` · appendix → `types` & mock logic · then `create_version` + `publish_version` (TestReady).*

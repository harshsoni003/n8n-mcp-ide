# N8N Note Taking Guidelines

> **Purpose**: Ensure every flow is understandable at a glance — by humans and AI agents alike.  
> **Rule of thumb**: A new developer (or an AI) should understand what a flow does within 10–20 seconds of opening it.

---

## 1. Flow Overview Summary Note (Sticky Note)

Every n8n flow **must** contain a **Summary sticky note** placed visibly at the top-left of the canvas.

This note should let anyone understand the flow's purpose, inputs, outputs, and setup requirements without opening a single node.

---

### Required Sections

| Section | What to Write |
|---------|--------------|
| **Name** | The workflow's full name (ensures this isn't just a copy-pasted note) |
| **Purpose** | Why this flow exists — the business reason, not just the technical steps |
| **Trigger** | What starts this flow (webhook, schedule, manual, subflow call, etc.) |
| **Connected Builds** | Links to parent/child workflows if this is part of a larger system |
| **Input Schema** | If used as a subflow — document what input fields are expected |
| **Dependencies** | All external services, APIs, and credentials used |
| **Setup Instructions** | Step-by-step setup for any third-party configuration needed (webhooks, API keys, etc.) |
| **Outputs** | What happens on successful completion and how that output is used downstream |

---

### Summary Note Template

Use this as your starting point for every new workflow:

```
DOC: Workflow Overview

Name: [Full workflow name — must match the actual workflow name]

Purpose: [Why does this flow exist? What business problem does it solve?]

Trigger: [Manual / Webhook / Schedule / Subflow — describe the trigger]

Connections:
- Parent: [Workflow name if triggered by another flow, or "None"]
- Children: [Subflows called by this flow, or "None"]

Input Schema (for subflows):
- FieldName (Type): Description
- FieldName (Type): Description

Dependencies:
- [Service name] — [Credential name, e.g. 4am-HubSpot-APIkey-PROD]
- [Service name] — [Credential name]

Setup Instructions:
1. [Step — e.g. Create API credential in n8n]
2. [Step — e.g. Configure webhook URL in VAPI dashboard]
3. [Step — e.g. Set ProjectID in "Set Parameters" node]

DOC: Outputs & Next Steps

Outputs: [What is produced when this flow succeeds?]

Output Structure:
- FieldName: Description
- FieldName: Description

Next Steps / Downstream Use:
- [How is the output consumed? Who/what uses it next?]
```

---

### Real-World Example

```
DOC: Workflow Overview

Name: PROD - Webhook - Sync New Leads to HubSpot

Purpose: Receives new lead submissions from the website contact form via webhook,
enriches the contact with company data, and creates a contact in HubSpot.
Outcome feeds the sales team's outreach pipeline.

Trigger: Webhook — POST /webhook/new-lead
         Sent by the website form on submission.

Connections:
- Parent: None (entry point)
- Children: Subflow - Enrich Contact Data, Subflow - Send Slack Notification

Dependencies:
- HubSpot — 4am-HubSpot-OAuth-PROD
- OpenAI — 4am-OpenAI-APIkey-PROD (used in enrichment subflow)

Setup Instructions:
1. Create HubSpot OAuth credential named "4am-HubSpot-OAuth-PROD"
2. Copy the webhook URL from "Trigger Webhook" node and register it
   in the website form's submission handler
3. Ensure "Enrich Contact Data" subflow is active before activating this flow

DOC: Outputs & Next Steps

Outputs: New HubSpot Contact created with enriched company data.

Output Structure:
- ContactID: HubSpot contact ID for the created record
- EnrichmentStatus: success | skipped | failed

Next Steps:
- HubSpot triggers a sequence for new contacts automatically
- Slack notification sent to #sales-leads channel
```

---

## 2. Notes Inside Nodes

Most nodes — especially any node doing something non-obvious — should have a note filled in the **Notes** field (found in the node's **Settings** tab).

**Always enable**: `Display Note in Flow?` → **ON**

This ensures the note is visible on the canvas without needing to open the node, which helps:
- Humans skim the flow quickly
- AI agents interpret the flow from the JSON export (sticky notes are not linked to nodes in JSON)

---

### Core Principle: "Why" > "What"

The node's name already tells you *what* it does (`Send Email`, `Fetch Leads`).  
The note should tell you **why** — the intent, context, or constraint that isn't obvious from the name alone.

| ❌ Bad Note (explains "what") | ✅ Good Note (explains "why") |
|------------------------------|------------------------------|
| "Sends an email" | "Using Gmail to send via TransmitSMS. Single sync flow handles both email and SMS inputs." |
| "Filters records" | "Drops contacts without a valid email before HubSpot call. HubSpot rejects null emails with a 400." |
| "HTTP request to API" | "Calls VAPI to initiate an outbound call. Limit: 10 concurrent calls. Docs: vapi.ai/docs/calls" |
| "Sets fields" | "Maps raw webhook payload to internal schema. Phone normalized to E.164 for Twilio downstream." |

---

### What to Include in a Node Note

Use this mental checklist when writing a node note:

- **Intent**: Why does this node exist in this flow? What decision led to using it?
- **API limits**: Rate limits, concurrency limits, quota constraints
- **Docs links**: If the API has relevant documentation, link it
- **Known issues**: Edge cases, gotchas, or known failure modes
- **Dependencies**: If this node relies on a specific field being present from a previous node

---

### Which Nodes Need a Note?

| Node Type | Note Required? |
|-----------|---------------|
| Trigger nodes (Webhook, Schedule) | ✅ Yes — describe what sends the trigger and expected payload |
| API / external service nodes | ✅ Yes — include why, any limits, and relevant docs |
| IF / Switch (non-obvious conditions) | ✅ Yes — explain the business rule being evaluated |
| Code nodes | ✅ Always — this is especially critical for code nodes |
| Set / Edit Fields (non-trivial mapping) | ✅ Yes — explain the mapping logic |
| Simple filter / limit nodes | ⬜ Optional — add only if the filter condition needs context |
| Merge nodes | ⬜ Optional — add if the merge strategy is non-obvious |

---

### Node Note Examples

**Webhook Trigger**
```
Entry point for new lead submissions from the website contact form.
Expected payload: { Name, Email, Phone, CompanyName, Message }
Phone must be provided; Email is optional but preferred.
```

**HTTP Request (external API)**
```
Calls TransmitSMS to send SMS via Gmail SMTP relay.
Using Gmail node avoids a separate SMS credential setup —
one sync flow handles both email and SMS sources.
Limit: 4 SMS per minute. Docs: transmitsms.com/api
```

**IF Node**
```
Checks if the incoming contact already exists in HubSpot
before creating a new record. Prevents duplicate contacts
in the CRM — HubSpot does not deduplicate automatically on this endpoint.
```

**Code Node**
```
Normalizes phone number to E.164 format (+countrycode) required by Twilio.
Input: raw string from form (may include spaces, dashes, parentheses)
Output: { PhoneE164: "+61412345678" }
edge case: Australian numbers without country code get +61 prefix added.
```

---

## 3. Quick-Reference Cheat Sheet

| What | Rule |
|------|------|
| Summary sticky note | Required in every workflow, top-left of canvas |
| Summary note name field | Must match the actual workflow name |
| Node Notes | Required on most nodes; always "Display Note in Flow?" → ON |
| Note principle | "Why" > "What" — context, not repetition |
| Code nodes | Always include a note — no exceptions |
| API limit notes | Always include known rate/concurrency limits |
| Docs links in notes | Include when API docs are relevant to the node's behavior |

---

## 4. Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-02-28 | Initial document created | Antigravity |

> 💬 **Suggest changes** by adding a comment below or editing this file directly.

---
description: N8N note taking — enforced rules for all workflow create/update operations
---

# N8N Note Taking — Agent Rule

> **MANDATORY**: Apply these rules to EVERY n8n workflow you create or update.
> Full reference: `docs/N8N_NOTE_TAKING.md`

---

## Pre-Flight Checklist

Before calling `n8n_create_workflow`, `n8n_update_full_workflow`, or `n8n_update_partial_workflow`, verify:

- [ ] A **Summary sticky note** exists at the top of the workflow with all required sections filled
- [ ] Every API / external service node has a **note** in its `notes` field
- [ ] Every non-trivial node has `notesInFlow: true` set
- [ ] Code nodes **always** have a note — no exceptions
- [ ] Node notes explain **why**, not just what (the name already says what)

---

## 1. Summary Sticky Note (Required in Every Workflow)

Every workflow must include a sticky note (`nodes-base.stickyNote`) placed at the top of the canvas.

### Required Content

```
DOC: Workflow Overview

Name: [Exact workflow name]

Purpose: [Business reason this flow exists]

Trigger: [What starts this flow — Webhook / Schedule / Manual / Subflow]

Connections:
- Parent: [Parent workflow name, or "None"]
- Children: [Subflow names called, or "None"]

Input Schema (subflows only):
- FieldName (Type): Description

Dependencies:
- [Service] — [Credential name following <Client>-<Service>-<Purpose>-<Env>]

Setup Instructions:
1. [Step]
2. [Step]

DOC: Outputs & Next Steps

Outputs: [What is produced on successful completion]

Output Structure:
- FieldName: Description

Next Steps:
- [How the output is consumed downstream]
```

### Sticky Note Node Example (n8n JSON)

```json
{
  "id": "sticky-overview",
  "name": "DOC: Workflow Overview",
  "type": "n8n-nodes-base.stickyNote",
  "typeVersion": 1,
  "position": [-200, -300],
  "parameters": {
    "content": "## DOC: Workflow Overview\n\n**Name:** DEV - Harsh - Webhook - Sync Leads\n\n**Purpose:** Receives lead form submissions and creates contacts in HubSpot...\n\n**Trigger:** Webhook — POST /webhook/new-lead\n\n**Dependencies:**\n- HubSpot — 4am-HubSpot-OAuth-PROD\n\n**Setup Instructions:**\n1. Create HubSpot credential\n2. Register webhook URL in form handler\n\n## DOC: Outputs\n\n**Outputs:** HubSpot Contact created\n- ContactID: HubSpot contact ID",
    "height": 500,
    "width": 400
  }
}
```

---

## 2. Node Notes (Required on Most Nodes)

Set `notes` and `notesInFlow: true` on every qualifying node.

### Which Nodes MUST Have a Note

| Node Type | Required |
|-----------|---------|
| API / external service nodes (HTTP, CRM, email, SMS, AI) | ✅ Always |
| Code nodes | ✅ Always — no exceptions |
| Webhook / trigger nodes | ✅ Always — document expected payload |
| IF / Switch with non-obvious conditions | ✅ Yes |
| Set / Edit Fields with non-trivial mapping | ✅ Yes |
| Simple filter or limit nodes | ⬜ Optional |
| Merge nodes | ⬜ Optional unless strategy is non-obvious |

### Node Note Properties (n8n JSON)

```json
{
  "name": "Send Slack Alert",
  "type": "n8n-nodes-base.slack",
  "notes": "Notifies #ops-alerts when a lead fails HubSpot sync. Message includes WorkflowName and ErrorMessage from the error context. Slack rate limit: 1 msg/sec per channel.",
  "notesInFlow": true
}
```

---

## 3. Core Principle: "Why" > "What"

The node's **name** already tells you what it does.  
The **note** must explain the *why* — intent, context, constraints.

### Writing Good Notes

**Include**:
- Why this node exists at this point in the flow
- API rate limits or concurrency constraints
- Links to relevant API documentation
- Known edge cases or failure modes
- Input/output field expectations if not obvious from the node name

**Do NOT write**:
- "This node sends an email" (the name `Send Email` already says that)
- "HTTP request" (meaningless)
- Restatements of the node type

### Note Templates by Node Type

**Webhook / Trigger**
```
Entry point for [describe what sends the trigger].
Expected payload fields: { Field1 (Type), Field2 (Type) }
[Any notes on authentication, IP allowlisting, etc.]
```

**API / HTTP Request node**
```
Calls [Service] to [action].
[Reason why this service was chosen / any architectural decision]
Rate limit: [X requests per Y]. Docs: [URL if relevant]
```

**IF / Switch**
```
[Business rule being evaluated — not just the technical condition]
[Why is this branching point necessary?]
[What happens on each branch?]
```

**Code node**
```
[What transformation is being performed and WHY it can't be done with native nodes]
Input: [field(s) consumed]
Output: [field(s) produced]
Edge cases: [null inputs, type coercion, etc.]
```

**Set / Edit Fields**
```
[Why are these fields being mapped or renamed here?]
[If this is preparing data for a downstream node, name that node]
```

---

## 4. Combining All Four Rules

Every workflow submission must satisfy all four agent rules:

| Rule | Mandatory | Key Checks |
|------|-----------|-----------|
| `/n8n_naming_conventions` | ✅ | Workflow name, node names, PascalCase attributes, credential names |
| `/n8n_error_handling` | ✅ | retryOnFail on API nodes, errorWorkflow in settings, design-level validation |
| `/n8n_note_taking` (this file) | ✅ | Summary sticky note, node notes with "why", notesInFlow: true |
| `/n8n_best_practices` | ⚠️ Soft | Workflow size, Code node usage, readability |

---

## 5. Enforcement Examples

### ✅ Correct — Node with proper note
```json
{
  "name": "Create HubSpot Contact",
  "type": "n8n-nodes-base.hubspot",
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 1000,
  "onError": "stopWorkflow",
  "notes": "Creates a new contact in HubSpot after lead enrichment. Uses upsert strategy keyed on Email — will update existing contact if email matches. HubSpot API limit: 100 req/10s.",
  "notesInFlow": true
}
```

### ❌ Wrong — Node without note
```json
{
  "name": "HTTP Request",
  "type": "n8n-nodes-base.httpRequest"
  // ❌ No notes
  // ❌ notesInFlow not set
  // ❌ Default node name
}
```

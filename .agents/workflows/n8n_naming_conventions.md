---
description: N8N naming conventions — enforced rules for all workflow create/update operations
---

# N8N Naming Conventions — Agent Rule

> **MANDATORY**: Apply these rules to EVERY n8n workflow, node, attribute, and credential you create or update.
> Full reference: `docs/N8N_NAMING_CONVENTIONS.md`

---

## Pre-Flight Checklist

Before calling `n8n_create_workflow`, `n8n_update_full_workflow`, or `n8n_update_partial_workflow`, verify:

- [ ] **Workflow name** follows the correct pattern for its stage
- [ ] **Every node** has a Verb + Object name (no default names like "HTTP Request", "Code", "Set")
- [ ] **Every attribute** key is PascalCase and ≤ 4 words
- [ ] **Credential names** used follow `<Client>-<Service>-<Purpose>-<Env>`

---

## 1. Workflow Names

```
DEV stage:  DEV - <Who> - <Trigger|Subflow> - <Point of the flow>
PROD stage: PROD - <Trigger|Subflow> - <Point of the flow>
```

**Rules**:
- `<Who>` = **the first name of the developer currently building or requesting this workflow**.
- If the user's name is known from context (e.g. they introduced themselves, it's in the conversation, or it exists in a previous workflow they own), use that name.
- If the user's name is **not known**, **ask before creating the workflow**: *"What name should I use in the DEV prefix for this workflow?"*
- Switch to `PROD` only after explicit project manager approval (no `<Who>` in PROD names).
- If handing over to another developer, rename with **their** first name or copy the workflow.

**Examples**:
```
DEV - Harsh - Webhook - Sync Leads to HubSpot      ← Harsh is building it
DEV - Pratik - Schedule - Send Weekly Report        ← Pratik is building it
DEV - Maria - WhatsApp - AI Echo Bot               ← Maria is building it
PROD - Webhook - Sync Leads to HubSpot             ← No person name in PROD
PROD - Subflow - Enrich Contact Data
```

---

## 2. Node Names

```
<Verb> <Object>   →   e.g. "Fetch Leads", "Send Email", "Filter New Records"
```

**Rules**:
- NEVER leave a node with its default type name (`HTTP Request`, `Code`, `Set`, `IF`, etc.).
- Use Title Case.
- Be specific: `Update Deal Stage` not just `Update`.

**Approved verbs by category**:
| Action type | Verbs |
|-------------|-------|
| Read | Fetch, Get, Read, Load, List |
| Write | Create, Update, Upsert, Delete, Insert |
| Transform | Format, Map, Merge, Filter, Split, Sort, Clean |
| Communicate | Send, Notify, Post, Publish |
| Control | Wait, Check, Route, Loop, Retry, Trigger |

**Good examples**: `Fetch Deals`, `Map Contact Fields`, `Route by Status`, `Send Slack Alert`  
**Bad examples**: `HTTP Request`, `Code`, `Set`, `node`, `email`

---

## 3. Attribute Names (in Code / JSON / Set nodes)

```
PascalCase, ≤ 4 words, answer "what is this?"
```

**Rules**:
- Every key you define in a `Set`, `Code`, or `Function` node must use PascalCase.
- Abbreviations stay fully uppercase: `ID`, `URL`, `API`, `CRM`.
- Match this convention in downstream tools: Airtable fields, ClickUp fields, HubSpot properties, Salesforce fields.

**Examples**:
```json
{
  "DealID": 12345,
  "CustomerID": 987,
  "CreatedAt": "2025-06-02T09:15:00Z",
  "AssignedToUserID": 42,
  "LeadSourceURL": "https://example.com",
  "IsEmailVerified": true
}
```

**Exception**: When mapping TO an external API that has its own convention (GraphQL, REST schemas), follow the target convention for that mapping node only.

---

## 4. Credential Names

```
<Client>-<Service>-<Purpose>-<Env>
```

| Part | Values |
|------|--------|
| Client | Client/project name, e.g. `4am`, `Acme`, `Internal` |
| Service | Integration name, e.g. `HubSpot`, `OpenAI`, `Gorgias` |
| Purpose | `APIkey`, `OAuth`, `Webhook`, `PersonalToken` |
| Env | `PROD`, `DEV`, `STAGING` |

**Examples**:
```
4am-Gorgias-APIkey-PROD
Acme-HubSpot-OAuth-DEV
Internal-OpenAI-APIkey-PROD
```

When creating a workflow that requires credentials, always note the expected credential name in comments so the user knows what to create.

---

## Enforcement Examples

### Creating a new workflow
```
✅ CORRECT:
  name: "DEV - Harsh - Webhook - Sync New Leads"
  nodes:
    - name: "Trigger Webhook"         ← Verb + Object ✅
    - name: "Fetch Lead Details"      ← Verb + Object ✅
    - name: "Map to HubSpot Fields"   ← Verb + Object ✅
    - name: "Create HubSpot Contact"  ← Verb + Object ✅
    - name: "Send Slack Alert"        ← Verb + Object ✅

❌ WRONG:
  name: "My New Workflow"
  nodes:
    - name: "Webhook"
    - name: "HTTP Request"
    - name: "Code"
    - name: "Set"
```

### Attribute naming in a Code node
```javascript
// ✅ CORRECT
const output = {
  DealID: $input.item.json.id,
  CustomerName: $input.item.json.name,
  CreatedAt: new Date().toISOString(),
};

// ❌ WRONG
const output = {
  deal_id: $input.item.json.id,
  customerName: $input.item.json.name,
  created_at: new Date().toISOString(),
};
```

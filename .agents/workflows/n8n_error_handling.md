---
description: N8N error handling — enforced rules for all workflow create/update operations
---

# N8N Error Handling — Agent Rule

> **MANDATORY**: Apply these rules to EVERY n8n workflow you create or update.
> Full reference: `docs/N8N_ERROR_HANDLING.md`

---

## Pre-Flight Checklist

Before calling `n8n_create_workflow`, `n8n_update_full_workflow`, or `n8n_update_partial_workflow`, verify:

- [ ] Every **API / external service node** has `retryOnFail: true` configured
- [ ] `maxTries`, `waitBetweenTries`, and `onError` are set appropriately per node
- [ ] The workflow's `settings.errorWorkflow` points to `<Client> - Error Workflow`
- [ ] The Error Workflow exists and starts with an `Error Trigger` node
- [ ] Flow design handles missing data, type issues, and invalid values **before** they reach API nodes

---

## 1. Which Nodes Get Retry on Fail?

### ✅ MUST have retryOnFail — API / External Service Nodes
- `nodes-base.httpRequest`
- `nodes-base.hubspot`, `nodes-base.salesforce`, `nodes-base.pipedrive`
- `nodes-base.sendGrid`, `nodes-base.twilioApi`, `nodes-base.gmail`
- `nodes-base.airtable`, `nodes-base.googleSheets`, `nodes-base.supabase`
- `nodes-base.slack`
- `nodes-langchain.openAi`, `nodes-langchain.anthropic`, `nodes-langchain.agent`
- Any node making an outbound network or API call

### ❌ Do NOT add retryOnFail — Internal / Flow Nodes
- `nodes-base.filter`, `nodes-base.limit`, `nodes-base.splitOut`, `nodes-base.aggregate`
- `nodes-base.if`, `nodes-base.switch`, `nodes-base.merge`, `nodes-base.splitInBatches`
- `nodes-base.set`, `nodes-base.removeDuplicates`
- Trigger nodes: `nodes-base.webhook`, `nodes-base.scheduleTrigger`, `nodes-base.manualTrigger`

> If a non-API node fails, fix the flow design — don't mask it with retries.

---

## 2. Retry on Fail — Node Configuration

Set these properties on every qualifying node:

```json
{
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 1000,
  "onError": "stopWorkflow"
}
```

### When to Deviate from Defaults

| Scenario | Adjustment |
|----------|-----------|
| Rate-limited API (HubSpot free, OpenAI tier) | `waitBetweenTries: 5000` or `10000` |
| Critical, low-frequency operation | `maxTries: 5` |
| Non-idempotent action (send email, SMS) | `maxTries: 1` — handle deduplication separately |
| Idempotent write (upsert, update) | Keep defaults |

---

## 3. Error Workflow — Workflow Settings

Every workflow you create MUST have an error workflow linked. Set it in the workflow's `settings` object:

```json
{
  "settings": {
    "errorWorkflow": "<workflow-id-of-client-error-workflow>"
  }
}
```

### Error Workflow Naming Convention
```
<Client name> - Error Workflow
```

Examples:
```
Acme Corp - Error Workflow
4am - Error Workflow
Internal - Error Workflow
```

### Error Workflow — Required Structure

When creating or verifying an error workflow, it MUST:

1. **Start with an `Error Trigger` node** (type: `nodes-base.errorTrigger`) — mandatory
2. **Log the error** to at least one persistent store (Airtable, Supabase, Google Sheets)
3. Optionally notify via Slack or email (add later if client hasn't decided yet)

#### Minimum Required Nodes
```
Error Trigger → Log Error to [Airtable | Supabase | Google Sheets]
```

#### Required Fields to Log
```javascript
{
  WorkflowName: "{{ $workflow.name }}",
  NodeName:     "{{ $execution.error.node.name }}",
  ErrorMessage: "{{ $execution.error.message }}",
  ErrorTime:    "{{ $now.toISO() }}",
  ExecutionID:  "{{ $execution.id }}",
  WorkflowID:   "{{ $workflow.id }}"
}
```

---

## 4. Design-Level Error Prevention

These must be applied **before** data reaches any API node:

### Missing Data
- Use `IF` / `Switch` nodes to check required fields exist
- Use `Set` nodes to apply default values for optional fields
- Use `Filter` nodes to drop incomplete records early

### Type / Format Issues
- Coerce data types explicitly (string → number, string → array)
- Trim whitespace from string inputs

### Invalid Values
- Validate emails contain `@` before email API calls
- Validate phone numbers match expected format before SMS
- Check IDs are non-null before CRM lookups
- Ensure dates are parseable before date-sensitive API calls

---

## 5. Enforcement Examples

### Creating a workflow with error handling
```json
{
  "name": "DEV - Harsh - Webhook - Sync Leads",
  "settings": {
    "errorWorkflow": "wf_acme_error_workflow_id"
  },
  "nodes": [
    {
      "name": "Trigger Webhook",
      "type": "nodes-base.webhook"
      // No retryOnFail — trigger node ✅
    },
    {
      "name": "Validate Email Field",
      "type": "nodes-base.if"
      // No retryOnFail — flow control node ✅
    },
    {
      "name": "Create HubSpot Contact",
      "type": "nodes-base.hubspot",
      "retryOnFail": true,      // ✅ API node
      "maxTries": 3,
      "waitBetweenTries": 1000,
      "onError": "stopWorkflow"
    },
    {
      "name": "Send Slack Alert",
      "type": "nodes-base.slack",
      "retryOnFail": true,      // ✅ API node
      "maxTries": 3,
      "waitBetweenTries": 1000,
      "onError": "stopWorkflow"
    }
  ]
}
```

### ❌ What NOT to do
```json
{
  "name": "My Workflow",          // ❌ No DEV/PROD prefix
  "settings": {},                 // ❌ No errorWorkflow set
  "nodes": [
    {
      "name": "HTTP Request",     // ❌ Default node name
      "type": "nodes-base.httpRequest"
      // ❌ No retryOnFail
    },
    {
      "name": "Filter",           // ❌ Vague node name
      "type": "nodes-base.filter",
      "retryOnFail": true         // ❌ retryOnFail on a transform node
    }
  ]
}
```

---

## 6. Combined Pre-Submit Checklist

Before submitting any n8n workflow call, confirm ALL of the following:

**Naming** (from `/n8n_naming_conventions`):
- [ ] Workflow name: `DEV - <Who> - <Trigger> - <Purpose>` or `PROD - <Trigger> - <Purpose>`
- [ ] Every node name: `<Verb> <Object>` — no defaults left
- [ ] Attribute keys: PascalCase ≤ 4 words

**Error Handling** (this rule):
- [ ] All API nodes have `retryOnFail: true`
- [ ] `maxTries`, `waitBetweenTries`, `onError` are set per node
- [ ] `settings.errorWorkflow` is set to the correct client error workflow ID
- [ ] Error Workflow exists, starts with `Error Trigger`, and logs required fields
- [ ] Flow design validates and sanitizes data before API nodes

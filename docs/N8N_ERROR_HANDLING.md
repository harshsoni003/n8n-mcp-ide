# N8N Error Handling Guidelines

> **Purpose**: Ensure every production flow is resilient, recoverable, and observable.  
> **Rule of thumb**: Make errors impossible by design first; handle the unavoidable ones gracefully second.

---

## General Philosophy

Error handling in n8n has two layers:

| Layer | Goal |
|-------|------|
| **Prevent errors** | Design flows so missing data, type mismatches, and invalid values never reach API nodes |
| **Recover from errors** | When external services fail, use Retry on Fail + an Error Workflow to catch and respond |

---

## 1. Which Nodes Need Error Handling?

### ✅ Apply to — API / External Service Nodes

Any node that calls an external service must have **Retry on Fail** enabled:

- HTTP Request
- CRM nodes (HubSpot, Salesforce, Pipedrive…)
- Email / SMS nodes (SendGrid, Twilio…)
- Storage nodes (Airtable, Google Sheets, Supabase…)
- Messaging nodes (Slack, Gmail…)
- AI / LLM nodes (OpenAI, Anthropic…)
- Any other node that makes an outbound network call

### ❌ Do NOT apply to — Internal / Flow Nodes

These should be **error-free by design** — if they fail, fix the flow logic:

| Category | Examples |
|----------|---------|
| Data transformation | Filter, Limit, Split Out, Aggregate, Sort |
| Flow control | IF, Switch, Merge, Loop Over Items, Wait |
| Data manipulation | Set, Edit Fields, Remove Duplicates |
| Trigger nodes | Webhook, Schedule, Manual Trigger |

> If a transformation or flow node is failing, the root cause is almost always **bad input data** or **incorrect flow design** — fix that, don't mask it with retries.

---

## 2. Preventing Errors by Design

Before adding retries, make the flow robust so recoverable errors don't occur in the first place:

### 2.1 Handle Missing Data
```
✅ Use IF nodes to check if a field exists before using it
✅ Use Set nodes to provide default values for optional fields
✅ Use Filter nodes to drop records without required fields early in the flow
```

### 2.2 Fix Formatting Issues
```
✅ Validate that fields expected as arrays are arrays (use Split Out or toJsonString())
✅ Coerce strings to numbers/dates explicitly, don't rely on implicit conversion
✅ Trim whitespace from string inputs before passing to APIs
```

### 2.3 Validate Data Values
```
✅ Verify emails contain "@" before sending to email APIs
✅ Verify phone numbers match expected format before SMS
✅ Check that IDs are non-null and non-zero before lookup calls
✅ Confirm date strings are parseable before passing to date-sensitive APIs
```

---

## 3. Retry on Fail

### How to Enable (per node)
1. Open the node
2. Click the **Settings** tab
3. Toggle **Retry On Fail** → ON
4. Configure **Max. Tries**, **Wait Between Tries**, and **On Error**

### Default Settings (sufficient for most builds)

| Setting | Default Value | Notes |
|---------|--------------|-------|
| Max. Tries | `3` | Increase to `5` for critical, low-frequency calls |
| Wait Between Tries | `1000 ms` | Increase to `5000 ms` for rate-limited APIs |
| On Error | `Stop Workflow` | Triggers the Error Workflow for visibility |

> ⚠️ **There is no single universal setting.** You are responsible for tuning these values based on the API's rate limits, criticality of the operation, and expected failure modes.

### When to Deviate from Defaults

| Scenario | Adjustment |
|----------|-----------|
| Rate-limited API (e.g., HubSpot free tier) | Increase Wait Between Tries → `5000–10000 ms` |
| Critical single operation (payment, contract) | Increase Max. Tries → `5`, On Error → `Continue` + manual alert |
| Idempotent writes (safe to repeat) | Keep defaults |
| Non-idempotent writes (e.g., send email) | Max. Tries → `1`, handle duplicates separately |

### Visual Reference

The screenshot below shows the recommended default configuration:

- **Retry On Fail**: ON ✅
- **Max. Tries**: 3
- **Wait Between Tries**: 1000 ms
- **On Error**: Stop Workflow

---

## 4. Error Workflow

Every client must have a dedicated **Error Workflow**. This is the flow that n8n automatically triggers when any node in a linked workflow fails after all retries are exhausted.

### Naming Convention
```
<Client name> - Error Workflow
```

**Examples**:
```
Acme Corp - Error Workflow
4am - Error Workflow
Artificial Grass - Error Workflow
```

### How to Link (per workflow)
1. Open the workflow
2. Click the **three dots (⋮)** in the upper-right corner
3. Select **Settings**
4. Set **Error Workflow** → select `<Client name> - Error Workflow`

### Building the Error Workflow

Every error workflow must:

1. **Start with an `Error Trigger` node** — this is mandatory; n8n will not route errors to it otherwise.
2. **Log the error** — at minimum, capture: workflow name, node name, error message, and timestamp.
3. **Notify** — optionally send an alert via Slack, email, or another channel.

#### Minimum Required Structure
```
[Error Trigger] → [Log Error to Airtable/Supabase]
                → [Send Slack/Email Alert]   ← optional, add later
```

#### Recommended Fields to Log

| Field | n8n Expression |
|-------|---------------|
| WorkflowName | `{{ $workflow.name }}` |
| NodeName | `{{ $execution.error.node.name }}` |
| ErrorMessage | `{{ $execution.error.message }}` |
| ErrorTime | `{{ $now.toISO() }}` |
| ExecutionID | `{{ $execution.id }}` |
| WorkflowID | `{{ $workflow.id }}` |

### Minimum Viable Error Workflow (Bootstrap)

When setting up a new client, create the error workflow **immediately** with at minimum:
- ✅ Error Trigger node
- ✅ A logging node (Airtable row, Supabase insert, or Google Sheets append)

Notifications (Slack, email) can be added later once the client decides their preference.

---

## 5. End-to-End Checklist

Use this before activating any flow in production:

### Per Node (API nodes only)
- [ ] **Retry on Fail** is enabled
- [ ] **Max. Tries** is appropriate for this API's rate limits and criticality
- [ ] **Wait Between Tries** is appropriate
- [ ] **On Error** is set (typically `Stop Workflow`)

### Per Workflow
- [ ] **Error Workflow** is set in workflow settings
- [ ] The linked Error Workflow exists and starts with `Error Trigger`
- [ ] The Error Workflow logs at minimum: WorkflowName, NodeName, ErrorMessage, ErrorTime

### Per Flow Design
- [ ] Missing data is handled with defaults or early filters
- [ ] Type/format issues are resolved before reaching API nodes
- [ ] Invalid values (bad email, null ID) are caught and handled

---

## 6. Quick-Reference Cheat Sheet

| What | Rule |
|------|------|
| API nodes | Always enable Retry on Fail |
| Default Max. Tries | 3 |
| Default Wait | 1000 ms |
| Default On Error | Stop Workflow |
| Error Workflow name | `<Client> - Error Workflow` |
| Error Workflow trigger | Must start with `Error Trigger` node |
| Minimum error log fields | WorkflowName, NodeName, ErrorMessage, ErrorTime |
| Transformation/flow nodes | No retries — fix the logic instead |

---

## 7. Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-02-28 | Initial document created | Antigravity |

> 💬 **Suggest changes** by adding a comment below or editing this file directly.

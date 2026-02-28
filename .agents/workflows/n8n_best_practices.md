---
description: N8N best practices — architectural guidance applied as suggestions (not hard rules) during workflow create/update operations
---

# N8N Best Practices — Agent Guidance

> **NOTE**: These are **non-mandatory** guidelines — use judgment based on solution architecture and customer requirements. Apply as suggestions, not blockers.
> Full reference: `docs/N8N_BEST_PRACTICES.md`

---

## Design Review — Soft Checks

When creating or updating any workflow, mentally evaluate the following before submitting. These do not block creation but should be noted if violated:

---

## 1. Avoid the "One-Mega-Workflow" Anti-Pattern

### Warning Signs
- The workflow has **> 20–30 nodes**
- Multiple unrelated responsibilities live in a single canvas
- There are many Merge nodes connecting parallel, loosely-related tracks

### What to Do Instead

**Prefer subflows** — decompose into focused workflows connected via `Execute Workflow` nodes:

```
Main Workflow (orchestrator)
├── Subflow: <Verb> <Object>    e.g. "Fetch and Validate Leads"
├── Subflow: <Verb> <Object>    e.g. "Sync Contact to HubSpot"
└── Subflow: <Verb> <Object>    e.g. "Send Slack Notification"
```

**Benefits to communicate to user if suggesting a split**:
- Scoped execution logs — failures are easier to pinpoint
- Subflows can be reused across multiple parent workflows
- Each piece can be developed and tested independently

### One Item Per Execution

Prefer architectures where each workflow run processes **one item**:
- Easier to trace which item caused a failure
- Failed items can be individually retried
- Always log `ExecutionID` so the user can jump to the failing run

```json
{
  "ExecutionID": "={{ $execution.id }}",
  "WorkflowName": "={{ $workflow.name }}"
}
```

---

## 2. Avoid the "Code Node Vibe Coder" Anti-Pattern

### The 10-Second Rule

> If you cannot explain what a Code node does in 10 seconds, it should not be a Code node.

### Warning Signs: When NOT to Use a Code Node
- The code was AI-generated and is > 30–40 lines without clear comments
- The same result can be achieved with native n8n nodes
- The node is named `Code`, `Code1`, `Code2` (default names)
- There are multiple chained Code nodes in sequence

### Prefer Native Nodes

Before writing any code, check if these native nodes can solve it:

| Need | Native Node |
|------|------------|
| Set / rename fields | Edit Fields (Set) |
| Filter records | Filter |
| Conditional branch | IF / Switch |
| Merge arrays | Merge |
| Split array | Split Out |
| Remove duplicates | Remove Duplicates |
| Sort | Sort |
| Date formatting | Date & Time node or expressions |
| HTTP calls | HTTP Request |
| String ops | n8n expressions `{{ }}` |

### When a Code Node IS Acceptable

Use a Code node only when:
- The logic genuinely cannot be expressed with native nodes or expressions
- The code is short (< 30 lines) and focused on one transformation
- You can explain every line

### If You Use a Code Node — Do It Right

1. **Name it**: `<Verb> <Object>` — e.g., `Format Phone Number`, `Build API Payload`
2. **Add a Note** describing what it does in plain English
3. **Comment the code** for any non-obvious logic:

```javascript
// Normalize phone to E.164 format required by Twilio (+countrycode)
const digits = $input.item.json.Phone.replace(/\D/g, '');
return [{ json: { PhoneE164: `+${digits}` } }];
```

4. **Handle edge cases**: null inputs, empty arrays, unexpected types

---

## 3. General Readability Guidance

These apply to every workflow regardless of complexity:

- **Arrange nodes left-to-right** — avoid spaghetti wiring
- **Use sticky notes** on complex canvases to label sections
- **Node Notes field**: explain *why*, not just *what* — future maintainers will thank you
- **Avoid deeply nested expressions**: if an expression is very long, move logic to a Set or Code node where it can be named
- **Don't over-engineer early**: build the simplest version first, add complexity only once logic is proven

---

## 4. Self-Check Before Submitting

| Question | If YES → Action |
|----------|----------------|
| Does this workflow have > 30 nodes? | Consider proposing a subflow split to the user |
| Does it process multiple items in one run? | Suggest one-item-per-run architecture + logging ExecutionID |
| Does it have Code nodes with default names? | Rename to `<Verb> <Object>` |
| Are Code nodes doing something native nodes could handle? | Prefer native — mention in the response |
| Are Code nodes > 40 lines and unexplained? | Add comments + Notes; flag to user if risky |
| Are there chained Code → Code1 → Code2 nodes? | Consolidate or replace with native nodes |

---

## 5. Combined Pre-Submit Reference

This rule compiles with the other two agent rules. Before every workflow submit, all three apply:

| Rule File | Focus | Mandatory? |
|-----------|-------|-----------|
| `/n8n_naming_conventions` | Names for workflows, nodes, attributes, credentials | ✅ Yes |
| `/n8n_error_handling` | Retry on Fail, Error Workflow, design-level validation | ✅ Yes |
| `/n8n_best_practices` (this file) | Workflow size, Code node usage, readability | ⚠️ Soft — apply with judgment |

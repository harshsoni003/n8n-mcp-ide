# N8N Best Practices

> **Purpose**: Prevent architectural mistakes before they become expensive problems.  
> **Important**: These guidelines are **not mandatory** — they are food for thought. Always align with your solution's architecture and the customer's requirements. Use your judgment.

---

## 1. The "One-Mega-Workflow" Anti-Pattern

### The Problem

Building everything into a single massive workflow feels fast at first — but as complexity grows, it becomes:
- **Impossible to read** at a glance
- **Painful to debug** when something breaks (which log entry caused the failure?)
- **Fragile** — a single node failure can block unrelated logic
- **Untestable** — you can't run one part without running all of it

The image below is an example of what to avoid:

> A canvas with 50+ nodes, multiple parallel tracks, dozens of merge nodes, and no clear entry/exit points is a maintenance nightmare.

---

### The Guideline

**Keep each workflow under 20–30 nodes.**

If your canvas is growing beyond that, it's a signal to break it apart.

---

### How to Fix It: Use Subflows

Split responsibilities across multiple focused workflows using **Execute Workflow** nodes:

```
Main Workflow (orchestrator, ≤ 20 nodes)
├── Subflow: Fetch & Validate Leads
├── Subflow: Enrich Contact Data
├── Subflow: Sync to HubSpot
└── Subflow: Notify via Slack
```

**Benefits of subflows**:
- Each subflow can be developed, tested, and debugged independently
- Execution logs are scoped — a failure in "Sync to HubSpot" doesn't bury "Fetch Leads" logs
- Subflows can be reused across multiple parent workflows
- Easier to hand over to another developer

---

### One Item Per Workflow Run

Where possible, design workflows to process **one item per execution**:

| Approach | Recommendation |
|----------|---------------|
| **Batch trigger** (e.g. Schedule gets 100 records) | Split out → trigger subflow per item |
| **Webhook** (single event) | Natural one-item design ✅ |
| **Loop** inside one workflow | Consider splitting into parallel executions instead |

**Why it matters**:
- Logs show exactly which item caused a failure
- Failed items can be retried individually
- No risk of a bad record at position 47 silently stopping records 48–100

**Practical tip**: When you do process multiple items, always **save the execution link** in your log entry so you can jump directly to the run that failed.

```javascript
// In your logging node, include:
{
  ExecutionID: "{{ $execution.id }}",
  ExecutionURL: "{{ $execution.id }}"  // Store so you can look up in n8n
}
```

---

## 2. The "Code Node Vibe Coder Hero" Anti-Pattern

### The Problem

It's tempting to use a Code node to solve everything — especially when an AI tool will write 100 lines of JavaScript in seconds. But this creates a hidden time bomb:

- You paste in generated code without fully understanding it
- It works… until it doesn't
- When it breaks, nobody (including you) can fix it quickly
- Debugging someone else's (or an AI's) opaque code node is some of the most frustrating work in n8n

The image below shows what to avoid:

> A chain of `Code` → `Code1` → `Code2` nodes with default names and no notes — when `Code2` errors, good luck.

---

### The Guideline

**The 10-second rule**: If you can't explain what a Code node does in 10 seconds, reconsider using it.

---

### Prefer Native Nodes Over Code Nodes

Before writing code, ask: *"Can n8n do this natively?"*

| Task | Use instead of Code |
|------|-------------------|
| Rename / set fields | **Edit Fields (Set)** node |
| Filter records | **Filter** node |
| Combine arrays | **Merge** node |
| Split a list | **Split Out** node |
| Conditional branching | **IF** / **Switch** node |
| Date formatting | **Date & Time** node or expressions |
| String manipulation | n8n expressions (`{{ }}`) |
| Deduplication | **Remove Duplicates** node |
| Sorting | **Sort** node |
| HTTP calls | **HTTP Request** node |

---

### When a Code Node IS Acceptable

A Code node is appropriate when:

- ✅ The logic genuinely can't be expressed with native nodes or expressions
- ✅ You can explain every line — no black boxes
- ✅ The code is short and focused (under ~30 lines per node)
- ✅ You've added comments explaining the non-obvious parts
- ✅ The node has a descriptive name (`Map Fields to HubSpot Format`, not `Code`)

---

### If You Must Use a Code Node — Do It Right

1. **Name it properly**: Use `<Verb> <Object>` — e.g., `Format Phone Numbers`, `Build Payload`
2. **Add a note**: Use the node's Notes field to describe what it does in plain English
3. **Keep it short**: If it's over 40–50 lines, break it into multiple named Code nodes or reconsider native nodes
4. **Comment non-obvious logic**:

```javascript
// Normalize phone to E.164 format for Twilio (requires leading +)
const phone = $input.item.json.Phone.replace(/\D/g, '');
return [{ json: { PhoneE164: `+${phone}` } }];
```

5. **Test edge cases manually**: What happens if the input is null? An empty array? A string where a number is expected?

---

## 3. Additional Architectural Tips

### Keep Flows Readable Top-to-Bottom
- Arrange nodes left-to-right, avoiding spaghetti connections
- Use sticky notes to label sections of a complex canvas
- Leave whitespace between logical sections

### Don't Over-Engineer Early
- Build the simplest version that works first
- Add retries, error handling, and subflow decomposition only once the logic is proven
- A working simple flow beats a broken complex one

### Document Decisions in Node Notes
- Use the **Notes** field on nodes to explain *why* a decision was made, not just *what* the node does
- Future you (or your colleague) will thank you

### Avoid Deeply Nested Expressions
- If an expression is so long it wraps multiple lines, move the logic to a Set or Code node where it can be named and tested
- `{{ $json.a?.b?.c?.[0]?.d ?? 'default' }}` in an inline field is a maintenance hazard

---

## 4. Quick-Reference Cheat Sheet

| Anti-Pattern | Warning Sign | Fix |
|---|---|---|
| One-mega-workflow | > 30 nodes on a single canvas | Break into subflows |
| Unnamed nodes | "HTTP Request", "Code1", "Set" remain | Rename to `<Verb> <Object>` |
| Code node black box | Can't explain it in 10 seconds | Use native nodes or add comments |
| Batch processing in one run | 100 items, one execution log | Split out → one item per run |
| Missing execution link in logs | Can't trace which run failed | Always log `ExecutionID` |
| Chained Code nodes | Code → Code1 → Code2 | Consolidate or replace with native nodes |

---

## 5. Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-02-28 | Initial document created | Antigravity |

> 💬 **Suggest changes** by adding a comment below or editing this file directly.

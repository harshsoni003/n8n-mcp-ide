# N8N Naming Conventions

> **Purpose**: Keep naming consistent, self-explanatory, and easy to hand over across every n8n canvas.  
> **Rule of thumb**: Start simple — rename only when it adds clarity.

---

## 1. Workflow Names

### Pattern

| Stage | Format |
|-------|--------|
| Development | `<DEV> - <Who> - <Trigger/Subflow> - <Point of the flow>` |
| Production  | `<PROD> - <Trigger/Subflow> - <Point of the flow>` |

### Rules
- Use **DEV** while you are actively developing or debugging the flow.
- `<Who>` = the **first name of the developer** currently building or owning the workflow.
  - If name is known from context, use it automatically.
  - If name is unknown, ask the developer before creating the workflow.
- If you hand over development, create a copy or rename with the **new developer's first name**.
- Switch to **PROD** only after the flow has been **approved by your project manager** and is live.
  - **PROD names have no `<Who>`** — the workflow belongs to the product, not an individual.

### Examples

```
DEV - Harsh - Webhook - Sync New Leads to HubSpot     ← Harsh is building it
DEV - Pratik - Schedule - Send Weekly Report Email     ← Pratik is building it
DEV - Maria - WhatsApp - AI Echo Bot                  ← Maria is building it
PROD - Webhook - Sync New Leads to HubSpot
PROD - Schedule - Send Weekly Report Email
PROD - Subflow - Enrich Contact Data
```

---

## 2. Node Names

### Pattern

```
<Verb> <Object>
```

### Rules
- Each node must have a **clear, self-explanatory name** that describes its action.
- Use title case: `Fetch Leads`, not `fetch leads` or `FETCH LEADS`.
- Be specific: `Filter New Records` not just `Filter`.
- Maintain consistency — if you write `Send Email` in one flow, don't write `Email Send` in another.

### Verb Reference

| Category | Good Verbs |
|----------|------------|
| Read | Fetch, Get, Read, Load, List |
| Write | Create, Update, Upsert, Delete, Insert |
| Transform | Format, Map, Merge, Filter, Split, Sort |
| Communicate | Send, Notify, Post, Publish |
| Control | Wait, Check, Route, Loop, Retry |

### Examples

```
✅ Fetch Leads
✅ Send Email
✅ Filter New Records
✅ Map Contact Fields
✅ Update Deal Stage
✅ Check Rate Limit
✅ Split by Status

❌ New node
❌ HTTP Request
❌ Code
❌ email
```

---

## 3. Attribute Names

### Pattern

```
PascalCase (UpperCamelCase), ≤ 4 words — answer "what happens here?"
```

### Rules
- Every word starts with a capital letter, no separators.
- Abbreviations stay fully uppercase: `ID`, `URL`, `API`.
- ≤ 4 words is the goal; go beyond only if absolutely necessary for clarity.
- Use the same convention in integrated tools: **Airtable, ClickUp, HubSpot, Salesforce, etc.**

### Examples

```json
{
  "DealID": 12345,
  "CustomerID": 987,
  "CreatedAt": "2025-06-02T09:15:00Z",
  "FirstName": "Jane",
  "LastName": "Doe",
  "AssignedToUserID": 42,
  "LeadSourceURL": "https://example.com/signup",
  "IsEmailVerified": true,
  "TotalRevenueUSD": 15000.00,
  "ThatOneVeryLongAttributeDescribingEverything": "42"
}
```

### Exceptions
| Case | Rule |
|------|------|
| Abbreviations | Keep fully uppercase: `ID`, `URL`, `APIKey` |
| Cross-tool consistency | Follow the target tool's convention when mapping to external APIs (GraphQL, REST/JSON schemas) |

---

## 4. Credential Names

### Pattern

```
<Client>-<Service>-<Purpose>-<Env>
```

| Part | Description | Examples |
|------|-------------|---------|
| `Client` | The client or project name | `4am`, `Acme`, `Internal` |
| `Service` | The service or integration | `Gorgias`, `HubSpot`, `OpenAI` |
| `Purpose` | What the credential is used for | `APIkey`, `OAuth`, `Webhook` |
| `Env` | Environment | `PROD`, `DEV`, `STAGING` |

### Examples

```
4am-Gorgias-APIkey-PROD
Acme-HubSpot-OAuth-DEV
Internal-OpenAI-APIkey-PROD
Client1-Airtable-PersonalToken-STAGING
```

### Rules
- Always include the environment suffix so it's immediately obvious which credentials are live.
- Never share or reuse `PROD` credentials between clients.
- When rotating keys, create a new credential with the same name convention; delete the old one once confirmed working.

---

## 5. Quick-Reference Cheat Sheet

| Area | Pattern | Example |
|------|---------|---------|
| Workflow (Dev) | `DEV - <Who> - <Trigger> - <Purpose>` | `DEV - Pratik - Webhook - Sync Leads` |
| Workflow (Prod) | `PROD - <Trigger> - <Purpose>` | `PROD - Webhook - Sync Leads` |
| Node | `<Verb> <Object>` | `Fetch Leads` |
| Attribute | `PascalCase ≤ 4 words` | `CustomerID`, `CreatedAt` |
| Credential | `<Client>-<Service>-<Purpose>-<Env>` | `4am-Gorgias-APIkey-PROD` |

---

## 6. Changelog

| Date | Change | Author |
|------|--------|--------|
| 2026-02-28 | Initial document created | Antigravity |
| 2026-03-01 | Made `<Who>` dynamic — uses the building developer's name, not a hardcoded default | Antigravity |

> 💬 **Suggest changes** by adding a comment below or editing this file directly. All proposed changes should be reviewed before being applied to active workflows.

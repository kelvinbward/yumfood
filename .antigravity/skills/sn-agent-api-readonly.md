# SN Agent API Command Schema (READ-ONLY)

> ⚠️ **BOUNDARY ENFORCEMENT:** This skill covers ONLY read-only commands. Write commands (`create_artifact`, `update_record`, and any mutation operations) are **explicitly disabled** in this environment to protect the Now SDK source-of-truth. If a task requires creating or modifying a record, it MUST be expressed as Fluent TypeScript in `src/` and deployed via `now deploy`.

---

## Request File Protocol

| Field | Value |
|---|---|
| **File Location** | `{workspace_root}/{instance}/agent/requests/req_{id}.json` |
| **Schema** | `{ "id": string, "command": string, "params"?: object }` |
| **Response Location** | `{workspace_root}/{instance}/agent/responses/res_{id}.json` |
| **Cleanup Rule** | Delete BOTH `req_` and `res_` files immediately after reading the response |
| **Polling** | `while [ ! -f "agent/responses/res_{id}.json" ]; do sleep 0.1; done` |

**Always call `check_connection` first.** Never issue any other command if connection is not confirmed.

---

## Core Introspection Commands

### `check_connection`
Validates the WebSocket connection to the live PDI. **No params.** Must succeed before any other command.
```json
{ "id": "conn-001", "command": "check_connection" }
```

### `get_table_metadata`
Returns field definitions, types, and references for a given table. Result should be cached to `structure.json` locally to reduce repeated queries.
```json
{ "id": "meta-001", "command": "get_table_metadata", "params": { "table": "x_food_inventory" } }
```

### `query_records`
Queries records from a table. Used primarily to **verify that a Now SDK deployment succeeded** — e.g., confirming a new table is accessible and populated correctly.
```json
{
  "id": "qry-001",
  "command": "query_records",
  "params": {
    "table": "x_food_inventory",
    "query": "active=true",
    "fields": "sys_id,name,quantity,expiration_date",
    "limit": 10
  }
}
```

---

## UI & Debugging Commands

### `take_screenshot`
Captures the current or specified PDI browser tab. Saves to `{workspace}/screenshots/`. Use for visual debugging and validating UI Builder output.
```json
{ "id": "ss-001", "command": "take_screenshot", "params": { "url": "https://{instance}.service-now.com/x_food_inventory_list.do" } }
```

### `run_slash_command`
Executes a SNUtils slash command on the live PDI for debugging. Key commands:
* `/tn` — Reveal technical field names on a live form (essential before writing Fluent field references)
* `/reload` — Hard reload the current page
```json
{ "id": "slash-001", "command": "run_slash_command", "params": { "command": "/tn", "url": "https://{instance}.service-now.com/x_food_inventory_form.do" } }
```

### `activate_tab`
Navigates the PDI browser tab to a specified URL, optionally reloading it.
```json
{ "id": "tab-001", "command": "activate_tab", "params": { "url": "https://{instance}.service-now.com/nav_to.do?uri=x_food_inventory_list.do", "reload": true } }
```

---

## Workflow: Deployment Verification Pattern

After every `now deploy`, use the following sequence to confirm success:

```bash
# 1. Write request file
echo '{ "id": "verify-001", "command": "check_connection" }' > agent/requests/req_verify-001.json

# 2. Poll for response
while [ ! -f "agent/responses/res_verify-001.json" ]; do sleep 0.1; done

# 3. Read response
cat agent/responses/res_verify-001.json

# 4. Cleanup IMMEDIATELY
rm agent/requests/req_verify-001.json agent/responses/res_verify-001.json

# 5. Then query_records to confirm deployed artifact is live
```

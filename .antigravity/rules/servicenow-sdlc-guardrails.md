# ServiceNow Agentic SDLC Protocols (WSL/Ubuntu)

## 🛑 ARCHITECTURAL BOUNDARY: Now SDK vs. SN Utils

This project enforces a **strict separation of concerns** to protect the Application-as-Code source-of-truth. Violating this boundary corrupts the git-tracked deployment record.

| Concern | Tool | Direction | Rationale |
|---|---|---|---|
| Code & Schema Generation | Now SDK (Fluent / TypeScript in `src/`) | **WRITE** | Immutable AaC source-of-truth |
| Live Introspection & Debug | SN Utils Agent API (`agent/requests/`) | **READ-ONLY** | Scout & validate — never mutate |

* **WRITE OPERATIONS (The Architect):** All code generation, table definitions, roles, ACLs, and business logic MUST be written in TypeScript using the ServiceNow Fluent framework in the `src/` directory. Deployed exclusively via `now deploy`. Do NOT use SN Utils commands (`create_artifact`, `update_record`) to create or modify records.
* **READ OPERATIONS (The Scout):** The SN Utils Agent API (`agent/requests/`) is restricted to strictly **READ-ONLY** operations: live introspection, schema validation, UI debugging, and deployment verification. **Never generate or alter application records via this channel.**

---

## System & Pathing (WSL/Ubuntu)

* **Environment:** Ubuntu on WSL. Use strict Unix file paths at all times.
* **Polling Pattern:** Use `while [ ! -f "file" ]; do sleep 0.1; done` for file-based async waits. Do NOT use Windows-style paths or PowerShell idioms.
* **Root API Directory:** All Agent API request files go to `{workspace_root}/{instance}/agent/requests/`. Responses arrive in `agent/responses/`. **Delete BOTH files immediately after processing.**
* **Screenshots:** Saved to `{workspace}/screenshots/`. Reference by filename only.

---

## Critical Guardrails

1. **PREREQUISITE:** ALWAYS call `check_connection` before ANY SN Utils Agent API operation. If connection fails, halt and report — do not attempt retries that may corrupt state.
2. **BROWSER TOOLS FORBIDDEN:** NEVER use internal IDE browser tools (e.g., `open_browser_url`, Playwright). Exclusively use the SN Agent API (`take_screenshot`, `run_slash_command`, `activate_tab`).
3. **NO SPECULATIVE WRITES:** Never call `create_artifact`, `update_record`, or equivalent write commands via the Agent API. These are disabled to protect the Now SDK source-of-truth.
4. **VERIFY AFTER DEPLOY:** After any `now deploy`, use `query_records` to confirm the deployment succeeded against the live PDI. Do not assume success.
5. **EPHEMERAL QUEUE FILES:** Request/response files in `agent/requests/` and `agent/responses/` must be deleted immediately post-processing. They are never committed to git.

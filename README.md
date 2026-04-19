# yumfood 🍎

An AI-Orchestrated Food Inventory & Recipe Generation Platform

A modern, enterprise-grade application built natively on ServiceNow using the Now SDK and Fluent framework. This project demonstrates an "Application-as-Code" (AaC) approach, leveraging an Agentic SDLC workflow with Antigravity and the SNUtils Agent API.

## 📝 Project Description
**yumfood** is designed to combat household food waste and cooking fatigue. By treating the ServiceNow Personal Developer Instance (PDI) as a headless backend, the application tracks pantry inventory, monitors expiration dates, and automates grocery replenishment. It integrates with external recipe APIs to suggest meals based on current stock, prioritizing items nearing their end-of-life.

## 🛠️ Technical Stack
* **Platform:** ServiceNow (PDI - Xanadu or later)
* **Framework:** ServiceNow Fluent (TypeScript-based Application-as-Code)
* **CLI:** Now SDK
* **IDE & AI:** Antigravity (with MCP for live PDI schema visibility)
* **DevOps:** SNUtils Agent API (ScriptSync 4.0), GitHub Actions
* **Frontend:** Next Experience / UI Builder

## 🚀 Agentic Development Workflow
This project is built using a pair-programming paradigm between a Human Architect and L3 Agents:
* **Orchestration:** Antigravity serves as the AI-native environment.
* **Connectivity:** CData Connect AI (MCP) allows agents to query the PDI's live metadata.
* **Deployment:** The SNUtils Agent API enables agents to push code directly to the PDI via the file system (agent/requests/ queue), bypassing manual UI interactions.

## ✨ Key Features
* **Fluent Data Model:** Programmatically defined tables for Inventory, Recipes, and Shopping Lists.
* **Waste Analytics:** Dashboarding for "Days-to-Waste" and high-risk expiration alerts.
* **Automated Replenishment:** Business logic that triggers shopping list entries when inventory levels drop to zero.
* **Headless Integration:** Outbound REST integration with recipe engines (e.g., Spoonacular) to provide dynamic cooking variety.

## 📋 Project Plan Checklist

### Phase 1: Environment & Tooling
- [X] Install Now SDK & Node.js.
- [X] Initialize `food` scoped application via SDK.
- [X] Configure Antigravity MCP connection to ServiceNow PDI.
- [X] Enable SNUtils ScriptSync Agent API and directory structure.

### Phase 2: Data Modeling (Fluent)
- [ ] Define `x_food_inventory` table in Fluent.
- [ ] Define `x_food_shopping_list` table in Fluent.
- [ ] Define `x_food_recipe` and `x_food_ingredient` tables.
- [ ] Deploy Fluent schema to PDI via Now SDK.

### Phase 3: Security & Access
- [ ] Define `x_food.user` and `x_food.admin` roles in Fluent.
- [ ] Configure ACLs (Create, Read, Write, Delete) for all tables.
- [ ] Validate role-based access in the PDI.

### Phase 4: Business Logic (Server-Side)
- [ ] Create `FoodUtils` Script Include via Fluent.
- [ ] Implement Business Rule for auto-shopping list generation.
- [ ] Implement "Mark as Purchased" logic for inventory replenishment.

### Phase 5: External Integrations
- [ ] Configure Outbound REST Message for Recipe API.
- [ ] Write logic to map inventory items to API search parameters.
- [ ] Implement Recipe discovery service.

### Phase 6: Next Experience UI
- [ ] Create Workspace in UI Builder.
- [ ] Build Inventory Dashboard with data visualizations.
- [ ] Implement "Recipe Matcher" modal.

### Phase 7: CI/CD & Documentation
- [ ] Configure GitHub Actions for automated deployment.
- [ ] Finalize README and documentation for portfolio presentation.

---

## 🏛️ Architectural Controls & Pair Programming

This repository enforces a formal **Read/Write Boundary** between the two AI-integrated toolchains. Human developers must be aware of these controls before contributing.

### The Pair Programming Paradigm

Development follows a structured collaboration model between the **Human Architect** and the **L3 Agent**:

| Role | Responsibility |
|---|---|
| **Human Architect** | Defines product vision, approves data models, reviews Fluent TypeScript before `now deploy`, and owns the git branching strategy. |
| **L3 Agent (Antigravity)** | Implements Fluent schema/logic in `src/`, uses the SN Agent API to scout the live PDI for schema validation, and reports deployment verification results. |

The Human Architect has final approval on all structural changes. The Agent is an implementation partner, not an autonomous deployer.

### The Read/Write Boundary (Critical)

This project uses **two separate toolchains** with strictly enforced responsibilities:

#### ✅ WRITE — Now SDK (Fluent Framework) — *The Source of Truth*
The `src/` directory contains the canonical, git-tracked representation of the application. All structural changes — tables, roles, ACLs, business rules, script includes — **must originate here** as TypeScript files and be deployed via `now deploy`.

* This is the **immutable source-of-truth**. If it isn't in `src/`, it doesn't officially exist.
* Direct record creation via any other mechanism is **prohibited** as it creates drift between the codebase and the live instance.

#### 🔍 READ — SN Utils Agent API — *Live Introspection Only*
The `agent/requests/` queue provides a file-system bridge to the live PDI for real-time debugging and validation. This channel is **locked to read-only operations**:

* ✅ Allowed: `check_connection`, `get_table_metadata`, `query_records`, `take_screenshot`, `run_slash_command`, `activate_tab`
* ❌ Prohibited: `create_artifact`, `update_record`, or any mutation command

Use the Agent API to **verify** that a `now deploy` succeeded — never to perform the deployment itself.

### Configuration Files
| File | Purpose |
|---|---|
| `.antigravityignore` | Excludes token-heavy auto-generated files (e.g., `agentinstructions.md`) from AI context |
| `.antigravity/rules/servicenow-sdlc-guardrails.md` | Declarative architectural constraints loaded by the AI agent |
| `.antigravity/skills/sn-agent-api-readonly.md` | Command schema reference for the read-only SN Agent API |

---

### 🛑 Repository Description (250 chars)
A ServiceNow application-as-code project using Fluent & Now SDK. Manages inventory, automates shopping lists, and generates recipes. Developed via an Agentic SDLC with Antigravity and SNUtils Agent API for high-impact enterprise resume value.

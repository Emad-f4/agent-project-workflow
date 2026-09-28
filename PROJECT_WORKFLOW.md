# Project Workflow

This file defines the **lifecycle of the whole project**; it defines neither the rules of a specific Agent nor the system architecture (those live in `AGENT_PROJECT_SPEC.md` and each Agent's `instructions.md`).

> This file is a **proposed template**. During Bootstrap it must be approved or adjusted by the user. Unnecessary Phases (e.g. UX for an API-only project) are removed or skipped with user approval, and the reason is recorded in `docs/decisions/`.

## 1. Overall Path

```text
IDEA
 ↓
DISCOVERY
 ↓
PRODUCT DEFINITION
 ↓
MVP / SCOPE
 ↓
PRD
 ↓
REQUIREMENTS
 ↓
UX / UI            (only projects with a user interface)
 ↓
ARCHITECTURE
 ↓
TASK BREAKDOWN
 ↓
IMPLEMENTATION
 ↓
QA
 ↓
DEPLOYMENT
 ↓
OPERATION
 ↓
ITERATION
```

General rule: each stage is one or more Phases, each Phase is closed with a user Gate (Spec sections 7.1 and 16), and no stage starts before the previous one is complete.

---

## 2. Stages

### 2.1 IDEA → Goal

* **Objective:** Turn the raw idea into a clear Goal and Scope, before any Phase or Task.
* **Input:** The user's idea.
* **Output:** `docs/product/idea.md`, `docs/product/goal.md`.
* **Gate:** Explicit user approval of Goal and Scope.
* **Owner:** Master Agent + product-agent.

### 2.2 DISCOVERY

* **Objective:** Understand the problem, users, market/competitors, and constraints.
* **Output:** `docs/product/problem.md`, `docs/product/users.md`, a list of assumptions and risks.
* **Exit:** The problem and target user are defined; critical assumptions are identified.

### 2.3 PRODUCT DEFINITION

* **Objective:** Define the product, its value proposition, and core features.
* **Output:** `docs/product/product-definition.md`.
* **Exit:** The user has approved the product.

### 2.4 MVP / SCOPE

* **Objective:** Determine the minimum deliverable product and what is out of scope.
* **Output:** `docs/product/mvp.md` (including Out-of-scope).
* **Exit:** Scope is approved; later changes only through Change Management (Spec section 9).

### 2.5 PRD

* **Objective:** A complete product requirements document.
* **Output:** `docs/product/prd.md`.
* **Exit:** The PRD is approved by the user.

### 2.6 REQUIREMENTS

* **Objective:** Turn the PRD into testable Requirements (Functional / Non-functional) and Business Rules.
* **Output:** `docs/requirements/`.
* **Exit:** Every Requirement has an ID and Acceptance Criteria; no ambiguity is left open or it is recorded in `pending_decision`.

### 2.7 UX / UI  (conditional)

* **When it applies:** Only if the project has a user interface or user interaction.
* **Not required for:** API-only, Backend-only, Infrastructure.
* **Sub-stages:**

```text
User Flow
 ↓
Wireframe
 ↓
Prototype
 ↓
UI Design
 ↓
UX Validation
```

* **Output:** `docs/design/` (flows, wireframes, prototype, design tokens).
* **Exit:** The user has approved the Prototype and UI, and UX Validation is done.

### 2.8 ARCHITECTURE

* **Objective:** Determine the stack, system structure, database, API, and dependency managers.
* **Output:** `docs/architecture/`, `docs/database/`, `docs/api/`, ADRs for key decisions.
* **Exit:** The architecture is approved; the stack and the manager for each language are recorded.

### 2.9 TASK BREAKDOWN

* **Objective:** Break Requirements and Architecture into executable Tasks for the Implementation Phases.
* **Output:** Complete Tasks in `tasks/PHASE-XX/` (Agent, dependencies, Skills/Tools, Inputs/Outputs, Acceptance Criteria).
* **Exit:** Every Requirement has at least one Task; dependencies contain no cycles; the user has approved the order.
* **Agent check:** The Agent of every Task must exist in the Registry. If it does not, a new Agent is created and approved right here (not at execution time) per Spec section 12.1, by asking the user.

### 2.10 IMPLEMENTATION

* **Precondition:** The Docker/`uv` environment is ready (Spec section 24). The first Task of this Phase must be creating it.
* **Objective:** Implement Tasks in dependency order.
* **Output:** Code in `apps/`, infrastructure in `infrastructure/`, tests.
* **Exit:** All required Tasks are DONE and validated; the project runs with `docker compose up -d --build`.

### 2.11 QA

* **Objective:** Verify conformance with Requirements and Acceptance Criteria.
* **Output:** `docs/testing/` (test plan, results, open bugs).
* **Exit:** Zero critical bugs; the user has approved the QA report.
* Bugs become corrective Tasks (Spec section 16.1).

### 2.12 DEPLOYMENT

* **Objective:** Deploy to the target environment.
* **Output:** `docs/deployment/`, deployment pipeline/scripts, runbook.
* **Exit:** Successful deployment and passing smoke test; user approval.

### 2.13 OPERATION

* **Objective:** Monitoring, support, and resolving operational issues.
* **Output:** Operational reports, updated runbook.

### 2.14 ITERATION

* **Objective:** Collect feedback and start the next cycle.
* **Behavior:** New feedback or a new Requirement enters Change Management and, if needed, a new Phase is created (from Discovery or Requirements).

---

## 3. Rules for Moving Between Phases

The next Phase starts only when all five conditions of Spec section 7.1 hold:

```text
Required Tasks Completed
+ Required Artifacts Exist
+ Validation Passed
+ User Decisions Resolved
+ Exit Criteria Satisfied
```

Then:

```text
Phase Completed → User Review → Feedback → Corrective Tasks
→ Validation → Phase Finalized → Next Phase
```

* Skipping a stage (e.g. going from PRD directly to Implementation) is forbidden, unless the user explicitly decides so and an ADR is recorded.
* Returning to a previous Phase when Scope/Requirements change is allowed and is done per Spec section 9.

---

## 4. Stage-to-Agent Mapping (default)

| Stage | Primary Agent | Contributors |
|-------|---------------|--------------|
| Idea / Goal | master | product |
| Discovery, Product, MVP, PRD | product | master |
| Requirements | product | qa |
| UX / UI | ux | frontend |
| Architecture | backend | database, devops |
| Task Breakdown | master | all specialized Agents |
| Implementation | backend, frontend, database, devops | qa |
| QA | qa | — |
| Deployment / Operation | devops | backend |
| Iteration | master | product |

The final mapping is set in the Task; this table is only a default.

---

## 5. Workflow for Existing / Partially Built Projects (Adoption)

When a project already has files and work but is not Agent-Ready, instead of starting from IDEA, this path is run (full details: `AGENT_PROJECT_SPEC.md` section 4.2):

```text
SCAN → SAFETY (Backup/Branch) → CLASSIFY + MIGRATION PLAN
   → GATE 1 (user approval)
   → MIGRATE (every file in its proper place)
   → SCAFFOLD (Agent-Ready structure)
   → STATUS ASSESSMENT (what state is the project in?)
   → GATE 2 (user approval of the report)
   → enter the current stage of the main Workflow
```

### File Mapping (default)

| Existing file type | Proposed destination |
|--------------------|----------------------|
| Idea, notes, problem, users, MVP, PRD | `docs/product/` |
| Requirements and Business Rules | `docs/requirements/` |
| Design, wireframe, prototype | `docs/design/` |
| Architecture, diagrams, decisions | `docs/architecture/`, `docs/decisions/` |
| Schema, migrations, ERD | `docs/database/` |
| API documentation | `docs/api/` |
| Test plans and reports | `docs/testing/` |
| Deployment documentation | `docs/deployment/` |
| Application code | `apps/` (only with user approval) |
| Terraform, K8s, CI/CD | `infrastructure/` |
| Scripts | `scripts/` |
| Config files that tools expect at the root | root (unchanged) |
| Unknown | `docs/_unsorted/` or ask the user |

### Determining the Current Stage

After the Scan, every stage of the main path (section 1) is assessed with evidence:

```text
complete | partial | missing | not_applicable
```

* Current stage = the first stage that is not `complete`.
* `complete` stages are marked `adopted` and have no historical Tasks.
* `partial` and `missing` stages get Tasks (e.g. writing an incomplete PRD, creating Docker, migrating to `uv`).
* Documents derived from code are `inferred` and are not valid until the user approves them.

### Example

```text
Project: unfinished FastAPI code + scattered notes + no Docker
Discovery / Product / MVP   → partial  (notes only; needs approval)
Requirements / Architecture → missing
Implementation              → partial
QA / Deployment             → missing
Current stage: MVP / SCOPE  (approve Scope first, then continue)
```

### Differences from a New Project

* There is no need to start from IDEA; but the Goal and Scope must be approved by the user.
* Until Gate 2, no coding or functional change is performed.
* The Docker and `uv` requirements become Tasks and are done before Implementation continues.

---

## 6. Project Customization

During Bootstrap, the Master Agent asks the user:

1. Does the project have a user interface? (If not, the UX/UI stage is removed.)
2. What is the rough stack? (It determines the dependency managers.)
3. Which stages should be simplified or merged? (e.g. PRD and Requirements in a small project.)

The result is recorded in `docs/decisions/` and the project's real Phases are created in `tasks/`.

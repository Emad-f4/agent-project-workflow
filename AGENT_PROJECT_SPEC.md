# Agent Project Specification

**Version:** 2.0

## 1. Purpose

This project must be built as **Agent-Ready**: a raw idea can be turned into an executable project, and its execution is carried out by several specialized Agents, under the management of one Master Agent, in a controlled and traceable way.

```text
Idea
  ↓
Project Workflow
  ↓
Phase
  ↓
Task
  ↓
Master Agent
  ↓
Specialized Agent
  ↓
Skills / Tools
  ↓
Execution
  ↓
Validation
  ↓
State Update
  ↓
Next Task / Next Phase
```

This file defines the **overall Agent-Ready architecture**. The workflow of the whole project is defined in `PROJECT_WORKFLOW.md`, and each Agent's rules are defined in that Agent's own folder.

---

# 2. Separation of Responsibilities

Three levels must be kept separate.

## 2.1 Project Specification

The file `AGENT_PROJECT_SPEC.md` (this file) defines: project structure, Agent structure, Task System, State Management, Registry, Skill/Tool Resolution, Master Agent, general rules, and the execution environment.

## 2.2 Project Workflow

The file `PROJECT_WORKFLOW.md` defines the lifecycle of the whole project:

```text
IDEA → DISCOVERY → PRODUCT DEFINITION → MVP / SCOPE → PRD → REQUIREMENTS
→ UX / UI → ARCHITECTURE → TASK BREAKDOWN → IMPLEMENTATION → QA
→ DEPLOYMENT → OPERATION → ITERATION
```

This workflow belongs to the **whole project**, not to one specific Agent.

## 2.3 Agent Instructions

Each Agent has its own rules and workflow:

```text
.agent/agents/backend/
├── AGENT.md
├── instructions.md
└── memory/
    └── feedback.md
```

`instructions.md` must contain these sections:

```text
Role, Responsibilities, Rules, Workflow, Inputs, Outputs, Validation,
Required Skills, Required Tools, Stop Conditions, Escalation Rules
```

```text
Project Workflow ≠ Agent Workflow
```

---

# 3. Entry Point: AGENTS.md

`AGENTS.md` is the entry point for all Agents (Claude Code, Codex, Cursor, etc.) and must stay **short** (roughly 30 lines at most).

Its only job is to guide the Agent in the right order:

1. If `.project/state.yaml` does not exist → go to **Bootstrap** (section 4).
2. Otherwise, read `.project/state.yaml`.
3. Read only the needed sections of `AGENT_PROJECT_SPEC.md` (Master Agent: sections 6 to 13 and 25; other Agents: their own `instructions.md` + section 25).
4. Read the current Task from `tasks/` and execute only that.

A ready-made version of this file accompanies this Spec. Project content (business rules, requirements, etc.) is never written inside `AGENTS.md`.

---

# 4. Bootstrap / First Run

When `.project/state.yaml` does not exist, the Master Agent must **first detect the project type**:

| Situation | Mode | Path |
|-----------|------|------|
| Folder is empty or contains only an idea | New project | Section 4.1 |
| Existing files (code, docs, config, Git history, etc.) but no `state.yaml` | **Partially built / existing project (Adoption)** | Section 4.2 |
| `state.yaml` exists | Active project | Normal cycle (section 10) |

If the detection is uncertain, the Master Agent asks the user.

## 4.1 New Project

The Master Agent must perform these steps in order:

```text
1. Create the folder structure per section 5
2. Create .project/state.yaml with status bootstrap
3. If PROJECT_WORKFLOW.md does not exist:
     - Propose one from the standard template
     - Wait for user approval (User Decision Gate)
4. Ask the user for the idea (goal, audience, constraints) and save it in docs/product/idea.md
5. Convert the idea into Goal and Scope (goal, scope, out-of-scope) in docs/product/goal.md
6. GATE: present Goal and Scope to the user and get explicit approval
7. Create PHASE-01 (Discovery) and .project/phase.yaml
8. Generate the initial tasks of PHASE-01 in tasks/PHASE-01/
9. Register the available Agents in .agent/registry/agents.yaml
10. Create .project/context.md, update state.yaml, and start the first Task
```

Bootstrap rules:

* Until the user has provided the idea, no executable Task is created.
* Until the Goal and Scope are approved by the user, no Phase or Task is created and no Implementation starts. A raw idea is never converted directly into Tasks or code.
* Bootstrap runs only once; afterward `state.yaml` is the source of truth.
* Docker and `uv` are not required during Bootstrap (see section 24).

Example `state.yaml` during Bootstrap:

```yaml
project:
  name: unnamed
  status: bootstrap
current_phase: null
active_tasks: []
blocked: false
user_decision_required: true
pending_decision: "Provide project idea"
```

## 4.2 Partially Built / Existing Project (Adoption Workflow)

Goal: bring a project that already exists into the Agent-Ready structure **without breaking it**, put every file in its proper place, and then report the **real status of the project**.

```text
A. Scan (read-only)
 ↓
B. Safety (backup)
 ↓
C. Classification + Migration Plan
 ↓
GATE 1: user approval
 ↓
D. Execute Migration (move files)
 ↓
E. Scaffold the Agent-Ready structure
 ↓
F. Status Assessment (what state is the project in?)
 ↓
GATE 2: user approval of the status report
 ↓
G. Create State, Phases, and remaining Tasks
```

### A. Scan (read-only)

In this step **no file is modified.** The Agent must produce a complete inventory:

* Folder and file structure, languages and stack, dependency managers
* Documentation, README, diagrams, notes
* Tests, Docker, CI, config files, and `.env`
* Git history (commits, branches, TODO/FIXME in code)
* Any task, issue, or plan mentioned in the files

Output: `.project/adoption/inventory.md`.

### B. Safety

Before any move:

* If the project has Git: uncommitted changes must be committed or stashed, and the work must be done on a separate branch (e.g. `adoption/agent-ready`).
* If it has no Git: ask the user to make a backup, or run `git init` and make an initial commit. Without a backup, step D does not start.
* Record the baseline: do Build/Test work right now? (Result in `.project/adoption/baseline.md`.)

### C. Classification + Migration Plan

For **every file**, the Agent determines its destination and writes the following table in `.project/adoption/migration-plan.md`:

| Current file | Type | Destination | Action | Reason |
|--------------|------|-------------|--------|--------|
| `notes/idea.txt` | Idea / Product | `docs/product/idea.md` | move | ... |
| `src/` | Code | `apps/<name>/` | move (needs approval) | ... |
| `Dockerfile` | Infrastructure | root (unchanged) | keep | tools expect it there |
| `misc.zip` | Unknown | `docs/_unsorted/` | ask user | ... |

Rules:

* **No file is deleted.** Files are only moved (using `git mv` so history is preserved).
* Files that tools expect at the root (`package.json`, `Dockerfile`, `pyproject.toml`, `.gitignore`, etc.) stay where they are.
* Moving **code** risks breaking imports and builds; it is done only with explicit user approval and with references fixed. If a move is high-risk, the default proposal is `keep` and the user decides.
* Unknown files are not guessed at; they go to `docs/_unsorted/` or the user is asked.
* Secret files (such as a real `.env`) are never moved or copied into reports; only their existence is noted, and they must be in `.gitignore`.

**GATE 1:** The Plan is shown to the user, and step D does not start without their approval.

### D. Execute Migration

* Perform the moves according to the approved Plan, preferably in several small commits.
* Fix broken references (imports, paths, documentation links, config).
* After moving, run Build/Test again and compare with `baseline.md`. If it got worse, fix it or roll back.
* Record every action in `.project/adoption/migration-log.md`.
* **In this step, no functional code changes and no dependency upgrades are made.**

### E. Scaffold

* Create the Agent-Ready structure (section 5): `.project/`, `.agent/`, `tasks/`, `docs/`, and the Spec/Workflow/AGENTS files.
* **No existing file is overwritten.** If a file with the same name exists, merge or ask the user.
* Create the needed Agents per section 12.1 (by asking the user).

### F. Status Assessment

The Agent assesses the status **based only on evidence** (file paths, commits, Build/Test results) and writes the report `docs/project-status.md`. The report must answer: **"What state is the project in right now?"**

Report contents:

1. **Project summary:** what it is, who it is for, what the stack is (citing evidence).
2. **Status of each stage of `PROJECT_WORKFLOW.md`:** for each stage (Discovery, PRD, Architecture, Implementation, QA, etc.), one of `complete` / `partial` / `missing` / `not_applicable`, together with the evidence files.
3. **Current stage:** exactly where the project sits in the Workflow.
4. **Code:** which parts are implemented, which are unfinished or broken, and the Build/Test status.
5. **Gaps:** documentation, tests, Docker, `uv`, etc. that do not exist.
6. **Deviations from the Spec:** e.g. use of `pip`, no Docker, a different folder structure.
7. **Risks and blockers.**
8. **Open questions for the user.**

Critical rules:

* **Requirements and Business Rules are not invented.** Any document derived from code or files (reverse-engineered) is saved with the label `status: inferred` and a list of evidence, and does not count as "Approved Documentation" until the user approves it (section 20).
* Progress percentage estimates are given only with their basis; if there is no valid basis, write "unknown".
* Wherever evidence is insufficient, the item goes under "open questions", not a guess.

**GATE 2:** The user reviews and corrects the report (e.g. "this part is actually not finished" or "this stack is wrong"). Step G does not start without their approval.

### G. State, Phase, and Task

After the report is approved:

* `phase.yaml` and `state.yaml` are created according to the **approved current stage**. Stages marked `complete` are marked `adopted` (no historical Tasks are created for them, since the work was already done).
* Tasks are created only for **remaining work** and **gaps**, for example:
  * Completing documentation and approving the derived docs (`inferred` → `approved`)
  * Migrating dependencies to `uv` (if Python) or the approved manager
  * Creating the Dockerfile and Docker Compose
  * Writing missing tests
  * Unfinished code work
* The Docker and `uv` Tasks must, per section 24, be completed before Implementation continues.
* `context.md` is created and the normal cycle (section 10) begins.

`state.yaml` during Adoption:

```yaml
project:
  name: existing-project
  status: adoption
adoption:
  step: F        # A to G
  gate_pending: GATE-2
current_phase: null
active_tasks: []
user_decision_required: true
pending_decision: "Review project-status.md"
```

### General Adoption Rules

* Adoption is a **temporary Phase**; coding or functional changes are forbidden in it.
* Every step before the Gates is read-only or planning only.
* If the Agent does not understand something at any step (unknown file, ambiguous stack), it asks; it does not guess.
* Large migrations (changing the stack, rewrites, dependency upgrades) are not done inside Adoption and become separate Tasks with a Gate.

---

# 5. Project Structure

```text
project/
│
├── AGENT_PROJECT_SPEC.md
├── AGENTS.md
├── PROJECT_WORKFLOW.md
│
├── .project/
│   ├── state.yaml
│   ├── phase.yaml
│   ├── context.md
│   └── adoption/          # only for existing projects (section 4.2)
│       ├── inventory.md
│       ├── baseline.md
│       ├── migration-plan.md
│       └── migration-log.md
│
├── tasks/
│   ├── PHASE-01/
│   │   ├── TASK-001.yaml
│   │   └── ...
│   └── PHASE-02/
│       └── ...
│
├── docs/
│   ├── product/
│   ├── requirements/
│   ├── design/
│   ├── architecture/
│   ├── database/
│   ├── api/
│   ├── testing/
│   ├── deployment/
│   └── decisions/        # ADRs
│
├── .agent/
│   ├── agents/
│   │   ├── master/
│   │   ├── product/
│   │   ├── ux/
│   │   ├── backend/
│   │   ├── frontend/
│   │   ├── database/
│   │   ├── qa/
│   │   └── devops/
│   ├── skills/
│   ├── tools/
│   ├── registry/
│   │   └── agents.yaml
│   └── factory/
│
├── apps/
├── infrastructure/
├── scripts/
│   └── progress.py       # computes Progress from Tasks
│
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── uv.lock
├── .env.example
└── .dockerignore
```

The Docker files and `pyproject.toml` are required from the Implementation phase onward (section 24).

---

# 6. Project State

The `.project/` directory holds the current state of the project and must answer the question: **"Where is the project right now?"** An Agent must not guess this from the conversation or its own memory.

## 6.1 state.yaml

Contains: current Phase, active Tasks, active Agents, project status, blocker, and need for a user decision.

**Progress is not stored in `state.yaml`.** Progress is always computed from the Tasks (section 19), to avoid inconsistency.

```yaml
project:
  name: example-project
  status: active

current_phase:
  id: PHASE-01
  name: Product Definition

active_tasks:
  - task: TASK-010
    agent: product-agent

blocked: false
user_decision_required: false
pending_decision: null
```

## 6.2 context.md

`.project/context.md` is a short, current summary of the project so every Agent can get started quickly (one page at most). It is only a **pointer** and summary, not the primary source of knowledge.

Contents:

```text
- Project Goal (one paragraph, linking to docs/product/goal.md)
- Current Scope and Out-of-scope (summary, with links)
- Current Phase and its objective
- Recent key decisions (links to ADRs)
- Approved stack and dependency managers
- Open questions or risks
```

Rules:

* The Master Agent creates it during Bootstrap and updates it at the start/end of every Phase and after every Scope or Architecture change.
* If it conflicts with `docs/` or `state.yaml`, those are authoritative and `context.md` must be corrected.
* New requirements or business rules are written only in `docs/`, not here.

## 6.3 Concurrency Rule

* **Default:** only **one active Task** exists at any moment (`active_tasks` has at most one member).
* Parallel work is allowed only when the user explicitly permits it and the Tasks share no dependencies or output.
* In parallel mode, each Task is recorded separately in `active_tasks` and no two Tasks work on the same output file.

---

# 7. Phase State

The file `.project/phase.yaml` holds the state of the current Phase.

```yaml
id: PHASE-01
name: Product Definition
status: in_progress

objective: >
  Turn the initial idea into a clear goal, scope, and defined features.

required_artifacts:
  - docs/product/problem.md
  - docs/product/users.md
  - docs/product/mvp.md
  - docs/product/prd.md

exit_criteria:
  - product_goal_defined
  - target_users_defined
  - mvp_defined
  - required_tasks_completed
  - user_review_completed
```

## 7.1 Phase Completion

A Phase is complete only when **all** of the following conditions hold at the same time:

```text
Required Tasks Completed   (all required Tasks DONE; CANCELLED/SKIPPED allowed)
        +
Required Artifacts Exist   (all files in required_artifacts exist)
        +
Validation Passed          (validation_result.passed=true for all Tasks)
        +
User Decisions Resolved    (pending_decision empty and user_decision_required=false)
        +
Exit Criteria Satisfied    (every item in exit_criteria is met)
```

Therefore:

```text
Task Done ≠ Phase Done
Phase Done ≠ Project Done
```

Completing all Tasks alone does not complete the Phase. The Master Agent must check each of the five conditions separately and report the result to the user, then run the Gate in section 16.

---

# 8. Task System

The Task is the main unit of project execution. Tasks live in `tasks/` and are organized by Phase.

Tasks are **not** moved around based on Status; the Status is stored inside the Task itself.

## 8.1 Task Definition

```yaml
id: TASK-010
title: Define MVP Features
phase: PHASE-01
status: ready
priority: high
agent: product-agent

dependencies:
  - TASK-007
  - TASK-008

required_skills:
  - product-discovery
  - requirements-analysis
required_tools: []

inputs:
  - docs/product/problem.md
  - docs/product/users.md

outputs:
  - docs/product/mvp.md

acceptance_criteria:
  - MVP features are explicitly defined
  - Scope is documented
  - Out-of-scope functionality is identified

# The fields below are filled in during execution
started_at: null
completed_at: null

blocked_by: []          # IDs of Tasks or other blocking items
blocked_reason: null

validation_result:
  passed: null          # true / false
  checked_by: null
  checked_at: null
  details: null

notes: []
```

## 8.2 Task Lifecycle

```text
BACKLOG → READY → IN_PROGRESS → REVIEW → DONE
```

Side states:

```text
IN_PROGRESS → BLOCKED → (issue resolved) → IN_PROGRESS
any state → CANCELLED   (Task is no longer needed; record the reason in notes)
BACKLOG/READY → SKIPPED (deliberately not done; requires Master Agent approval)
```

* A Task becomes `DONE` only when its Acceptance Criteria are validated and `validation_result.passed = true`.
* `CANCELLED` and `SKIPPED` Tasks are counted separately in Progress and do not block Phase completion.
* When a Task goes to `BLOCKED`, `blocked_reason` is mandatory.

---

# 9. Change Management

When a Requirement, Scope, or Architecture changes:

1. The Master Agent gets the change approved by the user (User Decision Gate).
2. The decision is recorded in `docs/decisions/` as an ADR (`ADR-NNN-title.md`, containing: Context, Decision, Consequences, date).
3. `DONE` Tasks that are affected are identified and:
   * If they only need review → they go back to `REVIEW`.
   * If they must be redone → they go back to `READY`.
   * The reason for the return is recorded in that Task's `notes`.
4. If a Phase was already completed and the change affects it, its status goes back to `in_progress` and the Gate is required again.
5. Related docs are updated and `state.yaml` is rewritten.

---

# 10. Master Agent

The Master Agent is responsible for Orchestration of the project. In every cycle it must:

1. Read the State (if it does not exist → Bootstrap).
2. Identify the current Phase and Task.
3. Check dependencies.
4. Select the appropriate Agent from the Registry. **If no suitable Agent exists:** mark the Task `BLOCKED` (`blocked_reason: agent_missing`), set `user_decision_required: true`, and start the Agent creation process (section 12.1) by asking the user. It must not guess the Agent, and must not give the Task to an unsuitable Agent.
5. Resolve the needed Skills and Tools.
6. Assign the Task to the Agent.
7. Validate the result.
8. Update the Task State and Project State.
9. Determine the next Task.
10. Check the Phase Exit Criteria and, if complete, run the next Gate.

The Master Agent must not enter Implementation without a specific Task.

---

# 11. Agent Registry

Agents are registered in `.agent/registry/agents.yaml`:

```yaml
agents:
  backend-agent:
    path: .agent/agents/backend
    capabilities: [python, fastapi, api, backend]

  frontend-agent:
    path: .agent/agents/frontend
    capabilities: [frontend, ui]

  qa-agent:
    path: .agent/agents/qa
    capabilities: [testing, validation]
```

The Master Agent selects the Agent based on Capability and the Task's needs.

---

# 12. Agent Definition

Each Agent has at least this structure:

```text
.agent/agents/<agent>/
├── AGENT.md
├── instructions.md
└── memory/
    └── feedback.md
```

* **AGENT.md:** the Agent's identity card (ID, Name, Version, Purpose, Capabilities, Path, Status).
* **instructions.md:** its specific rules and workflow. It must be read by the Agent before executing any Task.

## 12.1 Creating a New Agent

**Trigger:** whenever the Agent required by a Task does not exist in the Registry (during Task Breakdown, when assigning a Task, or during Bootstrap), creating a new Agent is mandatory and asking the user is required. The Master Agent does not guess it and does not give the Task to an unrelated Agent.

`.agent/factory/` must contain at least these files:

```text
.agent/factory/
├── questions.md            # list of questions to ask the user
├── instructions.template.md
└── AGENT.template.md
```

For a new Agent, the Factory determines these items and records them in `instructions.md`:

```text
Role, Responsibilities, Rules, Workflow, Inputs, Outputs, Validation,
Required Skills, Required Tools, Stop Conditions, Escalation Rules
```

**The Agent does not guess this information; it must ask the user.** Process:

1. The Master Agent/Factory asks the user about each of the items above (What is the Role? What are the responsibilities? What rules? What is the workflow? etc.). Questions are asked in short groups.
2. If the user does not know an item, the Factory may make a **suggestion**, but the suggestion is valid only with the user's explicit approval.
3. The approved answers are recorded in `.agent/agents/<agent>/instructions.md` and `AGENT.md` is created.
4. The Agent is registered in `.agent/registry/agents.yaml`.
5. Until the user gives final approval, the new Agent receives no Tasks and Tasks waiting for it stay `BLOCKED`.
6. After approval and registration in the Registry, the blocked Tasks return to `READY` (`blocked_reason` is cleared) and the Master Agent continues the cycle.

An Agent must not invent its own specific Workflow and Rules without definition and approval.

---

# 13. Skill and Tool Resolution

Skills and Tools are resolved in the same order:

```text
Agent-level  (.agent/agents/<agent>/skills/  or  tools/)
      ↓
Global       (.agent/skills/  or  tools/)
      ↓
Unavailable
```

* Global Skills and Tools must not be duplicated for different Agents.
* An Agent must not declare that a Skill or Tool does not exist without checking Global.
* If Unavailable, the Agent escalates to the Master Agent or user per its Escalation Rules.

---

# 14. Documentation

The project's main documentation lives in `docs/`. Project Knowledge must not be stored inside Agent Memory. Memory is only for feedback and stable information about how to work with the Agent.

---

# 15. State vs Task vs Documentation

```text
.project/  →  Where are we?
tasks/     →  What must be done?
docs/      →  What do we know?
```

Example: `.project/state.yaml` says the active Task is `TASK-010`; `tasks/PHASE-01/TASK-010.yaml` holds the full Task definition; and `docs/product/mvp.md` holds the product content.

---

# 16. User Decision Gates

For important decisions the Agent must not make the final decision automatically:

```text
Phase Completed → User Review → User Feedback → Apply Changes
→ Validation → Phase Complete → Next Phase
```

Until the Gate is complete, the next Phase does not start. While waiting, `user_decision_required: true` and `pending_decision` are recorded in the state.

## 16.1 Feedback Handling

Any feedback or change request from the user at a Gate (or any other time) is **not executed directly without a Task**:

1. The Master Agent records the feedback in `notes` or `docs/decisions/`.
2. A **corrective Task** is created for it in the current Phase (e.g. `TASK-025`, with a `title` starting with `Fix:` or `Revise:` and a `notes` field referencing the original feedback).
3. The corrective Task, like any other Task, has an appropriate Agent, Acceptance Criteria, and Validation.
4. Affected DONE Tasks go back to `REVIEW`/`READY` per section 9.
5. After Validation, the result is presented to the user again, and the Phase is finalized only with their approval.

If the feedback changes Scope or Requirements, section 9 (Change Management) also applies.

---

# 17. Validation

Every Task must have Validation appropriate to its type.

```text
Execute → Validate → Pass? ── YES → DONE
                          └── NO  → Fix / BLOCKED
```

The validation result is recorded in that Task's `validation_result`.

---

# 18. State Updates

State and Task must be updated after these events: Task start, Task end, blocking, Agent change, User Feedback, User Decision, Requirement/Scope/Architecture change, Phase completion, start of a new Phase.

---

# 19. Progress Tracking

Progress is **never written by hand**; it is computed only from the Task files:

```bash
uv run python scripts/progress.py            # current Phase
uv run python scripts/progress.py PHASE-02   # a specific Phase
```

Sample output:

```text
Phase: PHASE-01 — Product Definition
Total: 20 | Done: 9 | In Progress: 1 | Review: 2 | Ready: 6 | Blocked: 2
Cancelled/Skipped: 0
Active: TASK-010 (product-agent)
```

When answering "Where are we in the project?", the Master Agent must provide the real State information and the output of this script, not guesses or memory of the conversation.

---

# 20. Source of Truth

When information conflicts, the order of authority is:

```text
Current User Decision
  ↓ Approved Documentation
  ↓ Requirements
  ↓ Architecture
  ↓ Task Specification
  ↓ Project State
  ↓ Agent Instructions
  ↓ Agent Memory
  ↓ Assumption
```

An Agent must not violate more authoritative information based on an assumption.

---

# 21. Environment Configuration

* Configuration is read from environment variables or secret management.
* `.env.example` must be in the repository.
* A real `.env` and secrets are never committed.

---

# 22. Dependency Management

Every dependency manager must be specified, fixed, and have a lock file:

| Stack | Manager | Lock File |
|-------|---------|-----------|
| Python | `uv` | `uv.lock` (+ `pyproject.toml`) |
| Node / Frontend | `pnpm` (unless the user specifies another manager) | `pnpm-lock.yaml` |

Rules:

* For Python, `pip install` is not allowed as the primary dependency manager (neither on the host nor inside Docker).
* The manager for each stack is recorded in `docs/architecture/`, and an Agent must not introduce another manager without approval.
* Python examples: `uv add fastapi`, `uv add --dev pytest`, `uv sync`.

---

# 23. Docker

The project's official, reproducible environment is Docker-based.

Minimum files: `Dockerfile`, `docker-compose.yml`, `.dockerignore`.

```bash
docker compose up -d --build    # run
docker compose down             # stop
docker compose logs -f          # view logs
```

* All services the project needs are defined in Docker Compose.
* Dependencies inside Docker are also installed with the specified manager (for Python: `uv sync` from `uv.lock`).
* Official execution must not depend on manually installing dependencies on the host.
* A developer's personal use of a virtual environment is not forbidden, but the official environment must be Docker.

---

# 24. When Docker and Dependency Rules Apply

* In the phases **Discovery through Task Breakdown** (no code), the Docker, `uv`, and `pyproject.toml` requirements do **not** apply.
* From the **start of the Implementation phase** these requirements are mandatory, and creating them must itself be a specific Task (usually for the devops-agent) at the beginning of the Implementation phase.
* The Master Agent must not start a coding Task unless the project's Docker environment is ready.

---

# 25. Non-Negotiable Rules

Agents must not:

* Start executable work without a defined goal or without a specific Task.
* Invent Requirements or Business Rules.
* Change Scope without permission.
* Change a Phase without Exit Criteria.
* Mark a Task `DONE` without Validation.
* Ignore State, or write Progress by hand without computing it.
* Store Project Knowledge in Memory.
* Needlessly duplicate a Global Skill/Tool, or declare one nonexistent before checking Global.
* Manage dependencies outside the designated manager.
* Make official execution of the project depend on a manual host environment (from the Implementation phase).
* Put secrets inside the repository.
* Create complexity without real need.
* Make important Product or Architecture decisions without a Gate and without recording an ADR.

---

# 26. Core Execution Model

```text
USER → MASTER AGENT → PROJECT STATE → PROJECT WORKFLOW → CURRENT PHASE
→ CURRENT TASK → AGENT SELECTION → AGENT INSTRUCTIONS → SKILL RESOLUTION
→ TOOL RESOLUTION → EXECUTION → VALIDATION → TASK UPDATE
→ PROJECT STATE UPDATE → NEXT TASK → PHASE GATE → NEXT PHASE
```

If `state.yaml` does not exist, the path begins with **Bootstrap** (section 4).

---

# 27. Final Principle

> **The Project Workflow defines the path of the whole project; the Task defines what must be done now; the Master Agent decides which Agent does it; Agent Instructions define how that Agent works; Skills and Tools provide the ability to execute; and Project State defines exactly where we are.**

```text
Project Workflow   = the path of the whole project
Task               = the unit of execution
Master Agent       = Orchestrator
Agent Instructions = the Agent's own Rules + Workflow
Skills / Tools     = usable capabilities
Project State      = the project's moment-to-moment status
```

This separation must be maintained throughout the project.

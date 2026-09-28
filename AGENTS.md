# AGENTS.md

This project is Agent-Ready. Before doing anything, follow these steps in order.

## 1. Detect the Situation

* If `.project/state.yaml` **does not exist** → act as the Master Agent and detect the project type (`AGENT_PROJECT_SPEC.md` section 4):
  * Empty folder / idea only → **new project** (section 4.1)
  * Existing files without a state → **partially built project** (section 4.2): first Scan and Backup, then a file-migration Plan with user approval, then a status report. Do not delete any file and do not change the code.
* If it exists → read it. Do not guess the project status from the conversation.

## 2. What to Read

* **Master Agent:** sections 6 to 13 and 25 of `AGENT_PROJECT_SPEC.md`, then `PROJECT_WORKFLOW.md`.
* **Other Agents:** only the assigned Task + `.agent/agents/<agent>/instructions.md` + section 25 (Non-Negotiable Rules).
* Do not read the whole Spec every turn; only the needed sections.

## 3. Quick Rules

1. Do not do executable work without a specific Task.
2. Execute only the current Task (`active_tasks` in the state).
3. Do not invent Requirements, Business Rules, or Scope; if something is unclear, ask.
4. Do not mark a Task `DONE` without Validation.
5. After every important change, update the Task and `.project/state.yaml`.
6. Do not write Progress by hand: `uv run python scripts/progress.py`
7. Important Product/Architecture decision → user Gate + record an ADR in `docs/decisions/`.
8. Do not commit secrets.

## 4. Conflicting Information

Current user decision > approved documentation > Requirements > Architecture > Task > State > Agent Instructions > Memory > Assumption

> This file is intentionally short. Full details are in `AGENT_PROJECT_SPEC.md`. Do not write project content here.

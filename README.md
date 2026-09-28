[English](README.md) | [فارسی](README.fa.md)
# agent-project-workflow
A structured interface between users and AI coding agents for predictable project execution through defined workflows, phases, plans, context, and state.
# Agent Project Workflow

A structured workflow for making AI coding agents work within a predictable, phase-based project structure.

## Why?

Working with AI coding agents can be extremely fast, but speed can also become a problem.

When you give an agent a raw idea such as:

> "Build me an application for managing habits."

the agent may immediately start making decisions on its own.

It might create its own structure, introduce skills or workflows, jump directly into implementation, make architectural decisions before the requirements are clear, or move between different tasks without a well-defined project lifecycle.

The result can be a project that is generated quickly but is not necessarily **structured, traceable, predictable, or aligned with the original intent**.

This led to a simple question:

> **What if the agent had a defined project workflow to follow before it started building?**

Instead of treating the agent as something that simply receives a prompt and generates code, this project introduces a structured layer between the **user** and the **AI coding agent**.

```text
User
  │
  │ Idea / Requirements / Decisions
  ▼
Agent Project Workflow
  │
  ├── Project Specification
  ├── Project Lifecycle
  ├── Phases
  ├── Plans
  ├── Context
  ├── State
  └── Deliverables
  │
  ▼
AI Coding Agent
  │
  ├── Analyze
  ├── Plan
  ├── Execute
  ├── Verify
  └── Report
  │
  ▼
Project
```

The goal is not to make the agent less capable.

The goal is to give that capability a **structure to operate within**.

---

# The Idea

The core idea is to define the project's rules, structure, lifecycle, and agent responsibilities in a small set of Markdown files.

These files act as a persistent project-level contract between the user and the agent.

Instead of repeatedly telling the agent:

> "First analyze the requirements."

> "Don't start coding yet."

> "Let's define the architecture first."

> "Now move to the next phase."

the workflow itself defines these expectations.

The agent can therefore work through a defined process instead of improvising its own process for every project.

This makes project execution more:

* Predictable
* Structured
* Traceable
* Consistent
* Phase-oriented
* Easier to review
* Easier to continue across sessions

---

# What This Changes

Without a defined workflow:

```text
Idea
  ↓
Agent interpretation
  ↓
Agent decisions
  ↓
Code
  ↓
More agent decisions
  ↓
More code
```

With the workflow:

```text
Idea
  ↓
Requirements
  ↓
Project Definition
  ↓
Planning
  ↓
Architecture / Design
  ↓
Implementation
  ↓
Testing
  ↓
Review
  ↓
Deployment
```

The exact lifecycle can be adapted to the project.

The important part is that the agent does not decide the entire development process from scratch every time.

---

# Project Structure

The repository currently contains three core specification files.

```text
agent-project-workflow/
│
├── AGENTS.md
├── AGENTS.fa.md
│
├── AGENT_PROJECT_SPEC.md
├── AGENT_PROJECT_SPEC.fa.md
│
├── PROJECT_WORKFLOW.md
├── PROJECT_WORKFLOW.fa.md
│
├── README.md
└── LICENSE
```

Each file has a different responsibility.

---

## 1. `AGENTS.md`

This file defines the **behavioral instructions for the coding agent**.

It tells the agent how it is expected to operate within the project.

It can define things such as:

* How the agent should approach tasks
* How it should use the project workflow
* What it should do before making changes
* How it should handle requirements
* How it should create plans
* How it should verify its work
* When it should ask the user for decisions
* How it should maintain project context
* How it should report progress

In simple terms:

> **`AGENTS.md` defines how the Agent should behave.**

---

## 2. `AGENT_PROJECT_SPEC.md`

This file defines the **project-level specification for working with an AI agent**.

It describes the structure and concepts used by the workflow.

For example, the project can have concepts such as:

* Project state
* Current phase
* Context
* Plans
* Requirements
* Deliverables
* Decisions
* Phase transitions
* Human approval points

In simple terms:

> **`AGENT_PROJECT_SPEC.md` defines what the Agent-managed project is made of.**

---

## 3. `PROJECT_WORKFLOW.md`

This file defines the **lifecycle of the project**.

It describes how a project moves from an initial idea toward implementation and delivery.

A workflow may look like:

```text
Bootstrap
   ↓
Requirements
   ↓
Architecture
   ↓
Design
   ↓
Implementation
   ↓
Testing
   ↓
Review
   ↓
Deployment
   ↓
Maintenance
```

Each phase can define:

* Objective
* Inputs
* Agent responsibilities
* User decisions
* Expected outputs
* Required artifacts
* Exit criteria
* Conditions for moving to the next phase

In simple terms:

> **`PROJECT_WORKFLOW.md` defines how the project moves forward.**

---

# The Relationship Between the Files

The three files are intentionally separated.

```text
AGENT_PROJECT_SPEC.md
        │
        │ defines the project model
        ▼
PROJECT_WORKFLOW.md
        │
        │ defines the lifecycle
        ▼
AGENTS.md
        │
        │ tells the agent how to operate
        ▼
       AGENT
```

Or more simply:

| File                    | Main Question                                        |
| ----------------------- | ---------------------------------------------------- |
| `AGENT_PROJECT_SPEC.md` | What is the project structure and model?             |
| `PROJECT_WORKFLOW.md`   | How does the project move from one stage to another? |
| `AGENTS.md`             | How should the Agent behave while working on it?     |

This separation makes it possible to evolve the workflow without turning everything into one large instruction file.

---

# How to Use

The workflow is designed to be simple to adopt.

Copy the three core files into the root of your project:

```text
your-project/
│
├── AGENTS.md
├── AGENT_PROJECT_SPEC.md
├── PROJECT_WORKFLOW.md
│
└── ...
```

Then tell your coding agent to read these three files before starting work.

For example:

```text
Read AGENTS.md, AGENT_PROJECT_SPEC.md, and PROJECT_WORKFLOW.md.

These files define the project workflow and your responsibilities as the coding agent.
Follow them throughout the project.
```

After that, the agent uses these files as the project's governing instructions.

The intention is that the agent should **work within the defined workflow rather than inventing a separate workflow for the project**.

---

# From Raw Idea to Structured Project

The biggest use case is starting a project from an incomplete or raw idea.

For example:

```text
"I want an application that helps users build and quit habits."
```

Instead of immediately generating code, the workflow can guide the agent through:

```text
Raw Idea
   ↓
Understand the Problem
   ↓
Clarify Requirements
   ↓
Define the Product
   ↓
Plan the Project
   ↓
Define Architecture
   ↓
Design
   ↓
Implementation
   ↓
Testing
   ↓
Review
```

This allows the user to make important decisions at the appropriate stage.

The agent can still assist with analysis, research, planning, design, implementation, testing, and documentation, but those activities happen **inside a defined project structure**.

---

# Why Markdown?

The workflow intentionally uses Markdown rather than a custom application or complex orchestration system.

This provides several advantages:

* Human-readable
* Agent-readable
* Version-controlled
* Easy to modify
* Easy to review
* Easy to fork
* Easy to adapt to different agents
* No runtime dependency

The workflow itself becomes part of the repository and evolves alongside the project.

---

# Technology

The example project environment uses [`uv`](https://docs.astral.sh/uv/) for Python project and dependency management instead of `pip`.

The workflow itself is not tied to Python, however.

The specification can be adapted to different technology stacks and coding agents.

---

# Design Philosophy

This project is based on a simple principle:

> **An AI coding agent should not only know what to build; it should know how the project is expected to progress.**

A capable agent can generate code very quickly.

The harder problem is maintaining:

* Direction
* Context
* Consistency
* Project state
* Requirements
* Decisions
* Phase boundaries
* Human control

This workflow attempts to address that problem by introducing an explicit structure between the user's intent and the agent's execution.

---

# Status

This project is an evolving experiment in **agent-oriented software project management and development workflows**.

The current specification is intentionally simple and designed to be refined through real-world use.

Feedback, experiments, improvements, and alternative approaches are welcome.

---

# License

This project is released under the MIT License.

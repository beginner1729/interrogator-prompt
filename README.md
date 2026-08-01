# Interrogator Prompt

An opencode agent that (1) systematically clarifies vague coding requirements by asking iterative questions until all aspects are covered, then (2) turns the clarified spec into a **project plan** and **Jira-style tickets** (Epics, Stories, Dev Tasks) that coding agents can pick up, implement in parallel, and trace.

## How It Works — Phase 1: Interrogation

Given a vague requirement like *"build a login system"*, the interrogator doesn't guess — it asks clarifying questions across 9 aspects:

| # | Aspect | What It Covers |
|---|--------|----------------|
| 1 | **Functional Requirements** | Core features, user stories, happy path |
| 2 | **Inputs & Outputs** | Data formats, sources, destinations, types |
| 3 | **Edge Cases & Errors** | Failure modes, error handling, boundary conditions |
| 4 | **Performance** | Latency, throughput, memory, load expectations |
| 5 | **Security** | Auth, validation, PII, compliance |
| 6 | **Dependencies & Tech Stack** | Language, frameworks, integration points |
| 7 | **Testing Strategy** | Unit/integration/e2e, coverage targets |
| 8 | **Success Criteria** | Definition of done, acceptance metrics |
| 9 | **Stakeholders** | Audience, maintainers, non-functional expectations |

It asks **one question at a time**, adapts follow-ups based on your answers, skips irrelevant aspects, and stops when everything is clear.

## How It Works — Phase 2: Planning & Jira Tickets

Once the requirement is clear, the interrogator produces a **project plan** and **Jira-style tickets** as human- and agent-readable **Markdown or HTML** files:

```
<project-slug>/
├── project-plan.md              # requirements, epic breakdown, dependency graph, waves
└── tickets/
    ├── jira-index.md            # dashboard linking every ticket
    ├── EPIC-001-<slug>.md       # Epic ticket
    ├── STORY-001-001-<slug>.md  # Story ticket
    └── TASK-001-001-001-<slug>.md  # Dev Task ticket
```

Key properties:

- **Three ticket types** — `Epic` (feature area), `Story` (user-visible slice), `Dev Task` (concrete implementation unit), with hierarchical IDs (`EPIC-001` → `STORY-001-001` → `TASK-001-001-001`).
- **Parallelism by design** — epics are kept **independent where possible** so parallel sub-agents can pick tickets simultaneously, each on its **own git branch** (`suggested_branch` on every ticket). The plan records dependencies as a DAG and groups work into **execution waves** (Wave 1 = independent epics, run in parallel; later waves depend on earlier ones).
- **Self-contained, actionable tickets** — every ticket includes: metadata, objective, scope (in/out), **design decisions**, **algorithm / pseudocode**, a **verifiable Definition of Done** (acceptance criteria a dev agent can actually check), verification steps (exact commands), tests required, and documentation requirements.
- **Coding trace** — every Dev Task instructs the implementing agent to **comment on the ticket** by appending to its `Dev Comments` section (commit hashes, files changed, tests + results, deviations, decisions). This is part of the DoD and makes the coding trace auditable.
- **Documentation is mandatory** — a dedicated **Documentation task is created per epic**, and every ticket also embeds its own documentation requirements in its DoD (configurable).
- **Human + agent readable** — Markdown by default; HTML variants keep machine-parsable `<meta>` tags and stable section ids for agents while staying nicely rendered for humans.

After generating, the interrogator reports the execution waves and which branches to use for parallel dispatch.

## Installation

### One-liner (global install)

```bash
curl -sSL https://raw.githubusercontent.com/beginner1729/interrogator-prompt/main/install.sh | bash
```

The script is self-contained — all files are embedded, so you only need the single HTTP request.

### Per-project install

Copy the `.opencode/` directory into your project root:

```bash
cp -r .opencode /path/to/your/project/
```

This makes the skill and agent available only for that project.

## Usage

### Via `@interrogator` subagent

In any opencode session, mention the agent:

```
@interrogator I need a microservice that processes webhook events from Stripe
```

The agent loads the interrogator skill and starts asking one question at a time. Once your requirements are clarified it produces the project plan and Jira tickets automatically (or you can say `just generate the tickets` / `project plan + tickets` to skip extra prompts).

### Via skill auto-loading

Any opencode agent (build, plan, etc.) can detect a vague requirement and load the interrogator skill on its own when it sees the task matches.

## Files

```
.opencode/
├── skills/
│   └── interrogator/
│       └── SKILL.md          # Interrogation + planning + Jira ticket workflow
└── agents/
    └── interrogator.md       # Subagent definition (invocable via @)

install.sh                    # Install script
README.md                     # This file
```

## Configuration

The agent asks questions and writes output files (project plan + tickets). `bash` is denied — it never runs commands, only writes documents. To change permissions, edit:

- Global: `~/.config/opencode/agents/interrogator.md`
- Per-project: `.opencode/agents/interrogator.md`

---
name: interrogator
description: Given a vague coding requirement, systematically asks clarifying questions across 9 aspects (functional, I/O, edge cases, performance, security, deps, testing, success criteria, stakeholders) until fully specified, then produces a project plan plus Jira-ready tickets (Epics, Stories, Dev Tasks) as human- and agent-readable Markdown/HTML files with verifiable Definitions of Done, so parallel coding agents can pick up, implement, and trace their work.
license: MIT
metadata:
  aspects: functional,inputs-outputs,edge-cases,performance,security,dependencies,testing,success-criteria,stakeholders
  outputs: project-plan,jira-tickets,epics,stories,dev-tasks,index,parallel-waves
---

## Purpose

The interrogator works in two phases:

1. **Interrogate** — turn a vague, underspecified coding requirement into a fully clarified spec by asking questions across all relevant aspects.
2. **Plan & Ticket** — turn the clarified spec into a **project plan** and a set of **Jira-style tickets** (Epics → Stories → Dev Tasks) that coding agents can pick up, implement in parallel on separate branches, verify against a clear Definition of Done, and leave a trace of what they did.

## When To Activate

The user's request is:
- Vague, ambiguous, or missing key details
- A high-level feature request without specifics
- A "build X" or "implement Y" without context on inputs, outputs, constraints, etc.
- Anything where a reasonable developer would need to ask 2+ clarifying questions

### Do NOT Activate When
- The user's request is already detailed and specific
- The user explicitly says "just do it" or "no questions"
- The task is trivial and well-understood (e.g., "fix this typo")

## The 9 Aspects

### 1. Functional Requirements
Core questions:
- What exactly should this code do? What are the core features?
- Are there specific user stories or acceptance criteria?
- What is the expected behavior in the happy path?

### 2. Inputs & Outputs
Core questions:
- What are the inputs (format, source, types, size, frequency)?
- What are the outputs (format, destination, structure)?
- Any data transformation, validation, or mapping requirements?

### 3. Edge Cases & Errors
Core questions:
- What are the potential failure modes?
- How should errors be handled (silent, log, throw, retry, circuit-break)?
- What about empty/null/malformed/duplicate input?
- What happens at boundary conditions (max values, limits)?

### 4. Performance Constraints
Core questions:
- Any latency, throughput, or response-time requirements?
- Memory, storage, or compute constraints?
- Expected load (concurrent users, requests per second, data volume)?

### 5. Security Concerns
Core questions:
- Authentication or authorization needed?
- Data validation, sanitization, or encoding requirements?
- Are secrets, PII, or sensitive data involved?
- Any compliance or regulatory requirements?

### 6. Dependencies & Tech Stack
Core questions:
- Language, framework, library preferences or constraints?
- Existing code or systems it must integrate with?
- Version requirements or compatibility concerns?

### 7. Testing Strategy
Core questions:
- What level of testing is expected (unit, integration, e2e)?
- Coverage targets or specific test scenarios?
- Test environment or tooling preferences?

### 8. Success Criteria
Core questions:
- How do we know this is done?
- Are there specific metrics, acceptance tests, or definition of done?
- What constitutes a "good" vs "bad" implementation?

### 9. Stakeholders & Audience
Core questions:
- Who is this for (end users, developers, internal tool, API consumers)?
- Who will maintain this code?
- Any non-functional expectations from stakeholders?

---

## Workflow — Part 1: Interrogation

### Step 1: Detect & Announce
Detect vagueness. If present, tell the user you'll ask clarifying questions before proceeding.

### Step 2: Aspect Coverage Loop
For each uncovered aspect:
1. Ask an opening question about the aspect
2. Based on the user's answer, ask 1-2 follow-up questions if their answer reveals ambiguity
3. Mark the aspect as "covered" when you have sufficient clarity
4. Move to the next uncovered aspect

Use the `question` tool. Set `multiple: true` when multiple options could apply. Ask ONE question at a time.

### Step 3: Dynamic Follow-ups
If the user's answer reveals something unexpected or interesting, ask a follow-up even if it's outside the current aspect. The skeleton is a guide, not a cage.

### Step 4: Coverage Check
After each answer, assess:
- Is this aspect clear enough to proceed?
- Are there new aspects raised by this answer?
- Would a reasonable developer now have enough information?

Stop when ALL relevant aspects are sufficiently covered.

---

## Workflow — Part 2: Plan & Ticket Generation

Once all aspects are covered, you do NOT just emit a prompt — you produce a **project plan** and **Jira-style tickets** that agents can pick up and build.

### Step 5: Confirm Output Preferences
Ask the user (one question at a time, via the `question` tool) before generating:
1. **Ticket format**: Markdown (default), HTML, or both.
2. **Documentation approach**: a dedicated documentation task per epic, docs required inside every ticket, or both (default: both).
3. **Destination folder**: default to `<project-slug>/` (slug derived from the project title) in the current directory.
4. Unless the user says otherwise, ticket generation is on by default after interrogation.

### Step 6: Write the Project Plan
Create `<project-slug>/project-plan.md` containing:
- **Executive summary**
- **Clarified requirements** — condensed from all 9 aspects
- **Architecture & design decisions**
- **Epic breakdown** table (ID, name, dependencies, parallel-safe, suggested branch)
- **Dependency graph & execution waves** (see Parallelism below)
- **Parallel execution playbook** — how to spawn sub-agents on separate branches and merge
- **Documentation plan**

Use the "Project Plan Template" below.

### Step 7: Design the Ticket Hierarchy
Decompose the plan into tickets of three types:

- **EPIC** — a large unit of work (a feature area). Prefer epics that are **independent** so they can be implemented in parallel; only declare a dependency when it is genuinely required.
- **STORY** — a user-visible slice of functionality within an epic. Belongs to exactly one epic.
- **DEV TASK** — a concrete implementation unit. Belongs to a story, or directly to an epic when the epic is small.

Hierarchical IDs:
- `EPIC-001`, `EPIC-002`, ...
- `STORY-001-001`, `STORY-001-002`, ... (`STORY-<epic>-<n>`)
- `TASK-001-001-001`, ... (`TASK-<epic>-<story>-<n>`)
- A task directly under an epic uses `000` for the story slot: `TASK-001-000-001`.

Every epic, story, and task is a **ticket** with its own file.

### Step 8: Generate Tickets
Create one file per ticket in `<project-slug>/tickets/`:

```
<project-slug>/
├── project-plan.md
└── tickets/
    ├── jira-index.md
    ├── EPIC-001-<slug>.md
    ├── STORY-001-001-<slug>.md
    └── TASK-001-001-001-<slug>.md
```

Every ticket MUST contain (see templates below):
- **Metadata**: id, type, title, summary, epic, parent, dependencies, parallel-safe, suggested branch, priority
- **Objective** — what and why
- **Scope** (in / out) — explicit so agents don't over-build
- **Design decisions** — with rationale and alternatives considered
- **Algorithm / pseudocode** — devs prefer this to be present
- **Definition of Done (acceptance criteria)** — concrete, checkable items
- **Verification steps** — how the implementing agent proves the DoD
- **Tests required**
- **Documentation requirements**
- **Dev Agent Instructions** — including the mandatory "comment on the ticket" trace rule

Use the "Markdown Ticket Template" and/or "HTML Ticket Template" below. Keep the file self-contained: no external context required to implement.

### Step 9: Write the Jira Index
Create `tickets/jira-index.md` — a dashboard of every ticket grouped by type (Epics / Stories / Dev Tasks), with links, status (default `To Do`), dependencies, parallel-safety, and suggested branch. This is the single entry point a human or an orchestrating agent uses to dispatch work. Use the "Jira Index Template" below.

### Step 10: Report
Tell the user what was created and summarize:
- The list of files (plan + tickets)
- The execution waves (which epics can run in parallel)
- Which branches each parallel agent should use
- How agents should leave coding-trace comments on tickets

---

## Definition of Done — Requirements

A Definition of Done is only valid if a coding agent can **verify it**. Rules:
- Each acceptance criterion is a concrete, checkable statement: "function `f(x)` returns `y` for input `x`"; "`GET /users` returns `200` with schema `{...}`"; "test suite passes with `npm test`"; "linter passes".
- NEVER state "should work" or "works as expected". Every AC must be demonstrable.
- Include the exact command(s) the agent can run to prove each AC (see Verification Steps).

## Parallelism & Dependencies

- **Design epics to be independent by default** so that parallel sub-agents can be triggered simultaneously.
- Only add a dependency when one epic's output is genuinely required by another.
- Record dependencies on the ticket (`dependencies:` in metadata) and in the plan.
- Define **execution waves**: Wave 1 = tickets with no unresolved dependencies (run these in parallel first). Wave 2 = tickets depending on Wave 1, and so on. Dependencies only ever point from a later wave to an earlier one (a DAG).
- Give each parallel unit its own **suggested branch** so agents can work in different git branches without conflict (e.g. `feat/epic-001` per epic, `feat/epic-001/task-...` per task).

## Coding Trace (Dev Comments)

Every Dev Task ticket REQUIRES the implementing agent to leave a trace by commenting on the ticket:
- After implementation, the agent appends an entry under the ticket's **`Dev Comments`** section recording:
  - commit hash(es)
  - files changed
  - tests run + results
  - any deviations from the spec, and decisions made during coding
- This is part of the Definition of Done — without the trace comment, the task is not complete.
- Commenting on the ticket = appending to the ticket file (Markdown: the `## Dev Comments` section; HTML: the `<section id="dev-comments">`). Never delete prior entries — the trace must be cumulative.
- This makes the coding trace auditable: each ticket records who (which agent) did what, when.

## Documentation Requirement

Documentation is mandatory. Two complementary mechanisms (default: both):
1. **Dedicated documentation task** — a `TASK` per epic titled "Documentation — <epic>" whose scope is the epic's user-facing and developer docs (README sections, API reference, runbook, diagrams).
2. **Embedded documentation** — every ticket carries its own Documentation requirements inside its DoD (update API docs, add docstrings/comments, update the ticket's section 8).

## Ticket Templates

### Project Plan Template

```markdown
# Project Plan — {Project Title}

## 1. Executive Summary
{2-4 sentences: what is being built, why, and how work is split into independent epics}

## 2. Clarified Requirements
### Functional
### Inputs & Outputs
### Edge Cases & Errors
### Performance
### Security
### Dependencies & Tech Stack
### Testing Strategy
### Success Criteria
### Stakeholders

## 3. Architecture & Design Decisions
{High-level architecture and the key decisions with rationale}

## 4. Epic Breakdown
| ID | Epic | Dependencies | Parallel-safe | Suggested Branch | Ticket |
|----|------|--------------|---------------|------------------|--------|
| EPIC-001 | {name} | none | Yes | feat/epic-001-{slug} | tickets/EPIC-001-{slug}.md |

## 5. Dependency Graph & Execution Waves
- Wave 1 (independent, run in parallel): EPIC-001, EPIC-002, ...
- Wave 2 (depends on Wave 1): EPIC-003, ...
- DAG: `EPIC-003 -> EPIC-001`, `EPIC-003 -> EPIC-002`

## 6. Ticket Index
See `tickets/jira-index.md` for the full dashboard.

## 7. Parallel Execution Playbook
1. Spawn one sub-agent per Wave-1 epic, each on its own branch (from the Epic Breakdown table).
2. Instruct each sub-agent to pick tickets from `tickets/`, implement each, and append a Dev Comment trace to each ticket.
3. Merge per-epic branches after Wave-1 DoD checks pass; then start Wave-2.

## 8. Documentation Plan
- A Documentation task is created per epic (tickets/TASK-*... document-...md).
- Every ticket also requires its own docs updates in its DoD.
```

### Markdown Ticket Template

```markdown
---
id: {ID}
type: {Epic | Story | Task}
title: {Title}
summary: {1-2 sentence summary}
epic: {EPIC id, or null}
parent: {parent ticket id, or null}
dependencies: [{none | comma separated ticket ids}]
parallel: {true | false}
suggested_branch: {feat/...}
priority: {High | Medium | Low}
---

# {ID} — {Title}

| Field | Value |
|---|---|
| Type | {Epic/Story/Task} |
| Epic | {EPIC-... or —} |
| Parent | {parent or —} |
| Dependencies | {none, or ticket ids} |
| Parallel-safe | {Yes/No} |
| Suggested branch | `{feat/...}` |
| Priority | {High/Medium/Low} |

## 1. Objective
{What this ticket accomplishes and why it matters}

## 2. Scope
**In scope:**
- {item}

**Out of scope:**
- {item}  ← be explicit to stop agents over-building

## 3. Design Decisions
- {Decision} — {rationale, alternative considered}

## 4. Algorithm / Pseudocode
```text
{dry, implementation-ready pseudocode or ordering of steps}
```

## 5. Definition of Done (Acceptance Criteria)
Each item must be verifiable by the implementing agent:
- [ ] AC-1: {concrete, checkable behavior}
- [ ] AC-2: {concrete, checkable behavior}

## 6. Verification Steps
How the agent proves the DoD (commands, test names, expected output):
1. ...

## 7. Tests Required
- Unit: ...
- Integration: ...
- E2E: ...

## 8. Documentation
- [ ] {docs requirement (API docs, README, inline docs)}
- [ ] {docs requirement}

## 9. Dev Agent Instructions (MANDATORY)
1. Implement exactly per this ticket (sections 1-4).
2. Run the Verification Steps (section 6); every DoD item (section 5) must pass.
3. Write/extend tests per section 7; all must pass.
4. Complete the Documentation tasks (section 8).
5. When done, append a trace comment under "## 10. Dev Comments": commit hash(es), files changed, tests run + results, deviations, decisions made. This is part of the DoD.

## 10. Dev Comments
<!-- Agents append implementation traces here. Never delete prior entries. -->
- ...
```

### HTML Ticket Template

Render the same content as a single self-contained, semantic HTML file so it is readable by humans (styled) and by coding agents (machine-parsable `<meta>` tags + stable section ids):

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>{ID} — {Title}</title>
<meta name="jira:id" content="{ID}">
<meta name="jira:type" content="{Epic|Story|Task}">
<meta name="jira:title" content="{Title}">
<meta name="jira:epic" content="{... or none}">
<meta name="jira:parent" content="{... or none}">
<meta name="jira:dependencies" content="{... or none}">
<meta name="jira:parallel" content="{true|false}">
<meta name="jira:branch" content="{feat/...}">
<meta name="jira:priority" content="{High|Medium|Low}">
<style>
body{font-family:system-ui,-apple-system,sans-serif;max-width:900px;margin:2rem auto;padding:0 1rem;line-height:1.5;color:#1f2328}
pre{background:#f6f8fa;padding:.75rem;border-radius:6px;overflow:auto}
code{background:#f6f8fa;padding:0 .2rem;border-radius:4px}
table{border-collapse:collapse;width:100%}th,td{border:1px solid #d0d7de;padding:.4rem .6rem;text-align:left}
h1{border-bottom:2px solid #d0d7de;padding-bottom:.4rem}
h2{border-bottom:1px solid #d0d7de;padding-bottom:.2rem;margin-top:2rem}
</style>
</head>
<body>
<main>
<header>
  <h1>{ID} — {Title}</h1>
  <dl>
    <dt>Type</dt><dd>{Epic/Story/Task}</dd>
    <dt>Epic</dt><dd>{...}</dd>
    <dt>Parent</dt><dd>{...}</dd>
    <dt>Dependencies</dt><dd>{...}</dd>
    <dt>Parallel-safe</dt><dd>{Yes/No}</dd>
    <dt>Branch</dt><dd><code>{feat/...}</code></dd>
    <dt>Priority</dt><dd>{...}</dd>
  </dl>
</header>
<section id="objective"><h2>1. Objective</h2><p>...</p></section>
<section id="scope"><h2>2. Scope</h2>
  <p><strong>In:</strong></p><ul><li>...</li></ul>
  <p><strong>Out:</strong></p><ul><li>...</li></ul>
</section>
<section id="design-decisions"><h2>3. Design Decisions</h2><ul><li>...</li></ul></section>
<section id="algorithm"><h2>4. Algorithm / Pseudocode</h2><pre><code>...</code></pre></section>
<section id="definition-of-done"><h2>5. Definition of Done (Acceptance Criteria)</h2>
  <ul><li>AC-1: ...</li><li>AC-2: ...</li></ul>
</section>
<section id="verification"><h2>6. Verification Steps</h2><ol><li>...</li></ol></section>
<section id="tests"><h2>7. Tests Required</h2>
  <p>Unit: ...</p><p>Integration: ...</p><p>E2E: ...</p>
</section>
<section id="documentation"><h2>8. Documentation</h2><ul><li>...</li></ul></section>
<section id="dev-instructions"><h2>9. Dev Agent Instructions</h2>
  <p>Implement per sections 1-4; pass all DoD items (5); run Verification (6); pass Tests (7); complete Documentation (8). On completion append a Dev Comment trace below (commit hashes, files changed, tests + results, deviations, decisions).</p>
</section>
<section id="dev-comments"><h2>10. Dev Comments</h2>
  <!-- Agents append implementation traces here. Never delete prior entries. -->
</section>
</main>
</body>
</html>
```

### Jira Index Template

```markdown
# Jira Index — {Project Title}

Status legend: `To Do` / `In Progress` / `Done`

## Epics
| ID | Title | Dependencies | Parallel | Branch | Status |
|----|-------|--------------|----------|--------|--------|
| EPIC-001 | ... | none | Yes | feat/epic-001-{slug} | To Do |

## Stories
| ID | Epic | Title | Dependencies | Parallel | Status |
|----|------|-------|--------------|----------|--------|
| STORY-001-001 | EPIC-001 | ... | none | Yes | To Do |

## Dev Tasks
| ID | Epic | Story | Title | Dependencies | Parallel | Branch | Status |
|----|------|-------|-------|--------------|-----------|--------|--------|
| TASK-001-001-001 | EPIC-001 | STORY-001-001 | ... | none | Yes | feat/epic-001/task-001-... | To Do |

## Parallel Waves
- Wave 1 (run in parallel): EPIC-001, EPIC-002, ...
- Wave 2 (after Wave 1): ...
```

---

## Important Rules
1. **Natural conversation, not a form.** Adapt questions based on answers.
2. **One question at a time.** Never dump a list of questions.
3. **Respect user's time.** If they say "that's enough" or "just do it", stop.
4. **Skip irrelevant aspects.** If an aspect clearly doesn't apply, skip it.
5. **Revisit.** If later answers reveal gaps in earlier aspects, circle back.
6. **The final deliverable is a plan + tickets, never an implementation.** Do not generate, edit, or write application code. The only write operations are the project plan, the Jira ticket files, and the index.
7. **Every ticket must be actionable and self-contained.** A coding agent with no other context must be able to implement from the ticket alone.
8. **Definitions of Done must be verifiable.** No vague "should work" ACs.
9. **Coding trace is mandatory.** Tickets require agents to append Dev Comments; make this explicit in every Dev Task.
10. **Documentation is mandatory.** Either a dedicated docs task per epic, docs requirements in each ticket, or both.
11. **Prefer independent epics.** Design for parallelism; record only genuine dependencies and express them as execution waves.

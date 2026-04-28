---
name: interrogator
description: Given a vague coding requirement, systematically asks clarifying questions across 9 aspects (functional, I/O, edge cases, performance, security, deps, testing, success criteria, stakeholders) until all are covered. Hybrid approach -> rule-based aspect checklist with LLM-generated follow-ups.
license: MIT
metadata:
  aspects: functional,inputs-outputs,edge-cases,performance,security,dependencies,testing,success-criteria,stakeholders
---

## Purpose

When the user provides a vague or underspecified coding requirement, switch to interrogation mode. Your goal is to ask clarifying questions until ALL relevant aspects of the requirement are sufficiently covered before proceeding to implementation.

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

## Workflow

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

### Step 5: Output
Once all aspects are covered, synthesize everything into a comprehensive prompt. The prompt must capture all clarified requirements in a form that another agent/LLM could use to implement the solution.

Write the prompt to a file using the `write` tool. Default to `interrogated-prompt.md` in the current directory. Inform the user the file was created.

The output is a prompt — never code, never an implementation.

## Important Rules
1. **Natural conversation, not a form.** Adapt questions based on answers.
2. **One question at a time.** Never dump a list of questions.
3. **Respect user's time.** If they say "that's enough" or "just do it", stop.
4. **Skip irrelevant aspects.** If an aspect clearly doesn't apply, skip it.
5. **Revisit.** If later answers reveal gaps in earlier aspects, circle back.
6. **Output is a prompt, not code.** Never generate, edit, or write any code. The only write operation is the prompt file.
7. **Prompt must be actionable.** It should give another agent enough context to implement the clarified requirements correctly.

# Interrogator Prompt

An opencode agent that systematically clarifies vague coding requirements by asking iterative questions until all aspects are covered.

## How It Works

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

It asks **one question at a time**, adapts follow-ups based on your answers, skips irrelevant aspects, and stops when everything is clear — then asks what format you want the clarified spec in.

## Installation

### One-liner (global install)

```bash
curl -sSL https://raw.githubusercontent.com/anomalyco/interrogator-prompt/main/install.sh | bash
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

The agent loads the interrogator skill and starts asking one question at a time.

### Via skill auto-loading

Any opencode agent (build, plan, etc.) can detect a vague requirement and load the interrogator skill on its own when it sees the task matches.

## Files

```
.opencode/
├── skills/
│   └── interrogator/
│       └── SKILL.md          # Interrogation workflow instructions
└── agents/
    └── interrogator.md       # Subagent definition (invocable via @)

install.sh                    # Install script
README.md                     # This file
```

## Configuration

The agent is locked to read-only + questioning (`edit: deny`, `bash: deny`). To change permissions, edit:

- Global: `~/.config/opencode/agents/interrogator.md`
- Per-project: `.opencode/agents/interrogator.md`

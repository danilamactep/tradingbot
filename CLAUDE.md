# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

This is a **new project** (tradingbot) in early planning/setup. No application source code exists yet — only the BMAD AI development framework has been initialized.

## BMAD Framework

This project uses [BMAD](https://github.com/bmad-code-org) (v6.0.4) — an AI-assisted methodology for structured software development. All BMAD infrastructure lives in `_bmad/`.

### Key Directories

| Path | Purpose |
|------|---------|
| `_bmad/` | BMAD framework (do not edit manually) |
| `_bmad/_config/` | Agent and IDE customizations |
| `_bmad/_memory/` | Persistent agent memory and config |
| `_bmad-output/planning-artifacts/` | Generated PRDs, briefs, architecture docs |
| `_bmad-output/implementation-artifacts/` | Generated epics, stories, sprint plans |
| `docs/` | Project knowledge base (referenced by agents) |

### BMAD Configuration

- **Project name:** tradingbot
- **User:** Daniel
- **Skill level:** intermediate
- **Output language:** English

### Using BMAD Agents

Invoke agents via the Claude Code skills system (e.g., `/bmad-agent-bmm-pm` for the PM agent). Each agent has a customizable config at `_bmad/_config/agents/<agent-name>.customize.yaml` where you can add persistent memories, custom menu items, or override persona.

### Workflow Phases

BMAD organizes work in phases:
1. **Analysis** — market/domain/technical research, product brief (`_bmad/bmm/workflows/1-analysis/`)
2. **Planning** — PRD, UX design, architecture (`_bmad/bmm/workflows/2-plan-workflows/`)
3. **Specification** — epics, stories, sprint planning (`_bmad/bmm/workflows/3-*/`)
4. **Implementation** — dev stories, code review, sprint tracking (`_bmad/bmm/workflows/4-implementation/`)

To start: run the product brief workflow or PRD workflow to generate planning artifacts before any code is written.

## Development Practices

These apply during all implementation work.

### TDD — Test-Driven Development

- **Always present a test summary before implementing.** Brief descriptions only — what each test validates, no inputs/outputs unless critical to understanding the purpose.
- **Wait for Daniel's agreement** on the test summary before writing any code.
- Same cycle for modifications: test summary → discuss → agree → implement.

### Coding Standards

- **Plan first**: present implementation plan → discuss → agree → implement.
- **Readability**: methods should be 20–50 lines. Longer requires explicit justification.
- **Composition**: small independent components that compose into full functionality.
- **Single responsibility**: one concern per component, separation of concerns throughout.

### Regression Testing

- Golden input sets are **static files committed to the repo**.
- Never modify a golden set without explicit discussion.

## Session Hygiene Rules

These apply to all sessions — BMAD elicitation, planning, coding, and design.

1. **Only write agreed decisions — and ask when unclear.** After any meaningful unit of work completes (elicitation method, test plan, architecture decision, implementation plan), write confirmed decisions to the active artifact. Do not write proposals or items still under discussion. If it is unclear whether something was confirmed, ask explicitly before writing or dropping it.
2. **Suggest `/clear` after writing.** Once findings are written to file, suggest Daniel run `/clear` to free context. Do not start the next unit of work until this happens.
3. **Treat transitions as hard stops.** "I'm good now," "let's move on," or equivalent signals completion — do not start the next topic until findings are written and `/clear` is suggested.
4. **"Agree to everything else" means confirmed.** When Daniel says he agrees to everything except the items he explicitly commented on, treat all non-commented items as confirmed and write them to file.

### When to suggest `/clear`

- After each elicitation method (BMAD workflows)
- After test plan is agreed (coding)
- After architecture or design is agreed (coding)
- After implementation plan is agreed (coding)
- After any section of the PRD, architecture doc, or story is finalized

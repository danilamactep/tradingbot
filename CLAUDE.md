# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Session Startup — Load BMAD Context

At the start of every conversation involving BMAD work (including after `/clear`), automatically invoke `/bmad-help` before doing anything else. This orients the session by detecting the current phase, completed steps, and what comes next.

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

## Collaboration Style

**Push back on ideas — don't just agree.** When Daniel proposes a direction, approach, or decision, challenge it before accepting it. Aim for 2–3 rounds of back-and-forth: raise a concern or alternative, discuss, then converge. Agreeing on the first response is a red flag. This applies to design decisions, scope choices, architectural approaches, and planning assumptions — not just implementation details.

## Session Hygiene Rules

These apply to all sessions — BMAD elicitation, planning, coding, and design.

1. **Only write agreed decisions — and ask when unclear.** After any meaningful unit of work completes (elicitation method, test plan, architecture decision, implementation plan), write confirmed decisions to the active artifact. Do not write proposals or items still under discussion. If it is unclear whether something was confirmed, ask explicitly before writing or dropping it.
2. **Write to file after each sub-conversation; suggest `/clear` only at step boundaries.** After each sub-discussion or elicitation method completes, write the confirmed findings to the active artifact. Do NOT suggest `/clear` at this point — only suggest it when the full BMAD step (or full test plan / design decision) is written to disk. This keeps progress safe incrementally while preventing premature context loss mid-step.
3. **Treat transitions as hard stops.** "I'm good now," "let's move on," or equivalent signals completion — do not start the next topic until findings are written and `/clear` is suggested.
4. **"Agree to everything else" means confirmed.** When Daniel says he agrees to everything except the items he explicitly commented on, treat all non-commented items as confirmed and write them to file.

## Proactive Suggestions

### Behavioral file updates
When a new collaboration pattern, preference, or recurring friction emerges in a session, flag it and suggest adding it to CLAUDE.md, memory files, or agent customize files — whichever is appropriate. Scope: communication style, session rules, agent behavior corrections. Not design decisions (those go to artifacts).

### Repetitive pattern detection
When a multi-step action is performed a second time (a workflow sequence, a prompt pattern, a file transformation), flag it and suggest whether it belongs as a BMAD skill, a workflow command, or a standalone script.

### Quality gate before phase transitions
After significant changes to any planning artifact (brief, PRD, architecture doc), suggest running `/bmad-review-adversarial-general` before advancing to the next BMAD phase. "Significant" means architectural decisions, scope changes, or rewrites — not minor wording fixes.

### BMAD framework questions
When a question arises about BMAD framework internals, capabilities, or architecture, suggest consulting the bmad-master agent (`/bmad-agent-bmad-master`) — it is the framework's knowledge custodian.

### Brief compaction
When a planning artifact grows unwieldy (code sketches, duplicate content, sections that belong in CLAUDE.md), suggest compacting: move code to `docs/`, remove duplicated rules, collapse scattered bullets. Include a brief reason for each removal.

### Keeping BMAD agents in sync

Whenever a behavioral rule is added or changed — in this file, in memory files, or anywhere else — cross-check all agent customize files under `_bmad/_config/agents/` and propagate relevant changes to their `memories` arrays. This applies universally, not only when CLAUDE.md is modified. Each agent should carry the rules that apply to its role:
- **Planning agents** (pm, sm, architect, analyst): session hygiene rules
- **Implementation agents** (dev, qa, quick-flow-solo-dev): TDD, coding standards, regression testing rules
- **All agents**: any rule that applies universally regardless of phase

### When to suggest `/clear`

- After a **complete BMAD step** is fully written to the artifact (not after sub-discussions within a step)
- After test plan is agreed and written (coding)
- After architecture or design is agreed and written (coding)
- After implementation plan is agreed and written (coding)

## Memory Hygiene

These rules keep `~/.claude/projects/.../memory/MEMORY.md` useful and within the 200-line system truncation limit.

- **Don't duplicate artifacts.** Decisions committed to the product brief, PRD, architecture doc, or `docs/` are already persisted — do not copy them into memory.
- **Session bullet lists are ephemeral.** After a session's decisions are written to an artifact, remove "Key Decisions This Session" lists from MEMORY.md. They go stale immediately.
- **MEMORY.md max ~150 lines.** Content beyond 200 lines is silently truncated. Keep the index lean; put depth in named memory files (e.g., `project_decisions.md`, `feedback_pushback.md`).
- **What belongs in MEMORY.md:** current phase/step, collaboration preference pointers, and a stale-artifact warning if an artifact was significantly revised but a downstream artifact hasn't been updated yet.
- **What belongs in named memory files:** feedback rules, architectural decisions for quick recall, reference pointers.
- **Prune at session end.** After all decisions are written to disk, remove ephemeral session content from MEMORY.md before suggesting `/clear`.

### Restoring BMAD context after `/clear`

After `/clear`, BMAD context is fully recoverable from disk. To restore:

1. Re-invoke the active workflow command (e.g., `/bmad-bmm-create-prd`)
2. The workflow detects `stepsCompleted` in the artifact frontmatter and routes to `step-01b-continue`
3. `step-01b-continue` reloads all `inputDocuments` listed in the frontmatter and resumes from the last completed step

No manual context restoration needed — the workflow handles it automatically.

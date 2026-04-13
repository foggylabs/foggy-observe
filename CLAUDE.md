# CLAUDE.md — Foggy Observe

## What is this?

Foggy Observe is an AI coding skill that generates personalized analytics tracking for any product. The core skill is `/observe` (`.claude/skills/observe.md`). Skills share a common artifact: `tracking-plan.md` (the semantic layer).

## Repository Structure

```
foggy-observe/
  .claude/skills/
    observe.md                # /observe — full flow skill (setup + plan + tracking code)
  docs/
    tracking-plan-spec.md     # Tracking plan format specification v0.1
    design-doc.md             # Product design document
    deep-research.md          # Market research
  BACKLOG.md                  # Skill backlog and priorities
```

## Development

Skills are prompt engineering. Each skill file contains instructions for the AI coding tool with templates for generated output.

## Conventions

- Shared artifact: `tracking-plan.md` in the user's repo root
- Plan format spec: `docs/tracking-plan-spec.md`
- All free skills are open source. `/observe-diagnose` is the paid product.

## Backlog

Read BACKLOG.md for current priorities. All new work items go there.

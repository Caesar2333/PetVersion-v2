# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

>#### 这是一个windows项目，处在powershell环境中，全都是使用powershell指令。不准使用Bash执行powershell指令，这是不允许的。

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## Project Identity

This project is a modern frontend application built with:

- Next.js App Router
- React
- TypeScript
- Tailwind CSS
- shadcn/ui
- Motion for React
- Zustand

Optional tools:

- Anime.js for isolated complex visual effects only
- lucide-react for icons
- React Hook Form + Zod for complex forms and validation
- Impeccable for frontend design shaping, critique, audit, polish, hardening, and anti-AI UI review
- make-interfaces-feel-better for micro-interaction, handfeel, and final UI polish review

The goal of this stack is to reduce unnecessary custom CSS, keep UI implementation consistent, avoid generic AI-generated frontend patterns, and make the project easy for AI agents and humans to maintain.



# Agent Workflow Rules

Use Matt Pocock skills as workflow controllers. Do not blindly use all skills at once. Pick the smallest skill that matches the current phase.

### Required skill routing

- Use `/setup-matt-pocock-skills` once per repository before using engineering skills.
- Use `/grill-with-docs` before changing product behavior, domain concepts, state machines, architecture, or UI interaction rules.
- Use `/zoom-out` when the affected code area is unfamiliar or when the system relationships are unclear.
- Use `/prototype` before committing to uncertain UI, animation, state-machine, or data-model designs.
- Use `/to-prd` only after the requirement is sufficiently discussed and should be turned into a formal PRD.
- Use `/to-issues` after a PRD or implementation plan exists, and split work into thin vertical slices, not horizontal layers.
- Use `/tdd` for implementing stable logic, state machines, parsers, data transforms, regression-prone behavior, and testable feature slices.
- Use `/diagnose` for bugs, regressions, broken behavior, flaky behavior, or performance problems.
- Use `/improve-codebase-architecture` periodically after several feature slices, or when the codebase becomes hard to navigate.
- Use `/handoff` before ending a long session or transferring work to another agent.

### Communication and writing

- Use `/write-a-skill` when turning repeated workflow knowledge into a reusable skill.

### Safety rules

- Do not modify unrelated files while diagnosing a narrow bug.
- Do not rewrite CSS, layout, or architecture unless the active issue explicitly requires it.
- Before code edits, state the exact files likely to change and why.
- Prefer small vertical slices over large sweeping changes.
- Every implementation slice must have a verification step: typecheck, test, build, browser check, or a manual verification script.
- If the requirement is unclear, do not guess. Start `/grill-with-docs` and ask one question at a time.
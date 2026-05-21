# ADR 0002: Role-Based Action Mapping

## Status

Accepted (2026-05-21)

## Context

A pet's spritesheet contains animation rows. In pet.json, each row is identified by an **action name** chosen by the pet author (e.g. `running-left`, `waving`, `walk-left`). The state machine needs to decide which action to play for a given situation.

The reference project (PetExtension) hardcoded 9 state names directly in `HATCH_PET_SPEC` and the state machine. Every pet had to use exactly those names. This works for a closed set of built-in pets but blocks:
- Imported pets with different naming conventions
- Legacy Codex/hatch-8x9 pets that use names like `running-left` instead of `move-left`
- Multi-language pet packs

## Decision

Introduce a **semantic role** layer between the state machine and the pet's actual actions.

- The state machine only knows 9 fixed **roles**: `idle`, `move-left`, `move-right`, `greet`, `working`, `waiting`, `success`, `failed`, `special`.
- Each pet provides an **actionMap**: role → actual action name.
- The SpritePlayer exposes `playRole(role)` which resolves through the actionMap internally.
- A separate `playAction(actionId)` method exists for debug/preview of unmapped actions.

Resolution priority:
1. Explicit `actionMap` in pet.json
2. Action name exactly matches a role name (automatic)
3. Built-in alias table (e.g. `waving` → `greet`, `running-left` → `move-left`, `jumping` → `success`)
4. Manual user mapping in import UI
5. Fallback to `idle`

Unmapped actions become `extraActions` — preserved but not used by the state machine.

## Alternatives Considered

### Hardcoded action names (reference project approach)
- Simplest, but prevents importing pets from different ecosystems
- Rejected: v2 explicitly needs multi-pet and import support

### Action names as the sole identifier (no role layer)
- Pet authors must use exact names
- Rejected: too fragile, breaks legacy pets, no i18n path

### Full semantic annotation in pet.json (every action declares its "meaning")
- Most flexible but burdens pet authors
- Rejected for v2: overengineered for 9 actions

## Consequences

- State machine is decoupled from pet naming — any pet can work if its actions are mapped to the 9 roles
- Import flow must include an action mapping confirmation step
- The alias table needs maintenance as new naming conventions appear
- Player page must show both role and resolved action for debugging

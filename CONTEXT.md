# CONTEXT.md — petExtension-v2

Domain glossary for a Chrome browser pet extension (Manifest V3). Terms are resolved; implementation details belong elsewhere.

## Core Concepts

- **Pet**: A character animated on top of any webpage via a Shadow DOM overlay. Defined by a `pet.json` manifest plus a spritesheet image.
- **Spritesheet**: A single image containing all animation frames in a grid layout. Each column is a frame; each row is an action.
- **Cell**: One frame within the spritesheet. Dimensions come from the normalized manifest (`cellWidth` × `cellHeight`).

## Pet Manifest (pet.json)

- **Action**: A named animation sequence defined by `row` (spritesheet row index) and `frameCount`. The action name is the author's choice (e.g. `running-left`, `waving`).
- **Semantic Role**: One of 9 fixed standard roles the state machine outputs: `idle`, `move-left`, `move-right`, `greet`, `working`, `waiting`, `success`, `failed`, `special`. Never an action name.
- **actionMap**: Mapping from semantic role → actual action name. Resolved in priority: explicit `actionMap` in pet.json → exact name match → alias table → manual user mapping → fallback to `idle`.
- **Playback Rule**: Whether an action loops or plays once, and what to do after a one-shot completes. Defaults are built-in per role; a pet.json may override with `loop` and `next`.
- **extraActions**: Actions in pet.json not mapped to any semantic role. Retained for preview/debugging but not used by the state machine.
- **Preset**: A named template (`codex-9x9` or `hatch-8x9`) that supplies missing spritesheet dimensions for legacy pets. Only used when the manifest lacks full spritesheet fields.
- **Normalized Manifest**: The runtime representation after all fallbacks are resolved — complete spritesheet spec, full action specs with playback rules, and a complete actionMap.

## Spritesheet Resolution

Priority when loading a pet:
1. Full spritesheet fields in pet.json → use directly
2. Partial fields + `preset` declared → fill from preset
3. No preset, but image dimensions match a known preset → auto-detect
4. Ambiguous → user selects preset in import UI
5. Nothing works → manifest invalid, reject

Known presets: `codex-9x9` (256×288 cells, 9×9 grid), `hatch-8x9` (192×208 cells, 8×9 grid).

## Runtime Architecture

- **PetStateMachine**: Pure event-driven, no internal timers. Accepts semantic hooks, resolves priority, outputs a role. One-shot actions lock out non-drag hooks until completion or drag interruption. Must not know about cooldowns or raw browser events.
- **ActivityTracker**: Listens to raw browser events (keydown, input, click, scroll, wheel, mousemove, selectionchange, visibilitychange, focus, blur). Aggregates and throttles them into semantic hooks: `userActivity`, `activitySettled`, `userInactive`.
- **InteractionGate**: Thin input filter. Maintains cooldown timestamps for one-shot roles (`greet`, `success`, `special`, `failed`). Aggregates multiple raw triggers (dblclick, contextmenu, longPress, hoverDwell) into a single `specialTrigger` hook. Does NOT decide state priority.
- **SpritePlayer**: Renders spritesheet frames via CSS background-position. Accepts atlas + actions + actionMap from the normalized manifest. Exposes `playRole(role)` for the state machine and `playAction(actionId)` for the debug player. Frame duration is uniform (default 150ms), scaled by `playbackRate`. One-shot completion fires `onComplete`.
- **DragController**: Pointer Events on the pet shell. Emits `dragStart`, `dragMove` (left/right), `dragEnd` hooks.
- **Bounds**: Clamps pet position to viewport. Uses the current pet's `cellWidth`/`cellHeight` × `scale` — never hardcoded constants.

## State Priority

1. Dragging (`move-left` / `move-right`)
2. Boundary failure (`failed` — triggered by viewport resize pushing pet out of bounds)
3. One-shot (`greet`, `success`, `special`, `failed`)
4. Working
5. Waiting
6. Idle

## Storage

- **chrome.storage.local**: Settings, pet list, normalized manifests (metadata only, no blobs).
- **IndexedDB**: Spritesheet image blobs, keyed by `assetKey`. Object URLs created on load, revoked on unload.
- **extensionApi adapter**: Abstracts `chrome.*` APIs so UI pages can develop with HMR and mock data outside the extension environment.

## Pages

- **popup.html**: Daily control panel. Tab-based layout with two tabs: "Control" (enable/disable, scale, speed, reset position) and "Pets" (built-in + imported pet selection grid, navigate to player/import). React + Tailwind + shadcn/ui.
- **player.html**: Animation preview and state machine debug console. Select pet, trigger roles, inspect frames, adjust speed/scale, view actionMap. React + Tailwind + shadcn/ui.
- **import.html**: Import wizard. Upload pet.json + spritesheet, auto-detect preset, show auto-generated actionMap, let user correct mappings, save normalized manifest. React + Tailwind + shadcn/ui.

## Content Script Injection

- Conservative strategy: `document_idle`, top window only, blocked URL patterns (chrome://, edge://, about:, chrome-extension://, Chrome Web Store).
- Shadow DOM overlay with `petOverlay.css` (isolated styles). Pet interaction does not leak to the host page.

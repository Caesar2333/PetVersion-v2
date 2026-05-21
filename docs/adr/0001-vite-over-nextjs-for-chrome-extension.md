# ADR 0001: Vite over Next.js for Chrome Extension

## Status

Accepted (2026-05-21)

## Context

The project's CLAUDE.md defines the stack as Next.js App Router + React + TypeScript + Tailwind CSS + shadcn/ui. However, the deliverable is a Chrome Extension (Manifest V3), which requires:

- A background service worker (runs in a Service Worker context — no DOM, no React, no SSR)
- Content scripts injected into arbitrary web pages
- Static HTML popup/player/import pages

Next.js App Router cannot produce a background service worker or content script. SSR and RSC have no meaning in an extension context. The only viable Next.js output would be `next export` (static HTML), but that loses the framework's primary value while adding build complexity.

## Decision

Use **Vite** as the bundler (manual `rollupOptions.input` with multiple entry points), React for UI pages, and Tailwind CSS + shadcn/ui for styling. Do not use Next.js.

The reference project (PetExtension) uses this exact approach and it works stably for Chrome Extension MV3.

## Alternatives Considered

### Next.js static export + separate SW bundler
- Would need two build systems (Next.js for pages, something else for SW/content)
- `next export` is deprecated in recent Next.js versions
- Adds a framework dependency that provides zero value for this use case

### CRXJS Vite plugin
- Adds HMR for content scripts (nice but complex)
- Has had compatibility issues with newer Vite versions
- Reference project works fine without it

## Consequences

- Popup/player/import pages use React + Tailwind + shadcn/ui — same UI stack as CLAUDE.md intended, just without the Next.js framework
- Single build system (`vite build`) produces all extension artifacts
- No SSR/RSC/API routes — none are needed for a browser extension
- HMR for UI pages via `vite dev` + mock extension API adapter

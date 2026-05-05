# Agent Guide

<!--
  Lean by design. Per the ETH Zurich AGENTS.md study (Feb 2026), architectural
  overviews and directory listings increase inference cost ~20% without
  improving task success. This file contains only non-inferable rules.
  Richer context lives in docs/ and is read on demand.
-->

## Commands

- Dev server: `pnpm dev`
- Test: `pnpm test`
- Typecheck: `pnpm tsc --noEmit`
- DB push (dev): `pnpm db:push`
- DB generate migration: `pnpm db:generate`
- DB run migrations: `pnpm db:migrate`
- Refactor day: `pnpm refactor-day` (jscpd + knip)

## Stack

- Next.js (latest stable, App Router only — no Pages Router)
- TypeScript strict mode
- Tailwind CSS + shadcn/ui (primitives in `components/ui/`)
- SQLite + Drizzle ORM (`better-sqlite3`)
- Zod validates all server action inputs and env vars
- Vitest for tests
- pnpm

## File placement (one rule)

New code goes in the closest private folder to the route that uses it, unless it belongs in `src/db/` or `src/lib/`.

- New UI → `app/<route>/_components/`
- New logic → `app/<route>/_lib/`
- New server action → `app/<route>/actions.ts` (one file per route, multiple exports)
- New DB column or table → `src/db/schema.ts`
- New env var → `src/lib/env.ts`

## Promotion to `src/`

Do not promote speculatively. Promote ONLY when one of:

- A second route already imports the file
- `jscpd` flagged it as duplicated
- The file exceeds ~300 lines with clear sub-pieces

Promotion targets:

- Generic utility → `src/lib/`
- Real business logic → `src/domain/` (create folder on first promotion)
- Reused custom UI → `src/ui/` (shadcn primitives stay in `components/ui/`)

## Naming

- All files kebab-case
- Component files use suffix tags: `-table.tsx`, `-form.tsx`, `-modal.tsx`, `-panel.tsx`, `-chart.tsx`, `-editor.tsx`, `-grid.tsx`, `-sidebar.tsx`, `-button.tsx`
- Type files: `-record.ts` for row shapes, `-status.ts` for enums
- `page.tsx`, `layout.tsx`, `actions.ts` are Next.js conventions — exact names, no suffix
- Folder names: 3-word cap; verb-led for workflow routes; noun-led for resource routes; no bucket verbs (`manage-`, `handle-`); strip project-context tokens
- Migrations: numbered prefix, kebab body

## Collision-risk files (one agent at a time)

These get touched by every feature. Coordinate edits:

- `app/layout.tsx` — root nav and providers
- `src/db/schema.ts` — every feature adds tables/columns
- `src/lib/env.ts` — every feature with config adds vars
- `middleware.ts` — if present
<!-- skill appends domain orchestration files here as they get created on refactor day -->

## Counterintuitive

- Server actions live in the route folder (`app/<route>/actions.ts`), not a global `actions/` folder.
- No `src/features/`. **Ever.** The folder under `app/` IS the feature.
- Do not pre-create empty folders.
- When unsure between two locations, pick the smaller blast radius (the route folder).
- One `actions.ts` per route — do not split into one file per action.
- Do not introduce a different ORM. Drizzle only.
- Do not add `src/components/`, `src/hooks/`, `src/utils/`, `src/services/`, or `src/types/`. These get rejected.

## Where to find more context

- `docs/app-overview.md` — what this app does, pages, user flows
- `docs/ux-decisions.md` — specific UX decisions made during scaffolding (read when adding features that touch them)
- `docs/feature-roadmap.md` — V1 scope and future plans
- `prototype/` — the original visual prototype this project was built from

Do not paste content from these files into prompts. Reference them by path so agents read on demand.

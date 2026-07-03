---
name: pdlc-5-build
description: PDLC pipeline stage 5 — DB schema design and implementation of the 3 core features (Next.js + Supabase). Use this skill whenever the user is ready to build, e.g. "이제 구현하자", "DB 스키마 설계해줘", "ERD 만들어줘", "MVP 개발", "Supabase 테이블 만들자", "Next.js로 만들어줘", "핵심 기능 구현", or when working code must be produced after spec and plan are confirmed. Target is a locally runnable MVP.
---

# Stage 5: DB Schema + 3 Core Features (Next.js + Supabase)

Turn documents into working software. Scope: a **locally runnable MVP** — a DB reproducible from migrations plus 3 core features working in a browser. Deployment belongs to the stage-7 redeploy plan. The trap in this stage is scope creep — when an unspecced feature starts looking attractive, write it to the backlog and don't touch it.

## Inputs (required)

- `docs/03-spec/functional-spec.md` — flows, validation, error cases for the 3 FEATs
- `docs/03-spec/screen-design.md` — SCR layouts and the 4 states
- `docs/03-spec/policies.md` — data-lifecycle and permission policies drive schema design
- `docs/04-plan/milestones.md` — confirms which 3 features were selected
- `docs/pipeline-state.json` — set stage 5 `in_progress` on start

If invoked without specs: code can still be written, but stage-6 QA loses its verification baseline. Say so, and at minimum obtain behavior definitions for the 3 features.

## Outputs

- `docs/05-build/db-schema.md` — design doc + Mermaid ERD
- `docs/05-build/build-notes.md` — scope, run instructions, decision records (ADR-lite)
- Project root: Next.js app + `supabase/migrations/*.sql`

Read `references/stack-conventions.md` for tech conventions and the document templates.

## Workflow

### 1. Design the DB schema (before any code)

- Derive entities to support all FEATs (including backlog), but **create tables only for what the 3 features need**. Future-proofing goes in the design doc's "향후 확장" section, not in DDL.
- Per table: column definitions (type, constraints, defaults), indexes with their justifying query, policy-document reflections (soft delete → `deleted_at`, history retention → audit columns).
- Draw the ERD as Mermaid `erDiagram`.
- **Define RLS (Row Level Security) per table at design time.** "Who can read/write/delete this row" is the policy document's permission rules translated into SQL. RLS bolted on later always leaks.
- After user review, write migration SQL (`supabase/migrations/`).

### 2. Project setup

Follow the setup procedure in `references/stack-conventions.md` (App Router, TypeScript, directory layout, env vars, Supabase client init). If the repo already has code, respect the existing structure and record divergences from convention in build-notes.

### 3. Implement features (one at a time, vertically)

Never open all 3 features at once. Each feature is a vertical slice: DB → server logic → UI → the 4 states (empty/loading/error/no-permission) → manual check. While implementing:

- **Spec back-references**: maintain a "FEAT-001 → implementation file paths" mapping table in build-notes (not code comments). Stage-6 QA finds its test targets through this table.
- **When implementation must diverge from spec** (tech constraint, better approach found): don't change silently. Tell the user, record the decision in build-notes and `pipeline-state.json` `decisions`, and update stage-3 docs or flag them `stale`.
- **Validation lives on the server**: client validation is UX; the trust boundary is server actions/API. Translate the spec's validation table into server code.
- Error messages use the exact copy finalized in the spec.

### 4. Verify

- Re-run migrations from an empty DB to confirm reproducibility.
- Confirm `npm run build` passes (no ignoring type errors).
- Manually check each feature's main flow + at least 1 error case against the spec; record results in build-notes (formal QA is stage 6).

### 5. Wrap up

- Finish build-notes: run instructions (clone → browser, follow-along level), implemented/deferred scope, decision records.
- Update `docs/pipeline-state.json`: stage 5 `done`, outputs, `current_stage: 6`.
- Propose stage 6 (pdlc-6-qa), noting the FEAT ↔ file mapping table is QA's input.

## Language rule

Docs in Korean with English terms glossed. Code, comments, commit messages, identifiers in English. UI copy in Korean (exactly as specced).

## Completion criteria (self-check)

- [ ] ERD + per-table RLS policies in db-schema.md
- [ ] Migrations reproduce from an empty DB
- [ ] 3 features work in a local browser (all 4 states included)
- [ ] `npm run build` passes
- [ ] FEAT ↔ file mapping, run instructions, decision records in build-notes
- [ ] No unspecced features added (or added ones recorded in decisions)

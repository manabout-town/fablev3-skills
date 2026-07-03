---
name: pdlc-seed-data
description: PDLC pipeline add-on (right after stage 5) — generate realistic dummy/seed data, load it into the DB, and smoke-test the real logic against it. Guarantees the app never runs on an empty database when QA (stage 6) executes, when a test build is launched, or when stage-8 demo GIFs are recorded. Use this skill whenever the user says "더미데이터 넣자", "시드 데이터 만들어줘", "테스트 데이터로 돌려보자", "데이터 채워서 실행해보자", "빈 화면인데 데이터 좀", or before stage 6 / stage 8 whenever the DB is empty or unreproducible.
---

# Seed Data + Logic Smoke Test (더미데이터 검증)

Stage 5 proves the code compiles and the features work on whatever ad-hoc rows the developer happened to click into existence. That is not enough: QA, the adversarial cycle, and stage-8 GIF recording all need a **reproducible, realistic dataset** — and the logic needs to have actually run against it at least once. This add-on produces that dataset and performs the smoke test.

Two rules define the scope:
1. **Dummy data must be realistic in shape, fake in content.** Korean-plausible names, real-looking dates and amounts, but zero real personal data.
2. **The test is a real run, not a review.** Seed the DB, launch the app, and drive each feature's logic through the UI (or server actions). Reading the code and declaring it fine does not count.

## Position in the pipeline

Runs **immediately after stage 5 (build), before stage 6 (QA)**. Downstream consumers:
- **Stage 6 QA** executes TCs starting from `db reset` + seed (reproducible preconditions).
- **pdlc-adversarial** attacks a seeded build, not an empty one.
- **Stage 8 GIF recording** resets to this seed before every scene — GIFs must never show accidentally-empty screens (the deliberate empty-state scene uses the designated empty account below).

If the user reaches stage 6 or stage 8 and this add-on never ran, route here first.

## Inputs (required)

- `supabase/migrations/*.sql`, `docs/05-build/db-schema.md` — tables, constraints, RLS the data must satisfy
- `docs/03-spec/functional-spec.md`, `docs/03-spec/policies.md` — what states and edge values matter
- `docs/02-requirements/` personas — seed users mirror the personas
- `docs/pipeline-state.json` — add a top-level `seed_data` block (`{"status": "in_progress"}`) on start; do not touch numeric `stages` keys

## Outputs

- `supabase/seed.sql` (or `scripts/seed.ts` when logic-heavy generation is needed) — idempotent, re-runnable
- `docs/055-seed/seed-data.md` — dataset design (what accounts/rows exist and why) + smoke-test log
- A one-command reset path documented in build-notes: `supabase db reset` → migrations + seed → known state

## Workflow

### 1. Design the dataset (before writing any rows)

Derive from the spec, not from imagination:

- **Accounts**: one seed user per persona (matching role/permission differences), plus one **designated empty account** (`empty@demo.test`) that owns zero data — this is the stage-8 empty-state scene, kept deliberately, not accidentally.
- **Per FEAT**: enough rows to exercise every state the spec defines — normal rows, boundary rows (max length, 0 quantity, today/expiry-date edges, soft-deleted `deleted_at` rows), and rows that trigger each specced error/validation path.
- **Volume**: lists and pagination must look real — 15–30 rows where the UI shows a list, not 2.
- **RLS coverage**: data owned by *different* users so permission boundaries are actually testable (user A must not see user B's rows).

Write the design table into `seed-data.md` (account / table / row profile / which FEAT·state it serves) and get user confirmation before implementing.

### 2. Implement the seed

- Idempotent: running twice must not duplicate or crash (use fixed UUIDs / `on conflict`).
- Must pass through the same constraints and RLS as production writes — a seed that bypasses validation hides schema bugs.
- Verify reproducibility: `supabase db reset` from empty → migrations → seed → identical state, twice in a row.

### 3. Smoke-test the logic (the point of this skill)

Launch the app against the seeded DB and drive it:

- Per FEAT: happy path end-to-end + at least 1 specced error path, through the real UI or server actions.
- Confirm seeded data actually **renders** where the spec says it should (lists populated, detail pages resolve, computed values correct against hand-checked expectations).
- Confirm the RLS boundary: log in as user A, verify user B's rows are absent.
- Log every check in `seed-data.md` as SMK-### (target FEAT / action / expected / observed / pass·fail).

This is a smoke test, not stage-6 QA — no TC formality, but every SMK failure is real: fix stage-5 code (record the decision) or, if the seed itself was wrong, fix the seed. Zero open SMK failures is the exit condition.

### 4. Wrap up

- Update `pipeline-state.json`: `seed_data.status: done`, account count, row counts, SMK pass rate, one-line summary.
- Note in build-notes: the reset command and the seed-account credential table.
- Propose stage 6 (pdlc-6-qa), pointing out that TC preconditions can now assume the seeded state.

## Cautions

- Never seed real names, emails, phone numbers, or copied production data — GIFs of this data go public in stage 8.
- Don't let seed writing turn into feature building; if the seed exposes a missing feature, that's a backlog note, not scope.
- Keep the seed in version control next to migrations — a dataset that lives only in someone's local DB is not reproducible.
- Output language rule: deliverables in Korean, terms glossed in English on first mention.

## Completion criteria (self-check)

- [ ] Seed covers every persona + designated empty account
- [ ] Every FEAT has rows for normal / boundary / error states; lists look populated (15+ rows)
- [ ] `supabase db reset` reproduces the identical state twice
- [ ] SMK log complete, all pass (or failures fixed and re-run)
- [ ] RLS boundary verified across at least 2 seed users
- [ ] `seed_data` block updated in pipeline-state.json

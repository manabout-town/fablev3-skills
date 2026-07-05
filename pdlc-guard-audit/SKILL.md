---
name: pdlc-guard-audit
description: PDLC pipeline add-on (after stage 5 and pdlc-seed-data, before stage 6) — defensive-code audit. Systematically enumerates every guard point the spec implies (e.g., a logged-out user must not be able to comment or like), verifies each is enforced server-side in the actual build, and fixes the missing ones. Use this skill whenever the user asks for defensive-code checks, e.g. "방어 코드 점검하자", "방어적 코드 짜야 하는 데 있는지 봐줘", "비로그인 차단 확인", "권한 체크 빠진 데 없나", "가드 감사", or before QA when the build's guard coverage is unverified.
---

# Defensive Code Audit (방어 코드 감사)

Every state-changing action in the app has preconditions the spec implies but the code may not enforce: "비로그인 사용자는 댓글·좋아요를 할 수 없다", "남의 글은 수정할 수 없다", "마감된 예약은 취소만 가능하다". This add-on turns those implications into an explicit **guard inventory**, then verifies each guard against the running build — not by reading the code and nodding, but by attempting the forbidden action and confirming the server refuses.

The core stance: **UI hiding is not a guard.** A button that disappears when logged out is UX; the guard is the server rejecting the request when it arrives anyway. Every check in this audit must hold at the trust boundary (server action / API / RLS), because that's where a curious user with DevTools — or stage-6 QA's negative TCs, or Team Black — will hit it.

## Position in the pipeline

Runs **after stage 5 (build) and after `pdlc-seed-data`**, before stage 6 (QA). Seed accounts (multiple users + roles) are what make permission guards testable. Relationship to neighbors:

- **Stage 6 QA** tests spec'd negative cases per FEAT; this audit is broader — it sweeps *every* mutation entry point against *every* applicable guard class, including combinations the spec never wrote down.
- **pdlc-adversarial** attacks opportunistically after QA; running this audit first means Team Black wastes fewer round-1 attacks on trivially missing auth checks and digs for deeper breaks instead.
- Guards that should have been policy but weren't → route back as stage-3 POL additions (the doc is the source of truth, code follows).

## Inputs (required)

- `src/`, `supabase/migrations/*.sql` — the audit target is the build
- `docs/03-spec/functional-spec.md`, `docs/03-spec/policies.md` — source for deriving guard rules (who may do what, when)
- `docs/05-build/build-notes.md` — FEAT ↔ file mapping locates each entry point
- Seed accounts from `pdlc-seed-data` (at least 2 users with different data ownership, plus a logged-out context)
- `docs/pipeline-state.json` — add a top-level `guard_audit` block (`{"status": "in_progress"}`) on start; do not touch numeric `stages` keys

## Outputs

- `docs/056-guard/guard-audit.md` — guard inventory (GRD-###) + verification results + fixes
- Code fixes for missing guards (per stage-5 conventions), recorded in build-notes decisions
- New POL entries in `docs/03-spec/policies.md` when a guard had no policy backing

## The five guard classes

Derive the inventory by crossing every **mutation entry point** (server action, API route, direct DB write path) with these classes; mark N/A where a class doesn't apply, but mark it explicitly — silence is how gaps hide:

1. **인증 가드 (auth gate)** — action requires login. The canonical example: 비로그인 상태에서 댓글·좋아요 요청이 서버에서 거부되는가?
2. **권한 가드 (permission gate)** — actor may only touch what they own or their role allows. User A must not mutate user B's rows; a member must not call admin actions. RLS is the last line, but server code should refuse first with a proper error.
3. **상태 가드 (state gate)** — target must be in a valid state. Deleted/expired/closed targets refuse mutation (댓글이 달린 글이 삭제됐다면? 마감된 이벤트에 신청한다면?). Source: stage-3 state diagrams and POLs.
4. **입력 가드 (input gate)** — server-side validation of type, range, length, required fields, and referential existence (the posted `post_id` actually exists and is visible to the actor).
5. **중복·남용 가드 (duplicate/abuse gate)** — double-submit, replay of the same like, re-entry into a once-only flow. Idempotency or explicit rejection, per policy.

## Workflow

### 1. Build the guard inventory

List every mutation entry point from build-notes' FEAT ↔ file mapping (missing entries in that mapping are themselves a finding). For each, derive applicable guards from the spec/policies and assign GRD-### with: entry point / guard class / rule ("비로그인이면 401") / POL·FEAT reference (or "policy gap"). Get user confirmation on the inventory before verifying — the inventory *is* the deliverable's spine.

### 2. Verify each guard against the running build

Attempt the forbidden action for real: logged-out request, cross-user request with seed account B against A's data, mutation on a state-invalid target, malformed payload, double submit. Record per GRD: method / expected refusal / observed / **holds · missing · partial** (partial = UI blocks it but the server doesn't — counts as missing). Where server code passes but RLS is the only stopper, note it: defense-in-depth wants both layers.

### 3. Fix missing guards

Per stage-5 conventions (server actions, zod, RLS on, error copy from the spec). For each fix: record in build-notes decisions, re-run the failed attempt (must now refuse), and re-check the happy path still works (a guard that blocks legitimate users is a new bug, not a fix). If the guard had no policy backing, add the POL entry — code without a documented rule will drift.

### 4. Wrap up

- Summary table: total entry points, guards by class, holds/missing-fixed/N-A counts.
- Update `pipeline-state.json`: `guard_audit.status: done`, counts, one-line summary; policy gaps added to `decisions`.
- Propose stage 6 (pdlc-6-qa), noting negative TCs can now reference GRD IDs as preconditions already verified once.

## Cautions

- Verify by attempting, not by reading. "코드에 체크가 보인다" is not a pass; the request must actually be refused.
- Don't expand into feature work — a guard reveals a missing feature? Backlog note, not scope.
- Keep refusals user-respecting: correct status codes and the spec's error copy, not silent failures or stack traces.
- Output language rule: deliverables in Korean, terms glossed in English on first mention.

## Completion criteria (self-check)

- [ ] Every mutation entry point appears in the inventory (cross-checked against FEAT ↔ file mapping)
- [ ] All five guard classes explicitly marked (applicable or N/A) per entry point
- [ ] Every guard verified by an actual forbidden attempt, not code reading
- [ ] Zero `missing`/`partial` remaining (all fixed and re-verified, happy path intact)
- [ ] Policy-gap guards written back to policies.md as POL entries
- [ ] `guard_audit` block updated in pipeline-state.json

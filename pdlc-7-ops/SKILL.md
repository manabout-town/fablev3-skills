---
name: pdlc-7-ops
description: PDLC pipeline stage 7 — write 3 operational issue scenarios plus improvement and redeploy plans. Use this skill whenever the user asks for operations-phase work, e.g. "운영 이슈 시나리오", "장애 대응 시나리오 만들자", "운영 단계 하자", "개선 계획", "재배포 계획", "롤백 계획", "포스트모템 연습", or when post-launch response capability must be documented after QA.
---

# Stage 7: Ops Issue Scenarios + Improvement/Redeploy Plan

Simulate life after launch. QA asks "did we build to spec?"; operations asks "when the unexpected hits, how do we move?" Two threads: ① handle 3 ops issues in incident-response form (detect → assess → respond → prevent recurrence), ② convert the lessons into an improvement plan and a safe redeploy plan. This is where portfolio candidates diverge the most — many people can build; few think in operations.

## Inputs

- `docs/05-build/build-notes.md`, `docs/05-build/db-schema.md` — issues must emerge from this system's actual structure. No generic scenarios.
- `docs/06-qa/test-scenarios.md` — Deferred bugs (P3–P4) often seed ops issues.
- `docs/03-spec/policies.md` — policy grounding for responses.
- `docs/pipeline-state.json` — set stage 7 `in_progress` on start.

## Outputs

- `docs/07-ops/ops-issues.md` — 3 issues (OPS-001~003)
- `docs/07-ops/improvement-plan.md`
- `docs/07-ops/redeploy-plan.md`

Read `references/templates.md` for templates.

## Workflow

### 1. Pick 3 issues — from three different categories

Each issue must exercise a different muscle:

- **A. Technical failure**: external dependency outage (Supabase down, OAuth provider failure), error spike after deploy, performance degradation (slow queries, N+1)
- **B. Data/security**: RLS hole allowing out-of-permission access, data corruption from a bad migration, personal-data handling mistake
- **C. User/business**: usage collapse of a feature, abuse exploiting a policy gap, complaint surge

Each issue must be **grounded in this system's real structure** — for an "RLS hole," point to which condition is missing in which actual policy in db-schema.md. Fictional issues that can't occur in the real system get spotted by reviewers instantly. Discuss candidates with the user and confirm 3.

### 2. Write each issue as an incident response

Narrate each OPS as a timeline: detection (what surfaced it — an alert? a complaint? does this system even have that detection?) → impact assessment (scope, severity call) → immediate response (mitigation; notice draft if needed) → RCA (Root Cause Analysis — 5 Whys) → recovery → prevention. Prevention means structural measures (monitoring added, DB constraint, policy change, test added), never "we'll be careful."

Realizing there is no detection mechanism is itself a key finding — that gap becomes improvement-plan priority #1.

### 3. Improvement plan

Merge the 3 issues' prevention items + QA Deferred bugs + stage-5 backlog into one prioritized list. Per item: impact (what risk/pain it reduces), effort (S/M/L), priority verdict. Lead with quick wins per Impact-Effort. Split "next sprint" from "backlog" and decide with the user whether to update the stage-4 plan.

### 4. Redeploy plan

Define the procedure to ship improvements safely: pre-deploy checklist (migration backward-compatibility, build + regression TCs), deploy sequence (Vercel + Supabase migration order — DB first, code second, assuming backward compatibility; split incompatible changes expand-migrate-contract), post-deploy verification (smoke test list), and a **rollback plan** (numeric trigger criteria, the concrete rollback procedure, migration reversibility and the fallback when irreversible). If no real deployment target exists, mark it as an "if deployed" plan but keep the procedures executable-grade.

### 5. Wrap up

- Apply user review.
- Update `docs/pipeline-state.json`: stage 7 `done`, 3 outputs, `current_stage: 8`. Feed issue-derived risks into `risks`.
- Propose stage 8 (pdlc-8-portfolio), noting stage 8 consumes all of stages 1–7 plus the decisions log.

## Output language rule

Deliverables in Korean, terms glossed in English on first mention.

## Completion criteria (self-check)

- [ ] 3 issues from distinct categories A/B/C
- [ ] Each issue points at real system structure (files, policies, tables)
- [ ] Complete detect→assess→respond→RCA→prevent timeline
- [ ] Improvement plan merges OPS preventions + Deferred bugs + backlog, prioritized
- [ ] Redeploy plan has numeric rollback triggers and procedure

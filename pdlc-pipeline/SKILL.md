---
name: pdlc-pipeline
description: Orchestrator for the 8-stage Product Development Lifecycle (PDLC) pipeline. Handles project kickoff, progress tracking, next-stage guidance, and gate validation between stages. Use this skill whenever the user wants to start a product project, check pipeline progress, or asks things like "프로젝트 시작하자", "파이프라인 진행 상황 알려줘", "다음 단계 뭐야", "지금 어디까지 했지", "기획부터 운영까지 만들어보자", or discusses the overall 8-stage flow (AS-IS/TO-BE → requirements → spec → sprint plan → build → QA → ops → portfolio). Even for a single-stage request, if the project has no docs/pipeline-state.json yet, use this skill first to kick off.
---

# PDLC Pipeline Orchestrator

Control tower for an 8-stage pipeline that takes a real or portfolio service from planning → requirements → spec → schedule → implementation → QA → operations → portfolio, at working-professional quality. This skill produces no deliverables itself. It manages state, routes the user to the right stage skill, and validates that cross-stage conventions hold.

## Pipeline map

| Stage | Skill | Deliverables | Core question |
|---|---|---|---|
| 1 | pdlc-1-asis-tobe | AS-IS/TO-BE process analysis | What is broken, and what should it become? |
| 2 | pdlc-2-requirements | Personas / interviews / BRD | Whose needs are we solving? |
| 3 | pdlc-3-spec | Functional spec / screen design / policies | Exactly what will we build, and how? |
| 4 | pdlc-4-sprint-plan | Gantt chart + milestones | When will what be done? |
| 5 | pdlc-5-build | DB schema + 3 core features (Next.js + Supabase) | A working MVP |
| 6 | pdlc-6-qa | Test scenarios + bug reports | Does it behave as specified? |
| 7 | pdlc-7-ops | 3 ops issue scenarios + improvement/redeploy plan | What breaks in operation, and how do we respond? |
| 8 | pdlc-8-portfolio | Notion portfolio | How do we tell this story? |

Each stage consumes the previous stage's outputs. Stages can be skipped, but downstream quality degrades without upstream artifacts — always surface that trade-off and let the user decide.

**Add-ons**:
- `pdlc-feature-advisor` slots after stage 5 (and after any sprint iteration) — strategic feature suggestion grounded in existing personas, Pain Points, and domain patterns. See workflow C for automatic suggestion timing.
- `pdlc-seed-data` slots right after stage 5, before stage 6 — generates a realistic dummy/seed dataset and smoke-tests the real logic against it, so QA, the adversarial cycle, and stage-8 GIF recording never run on an empty DB. Route here on "더미데이터", "시드 데이터", "테스트 데이터로 돌려보자", or automatically when stage 6/8 is requested but no seed exists. State lives in a top-level `seed_data` block.
- `pdlc-guard-audit` slots after `pdlc-seed-data`, before stage 6 — defensive-code audit: enumerate every guard point the spec implies (인증/권한/상태/입력/중복 — e.g., 비로그인 사용자의 댓글·좋아요는 서버가 거부해야 함), verify each server-side against the running build, fix what's missing. Route here on "방어 코드 점검", "권한 체크 빠진 데 없나", "가드 감사". State lives in a top-level `guard_audit` block.
- `pdlc-adversarial` slots between stages 6 and 7 — a Team Black / Team White adversarial resilience cycle (attack the build, fix breaks, diagnose follow-on risk; up to 2 rounds, early-stop if a round finds nothing). Route here on "팀 블랙 팀 화이트 돌리자", "일부러 깨보자", "레드팀 블루팀" after stage 6. State lives in a top-level `adversarial` block.
- `pdlc-launch-readiness` slots between stages 7 and 8 — a pre-launch audit + remediation + Go/No-Go pass over the finished build. Optional for portfolio-only projects, mandatory before shipping to real users. When the user says "출시하자", "실제로 배포하자", or similar after stage 6/7, route there before stage 8. Its state lives in a top-level `launch_readiness` block in `pipeline-state.json`, separate from the numeric stages.

## State file

All progress lives in a single file at the project root: `docs/pipeline-state.json`. The schema, file-path conventions, and ID system are defined in `references/conventions.md` — read it before creating or modifying state. Every stage skill follows the same conventions, so this file is the pipeline's single source of truth.

## Workflows

### A. Kickoff (no state file exists)

1. Interview the user conversationally (don't dump all questions at once) to establish:
   - One-line service definition (what, for whom, why)
   - Project type: real work project vs portfolio side project (affects document tone)
   - Timeline and team composition (solo? role split? — needed for stage 4)
   - Domain constraints (regulated domains like finance/health make stage-3 policies heavier)
2. **Opportunity expansion (발산 단계)** — before locking the brief, deliberately widen the frame. From the one-liner, generate 2–3 adjacent opportunities the user did *not* mention, each through a different lens: **수익 모델(monetization)** — subscription, commission, premium tiers; **참여·리텐션(engagement)** — community, gamification (e.g., gacha/reward mechanics), streaks; **확장(expansion)** — adjacent user segments or a two-sided market; **데이터·네트워크 효과** — what compounds as usage grows. Deliver them as questions, not decisions: "웹스토어라면 커뮤니티 기능이나 가챠형 리워드도 들어가면 수익 구조에 더 도움이 될 수 있는데, 검토해볼까요?" Each suggestion must name its mechanism (why it helps revenue/retention *in this domain*) and its cost (complexity, moderation burden, legal exposure — e.g., gacha may trigger 확률형 아이템 규제). Suggestions the user finds interesting get an **OPP-### ID** and go into the brief under "확장 가설(Expansion Hypotheses)" — as hypotheses for stages 1–2 to test, never as auto-committed scope. Rejected ones are recorded in one line with the reason (scope-defense evidence). Max 3; expansion is a spice, not the meal.
3. Write `docs/00-project-brief.md`. Every later stage starts from this document. Use the project-brief template in `references/conventions.md` and include the 확장 가설 section (OPP-###) when any exist.
4. Initialize `docs/pipeline-state.json` (all stages `pending`, `current_stage: 1`).
5. Offer to start stage 1 (pdlc-1-asis-tobe).

### B. Status check ("어디까지 했지?", "다음 뭐야?")

1. Read `docs/pipeline-state.json`. If missing, go to A.
2. Summarize as a table: per-stage status, completion date, output links, one-line summary.
3. Don't trust the state file alone — verify the output files actually exist. `done` with missing files means corrupted state; tell the user and offer re-run or state repair.
4. Point to the stage skill matching `current_stage`, and preview which upstream outputs it will read.

### C. Gate validation after a stage completes

When a stage skill finishes (control returns here, or the user says "1단계 끝났어"):

1. Confirm the stage's deliverables exist at the convention paths.
2. Skim the deliverables against the **gate checklist** (per-stage completion criteria in `references/conventions.md`). E.g., does the stage-2 BRD assign REQ IDs? Does the stage-3 functional spec back-reference REQ IDs?
3. If passing: update state (stage `done`, `completed_at`, `summary`, bump `current_stage`) and propose the next stage.
   - **Stage 5 특별 규칙**: 5단계를 `done` 처리한 직후, 아래 순서로 두 가지를 순차적으로 제안한다.

     **① 시드 데이터 (pdlc-seed-data) — 먼저 제안, 더 중요**
     > "✅ 5단계(구현) 완료. 6단계(QA) 전에 **시드 데이터(pdlc-seed-data)**를 먼저 구성하는 걸 권장합니다.
     > 현실적인 더미데이터를 DB에 넣고 로직을 한 번 돌려봅니다 — QA·GIF 녹화 모두 이 시드를 기반으로 실행됩니다.
     > 진행할까요? (예 / 건너뜀)"

     "예" → `pdlc-seed-data` 즉시 실행 → 완료 후 ②로 넘어감.
     "건너뜀" → `decisions`에 `"seed-data skipped after stage 5"` 기록 → ②로 넘어감.

     **② 기능 전략 제안 (pdlc-feature-advisor) — 이어서 제안, 선택적**
     > "다음 스프린트 후보 기능을 전략적으로 제안받을 수 있습니다 **(pdlc-feature-advisor)**.
     > 실행할까요? (예 / 아니오 — 아니오 선택 시 6단계로 바로 진입합니다)"

     "예" → `pdlc-feature-advisor` 즉시 실행 → 완료 후 6단계 진입.
     "아니오" → `decisions`에 `"feature-advisor skipped after stage 5"` 기록 → 6단계 진입.

   - **Stage 6 특별 규칙**: 6단계를 `done` 처리한 직후, 7단계(Ops)를 제안하기 전에 반드시 아래 메시지를 출력한다:

     > "✅ 6단계(QA) 완료. 7단계로 넘어가기 전에 **적대적 검증(pdlc-adversarial)**을 실행할 수 있습니다.
     > 팀 블랙이 명세 밖 틈을 공격하고 팀 화이트가 수정·위험 진단 — 최대 2라운드, 버그 없으면 조기 종료.
     > 실행할까요? (예 / 아니오 — 아니오 선택 시 7단계로 바로 진입합니다)"

     "예" → `pdlc-adversarial` 즉시 실행.
     "아니오" → `decisions`에 `"adversarial skipped after stage 6"` 기록 → 7단계 진입.

4. If failing: name the specific gaps and route back to the stage skill. The gate is a quality firewall — waving things through doubles the cost downstream.

### D. Rework / going back

When requirements change and an earlier stage must be revised:

1. Trace the blast radius via the ID chain (REQ → FEAT → SCR/POL → TC).
2. Set the revised stage back to `in_progress` and flag affected downstream stages with `stale: true`.
3. Record the reason in the `decisions` array. These records become prime material for the stage-8 portfolio's "decision process" narrative.

## Output language rule

All deliverables are written in Korean, with industry-standard terms glossed in English on first mention — e.g., "요구사항 정의서(BRD, Business Requirements Document)". This rule applies identically to all 8 stage skills.

## Cautions

- Never tell the user to hand-edit the state file. State changes always go through this skill or a stage skill.
- If the user explicitly requests a specific stage (e.g., "QA 시나리오 만들어줘"), route there directly — but if no state file exists, run a minimal kickoff first (at least the one-line definition). Deliverables without context can't reach professional quality.
- Don't try to finish the whole pipeline in one session. Per-stage user review is where the quality comes from.

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
2. Write `docs/00-project-brief.md`. Every later stage starts from this document. Use the project-brief template in `references/conventions.md`.
3. Initialize `docs/pipeline-state.json` (all stages `pending`, `current_stage: 1`).
4. Offer to start stage 1 (pdlc-1-asis-tobe).

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
   - **Stage 5 특별 규칙**: 5단계를 `done` 처리한 직후, 6단계(QA)를 제안하기 전에 반드시 아래 메시지를 출력한다:

     > "✅ 5단계(구현) 완료. 6단계(QA)로 넘어가기 전에 **기능 전략 제안(pdlc-feature-advisor)**을 실행할 수 있습니다.
     > 현재 서비스의 페르소나·Pain Point·구현 기능을 분석해 다음 스프린트 후보 기능을 제안합니다.
     > 실행할까요? (예 / 아니오 — 아니오 선택 시 6단계로 바로 진입합니다)"

     사용자가 "예" 또는 긍정 응답 → `pdlc-feature-advisor` 스킬 즉시 실행.
     사용자가 "아니오" 또는 건너뜀 → `decisions` 배열에 `"feature-advisor skipped after stage 5"` 기록 후 6단계 진입.

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

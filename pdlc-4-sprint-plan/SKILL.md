---
name: pdlc-4-sprint-plan
description: PDLC pipeline stage 4 — sprint planning, Gantt chart, and milestones. Use this skill whenever the user asks for schedule/planning work, e.g. "간트차트 만들어줘", "일정 계획 세우자", "스프린트 짜자", "마일스톤 정의", "로드맵", "WBS", "개발 일정", or when scope and dates must be locked before implementation. Outputs both a Mermaid gantt (for documents) and an interactive HTML gantt (for presentations).
---

# Stage 4: Sprint Plan (Gantt + Milestones)

Convert the spec into an executable schedule. Two decisions carry all the weight: ① **final selection of the 3 core features** for stage 5, ② **exit criteria** for each milestone. The Gantt chart is merely the visualization of those decisions — decisions first, pretty chart second.

## Inputs

- `docs/03-spec/functional-spec.md` — FEAT list and ★ candidates. Required.
- `docs/00-project-brief.md` — timeline and team constraints.
- `docs/pipeline-state.json` — set stage 4 `in_progress` on start.

## Outputs

- `docs/04-plan/milestones.md`
- `docs/04-plan/gantt.md` (Mermaid)
- `docs/04-plan/gantt.html` (interactive)

Templates and the HTML skeleton: read `references/templates.md`.

## Workflow

### 1. Finalize the 3 core features

Decide with the user among the stage-3 ★ candidates. Criteria: Must-REQ coverage, demo-ability (do the 3 form one connected user journey?), balanced implementation difficulty. If the service needs auth, don't count it among the 3 — schedule it as foundation work. Record the selection and elimination rationale in `decisions`.

### 2. WBS (Work Breakdown Structure)

Decompose the confirmed scope into tasks. Hierarchy: milestone > task > (if needed) subtask. Each task gets a role (even solo projects label 기획/디자인/개발/QA), estimated effort, and dependencies. Include the pipeline's own remaining stages (5 build, 6 QA, 7 ops, 8 portfolio) in the schedule — this plan is not fiction inside a document; it is the plan this project will actually follow.

### 3. Define milestones

3–5 milestones. Each has a date + **verifiable exit criteria** + deliverables. Ban vague criteria like "개발 완료" — write "핵심 기능 3개가 로컬에서 시연 가능하고 TC 통과율 90% 이상" instead.

### 4. Build the Gantt charts

**Mermaid** (`gantt.md`): milestones as sections, dependencies via `after`, critical-path tasks marked `crit`. Set `excludes weekends` per project nature.

**HTML** (`gantt.html`): use the skeleton in `references/templates.md` — a single self-contained file. Drop task data into the JS arrays to get bars + milestone diamonds + a today line + hover details. Must work with zero external CDN (offline presentations happen).

The two charts must not diverge. Lock the data first, then render both formats from it.

### 5. Risks and buffer

Identify 2–3 schedule risks with RISK IDs (e.g., Supabase RLS learning curve) and mitigations. Reserve an explicit 15–20% of total duration as buffer — a plan without buffer is not a plan, it's a wish.

### 6. Wrap up

- Apply user review (especially schedule realism).
- Update `docs/pipeline-state.json`: stage 4 `done`, 3 outputs, `current_stage: 5`, record the 3 confirmed features in `decisions`.
- Propose stage 5 (pdlc-5-build).

## Output language rule

Deliverables in Korean, terms glossed in English on first mention. Mermaid task names may be Korean.

## Completion criteria (self-check)

- [ ] 3 core features confirmed + elimination rationale recorded
- [ ] Every task has role, effort, dependencies
- [ ] Every milestone has verifiable exit criteria
- [ ] Mermaid gantt (with crit) + HTML gantt, data identical
- [ ] Explicit 15–20% buffer; schedule risks recorded

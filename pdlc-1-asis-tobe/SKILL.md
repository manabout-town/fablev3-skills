---
name: pdlc-1-asis-tobe
description: PDLC pipeline stage 1 — write the AS-IS/TO-BE process analysis document. Use this skill whenever the user asks for current-process analysis or an improved future process, e.g. "AS-IS 분석", "현행 프로세스 분석해줘", "TO-BE 설계", "프로세스 개선안", "문제 정의부터 하자", or when a new service idea needs process-level validation as the first step of product planning.
---

# Stage 1: AS-IS / TO-BE Process Analysis

Decompose the current (AS-IS) process, diagnose pain points with evidence, and design the improved (TO-BE) process — the gap between them is the reason this service should exist. If this document is weak, stage-2 requirements degrade into "a list of features someone wanted."

## Inputs

- `docs/00-project-brief.md` — read first. If missing, route the user to the pdlc-pipeline skill for kickoff, or at minimum get a one-line service definition and generate a brief on the spot.
- `docs/pipeline-state.json` — set stage 1 to `in_progress` on start.

## Output

- `docs/01-process-analysis.md` — read `references/template.md` and follow it exactly.

## Workflow

### 1. Pin down the target process

Narrow "whose process, doing what" through conversation. Even for a brand-new service with no existing system, an AS-IS exists — it's the workaround users employ today (spreadsheets, KakaoTalk, pen and paper, a patchwork of competitor tools). Concluding "there is no AS-IS" means the analysis failed.

### 2. Decompose the AS-IS

Break the process into steps. For each: actor, tools used, time taken, problems occurring. For a work project, ask the user for real numbers; for a side project, use reasonable estimates and label them as estimates. Never write an evidence-free pain point — not "inconvenient" but "15 min per case, 40 cases/month, ~10% mis-entry rate". Without magnitudes, stage 2 can't prioritize.

### 3. Extract pain points and analyze causes

List per-step problems as pain points, each with a PP-### ID. Dig past symptoms to causes — apply 5 Whys at least a level or two and record why the problem occurs. Different causes lead to different TO-BE designs.

### 4. Design the TO-BE

**Expand before you converge.** Before drafting the TO-BE, look past the PP list once: does the brief carry 확장 가설(OPP-###)? Do any *new* opportunities emerge from the AS-IS itself (e.g., a step users repeat daily is a retention hook; a step where users consult others is a community seed; a purchase step is a monetization surface)? Raise at most 1–2 as questions to the user — "이 단계에서 사용자들이 서로 후기를 찾아다니는데, 커뮤니티 기능을 넣으면 수익 구조에도 도움이 되지 않을까요?" — with mechanism and cost stated. Accepted ones become OPP entries in the brief; then converge.

Decompose the TO-BE the same way, then build a mapping table showing which PP (and, where applicable, OPP) each improvement resolves. If a PP goes unresolved, explicitly mark it "out of scope for this round" (this becomes your scope-defense evidence later). The TO-BE does not fix implementation — write "a notification is sent automatically" and leave how (push? email?) for stage 3.

### 5. Visualize

Draw both AS-IS and TO-BE as Mermaid `flowchart`s. With 2+ actors, separate actors with subgraphs (Mermaid has no swimlanes). Embed the diagrams as code blocks in the document.

### 6. Quantified expected impact

Table the improvement metrics of the TO-BE (e.g., processing time 15 min → 2 min). Even for a side project this table is mandatory — it's what makes the "hypothesis → validation" story work in the stage-8 portfolio.

### 7. Wrap up

- Get user review and apply it.
- Update `docs/pipeline-state.json`: stage 1 `done`, `completed_at`, `outputs: ["docs/01-process-analysis.md"]`, one-line `summary`, `current_stage: 2`.
- Propose stage 2 (pdlc-2-requirements), noting that this document's PP IDs will serve as evidence for stage-2 requirements.

## Output language rule

Deliverables in Korean, industry terms glossed in English on first mention — e.g., "현행 분석(AS-IS Analysis)".

## Completion criteria (self-check)

- [ ] AS-IS decomposed into steps, each with actor·tools·time
- [ ] Every pain point has a PP ID, quantitative/qualitative evidence, and cause analysis
- [ ] TO-BE steps mapped to PPs in a table (unresolved PPs marked out of scope)
- [ ] OPP-### exists in brief → included in TO-BE mapping table as additional column (if none exist, skip)
- [ ] Two Mermaid flowcharts (AS-IS, TO-BE)
- [ ] Quantified expected-impact table

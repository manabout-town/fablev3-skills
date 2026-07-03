---
name: pdlc-2-requirements
description: PDLC pipeline stage 2 — requirements definition. Produces personas, user interview design/simulation, and a BRD (Business Requirements Document). Use this skill whenever the user asks for requirements work, e.g. "요구사항 정의", "BRD 만들어줘", "페르소나 만들자", "유저 인터뷰 설계해줘", "누가 쓸 서비스인지 정리하자", or when what-to-build must be pinned down from the user's perspective after process analysis.
---

# Stage 2: Requirements (Personas / Interviews / BRD)

Turn the stage-1 problem diagnosis into "whose needs, specifically." The three deliverables form a logical chain: personas set hypotheses → interviews test them → the BRD locks in validated requirements with priorities. Break the chain (a BRD without interviews, interviews unrelated to personas) and the requirements become the author's imagination.

## Inputs

- `docs/00-project-brief.md`, `docs/01-process-analysis.md` — required reading; PP IDs are the evidence base for requirements. If stage-1 output is missing, state the quality trade-off and let the user choose.
- `docs/pipeline-state.json` — set stage 2 `in_progress` on start.

## Outputs

- `docs/02-requirements/personas.md`
- `docs/02-requirements/interviews.md`
- `docs/02-requirements/brd.md`

Read `references/templates.md` and follow the templates.

## Workflow

### 1. Personas (2–3)

Start from the stage-1 actors. One primary + 1–2 secondary. Build each around **behavior and goals**, not demographic trivia: current behavior (what this person does in the AS-IS), goals (JTBD, Jobs to be Done), frustrations (linked PP IDs), tech fluency, one defining quote. Give each persona 2–3 explicit hypotheses to test — these feed the interview design. If the brief carries 확장 가설(OPP-###), assign each OPP to the persona best positioned to falsify it and include it among that persona's hypotheses — an OPP nobody interviews about is a fantasy, not a hypothesis.

### 2. Interview design and execution

**Design**: 8–12 questions per persona that test its hypotheses. No leading questions ("~하면 편하시겠죠?" ❌); use past-behavior questions ("최근에 ~했던 경험을 말해주세요" ⭕). Define screening criteria too.

**Choose the execution mode** — ask the user:
- **Real interviews possible**: deliver only the interview guide; when the user returns with transcripts/notes, help synthesize.
- **Simulated interviews** (typical for side projects): simulate 2 virtual respondents per persona. Crucially, **include responses that contradict some hypotheses**. A simulation where everyone agrees is self-affirmation, not validation — and it's the first thing a portfolio reviewer flags. Label the document "가상 인터뷰 시뮬레이션" explicitly.

**Synthesis**: cluster responses via affinity mapping into 3–5 insights with INS-### IDs. Record a verdict per hypothesis (supported / contradicted / partial). If a hypothesis is contradicted, revise the persona or direction and log it in `decisions` — this pivot record is evidence of real product judgment.

### 3. Write the BRD

- Split into functional requirements (FR) and non-functional requirements (NFR), each with a REQ-### ID.
- Every REQ gets an **evidence column**: which PP, INS, or OPP it came from. Delete evidence-free REQs or confirm their origin with the user.
- Prioritize with MoSCoW (Must/Should/Could/Won't). Always fill in Won't — naming what you will not do is your scope defense line.
- **OPP handling**: an OPP whose interview verdict is *supported* may enter as Should/Could (Must only with the user's explicit call); *contradicted* → record in Won't with the verdict as the reason; *untested* → Could at most, flagged "미검증". This keeps expansive ideas alive without letting them bloat the 3-feature MVP.
- Define 2–3 service-level KPIs (concretizing the brief's success criteria).
- Since stage 5 implements only **3 core features**, warn now if Musts are bloated. Musts should be coverable by 3 features (roughly 5–8 REQs).

### 4. Wrap up

- Apply user review.
- Update `docs/pipeline-state.json`: stage 2 `done`, 3 outputs, `current_stage: 3`.
- Propose stage 3 (pdlc-3-spec), noting BRD REQ IDs become the back-reference targets of the functional spec.

## Output language rule

Deliverables in Korean, industry terms glossed in English on first mention.

## Completion criteria (self-check)

- [ ] 2+ personas, each with 2–3 explicit hypotheses
- [ ] Interview questions are hypothesis-testing, no leading questions
- [ ] (If simulated) contradicting responses included; simulation labeled
- [ ] Every REQ has ID + MoSCoW + evidence (PP/INS reference)
- [ ] NFRs included (performance/security/availability as applicable)
- [ ] Won't (out-of-scope) list exists

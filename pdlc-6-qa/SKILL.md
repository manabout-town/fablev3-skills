---
name: pdlc-6-qa
description: PDLC pipeline stage 6 — QA test scenario design, test execution, and bug reporting. Use this skill whenever the user asks for verification work, e.g. "테스트 시나리오 만들어줘", "QA 하자", "테스트 케이스 작성", "버그 리포트 템플릿", "검증해줘", "테스트 돌려보자", or when the implementation must be verified against the spec after stage 5.
---

# Stage 6: QA Test Scenarios + Bug Reports

Systematically verify "does it behave as specified." QA's oracle is not the code but the **stage-3 spec** — code that runs fine but differs from spec is a defect, and a spec too ambiguous to judge against is also a finding (a spec defect). Without this stance, QA degenerates into "I check the thing I built myself."

## Inputs (required)

- `docs/03-spec/functional-spec.md` — the test oracle; FEAT flows/validation/error cases are the source of TCs.
- `docs/03-spec/policies.md`, `docs/03-spec/screen-design.md` — criteria for policy violations and missing states.
- `docs/05-build/build-notes.md` — FEAT ↔ file mapping, run instructions.
- `docs/pipeline-state.json` — set stage 6 `in_progress` on start.

## Outputs

- `docs/06-qa/test-scenarios.md` — scenarios (TS) + cases (TC) + execution results
- `docs/06-qa/bug-report-template.md` — reusable template
- `docs/06-qa/bug-reports/BUG-###.md` — one report per bug found

Read `references/templates.md` for templates.

## Workflow

### 1. Test design

Per core feature (FEAT), build a scenario (TS) with TCs in three classes:

- **Happy path**: the spec's main flow verbatim
- **Negative**: the spec's error cases + applicable POLs (no permission, validation violations, duplicate submission)
- **Boundary**: edges of validation rules (max length, 0 items, max count, date boundaries)

Add at least one cross-feature scenario: the integrated user journey spanning all 3 features (the demo journey defined in stage 4). Every TC carries preconditions / steps / expected result (spec wording verbatim) / FEAT·POL references. If an expected result can't be derived from the spec, log a spec defect and repair the spec with the user first.

5–10 TCs per feature is the right range. Get user review, then execute.

### 2. Test execution

Bring up the local environment per build-notes and run the TCs. If browser automation is available (Chrome MCP etc.), execute directly; otherwise handle TCs verifiable via code inspection + direct server-action calls first, then hand the user a manual execution script and collect results. Record the execution mode per TC (automated / code-inspection / manual).

Verdicts: Pass / Fail / Blocked (prior TC failure prevents run) / N/A. **Don't fear Fails** — a QA document with zero Fails earns zero trust. If there truly are no defects, your boundary TCs are too soft; push the boundaries harder.

### 3. Bug reports

Write a BUG-### report per Fail. Principles:

- **Reproduction steps a stranger could follow**: environment, data state, click-by-click.
- **Expected vs actual** with spec citations: "FEAT-002 유효성 표 기준 X여야 하나 실제 Y".
- **Severity and priority are separate axes**: severity = size of impact (Critical/Major/Minor/Trivial), priority = fix order (P1–P4). A trivial bug on the landing page can be P1; a severe bug in an extreme corner can be P3.
- Mark root-cause guesses as guesses, distinct from confirmed diagnoses.

### 4. Fix and re-verify

Fix P1–P2 bugs in this stage (respect stage-5 conventions; include BUG IDs in fix commits). Re-run the failing TC plus neighboring TCs for regression. Record fix + re-verification in the report and close it. P3–P4 may defer to backlog at the user's call (mark Deferred).

### 5. Wrap up

- Write the summary: TC totals, pass rate, bug distribution by severity, spec defects found.
- Update `docs/pipeline-state.json`: stage 6 `done`, outputs, `current_stage: 7`.
- Propose stage 7 (pdlc-7-ops), noting that what QA can't catch — production-environment issues — is stage 7's subject.

## Output language rule

Deliverables in Korean, terms glossed in English on first mention.

## Completion criteria (self-check)

- [ ] 3 features × (happy + negative + boundary) TCs + 1+ integration scenario
- [ ] Every TC back-references FEAT/POL; expected results quote the spec
- [ ] All TCs executed with results and execution mode recorded
- [ ] A BUG report per Fail (severity/priority separated)
- [ ] P1–P2 fixed + re-verified; disposition recorded for the rest

---
name: pdlc-3-spec
description: PDLC pipeline stage 3 — write the functional specification, screen design (wireframe definitions), and policy document. Use this skill whenever the user asks for spec-level documents, e.g. "기능명세서 만들어줘", "화면설계 해줘", "와이어프레임 정의", "정책서 작성", "IA 잡아줘", "스펙 문서", or when what-to-build must be fixed precisely before development starts after the BRD is confirmed.
---

# Stage 3: Functional Spec / Screen Design / Policies

Convert the BRD's "what is needed" into "exactly what and how," at a level a developer can implement verbatim. Division of labor: functional spec = system behavior, screen design = what the user sees, policies = exceptions and rules. Policies are the one that most often ends up thin — yet every mid-development question ("탈퇴한 유저의 글은?", "중복 신청하면?") is the policy document's job to answer.

## Inputs

- `docs/02-requirements/brd.md` — required; all REQ IDs must be covered.
- `docs/02-requirements/personas.md`, `docs/01-process-analysis.md` — context for screen flows and policy judgment.
- `docs/pipeline-state.json` — set stage 3 `in_progress` on start.

## Outputs

- `docs/03-spec/functional-spec.md` (FEAT IDs)
- `docs/03-spec/screen-design.md` (SCR IDs)
- `docs/03-spec/policies.md` (POL IDs)

Templates: `references/templates.md`. Policy edge-case checklist: `references/policy-checklist.md`. Read both.

## Workflow

### 1. Derive features and the coverage matrix

Group Must/Should REQs into features (FEAT). One REQ may span several FEATs and vice versa. Build the **coverage matrix** (REQ × FEAT) first and confirm no Must REQ is uncovered. Coulds go into the FEAT list as backlog only.

### 2. Write the functional spec

Per FEAT: overview, related REQs, user story, behavior flows (main + alternative), input/output definitions, validation rules, error cases, related policies (POL references). Write flows as numbered "user does A → system does B" sentences — these sentences are the literal source text for stage-6 test cases. For features with complex state transitions (orders, reservations), include a Mermaid `stateDiagram-v2`.

Mark **candidate core features (★)** for stage-5 implementation now. Criteria: high Must-REQ coverage, and the ability to demo service value as one connected journey. Final selection happens in stage 4, but candidates must be marked here so screen-design priorities are right.

### 3. Write the screen design

- **IA (Information Architecture)**: screen hierarchy as a Mermaid diagram.
- **Screen flows**: SCR-to-SCR movement per major task.
- **Per-screen definitions**: purpose, entry paths, component table (element/type/behavior/linked FEAT), and state definitions. **State definitions are where professional detail lives** — beyond the default state, define empty, loading, error, and no-permission states for every screen.
- Express wireframes as text layouts (ASCII or structural description). Structure definition beats pretty pictures in this document.

### 4. Write the policy document

Walk `references/policy-checklist.md` and derive the policies that apply to this service. Each POL follows situation → rule → rationale. Cross-check that functional-spec error cases and policies reference each other. If any personal data is collected, the collection-items/purpose/retention table is mandatory (per Korean 개인정보보호법).

For every **state-changing action** (write/update/delete/like/comment/apply), the policy doc must answer: who may do it (login? role? ownership?), in what target state, and what the system does when a non-eligible actor tries anyway (error copy included). These rules become the guard inventory that `pdlc-guard-audit` verifies against the build after stage 5 — an action with no stated rule is a guard gap waiting to ship.

### 5. Cross-validate and wrap up

- Verify mutual references across the three documents: every FEAT → REQ, SCR → FEAT, FEAT error case → POL. Hunt down orphan IDs (referenced by nothing).
- Apply user review.
- Update `docs/pipeline-state.json`: stage 3 `done`, 3 outputs, `current_stage: 4`.
- Propose stage 4 (pdlc-4-sprint-plan), noting the FEAT list and ★ candidates are its inputs.

## Output language rule

Deliverables in Korean, terms glossed in English on first mention. UI copy (button labels, error messages) is finalized in actual service language (Korean).

## Completion criteria (self-check)

- [ ] REQ × FEAT coverage matrix; Must REQs 100% covered
- [ ] Every FEAT has main + alternative flows, error cases, POL references
- [ ] Core-feature candidates (★) marked
- [ ] Every SCR has a component table + 4 states (empty/loading/error/no-permission)
- [ ] IA diagram + screen flows
- [ ] Policy doc covers all applicable checklist areas

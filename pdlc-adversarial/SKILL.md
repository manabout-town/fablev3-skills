---
name: pdlc-adversarial
description: PDLC pipeline add-on (between stages 6 and 7) — adversarial resilience cycle. Team Black tries its hardest to break the build; Team White fixes what breaks and diagnoses follow-on risk. Runs up to 2 rounds, but stops early if Team Black finds nothing in a round. Use this skill whenever the user asks for adversarial or resilience testing, e.g. "팀 블랙 팀 화이트 돌리자", "버그 유발 테스트", "일부러 깨보자", "레드팀 블루팀", "공격 테스트", "내구성 검증", or after stage 6 QA when the user wants to stress the build beyond spec-based testing before operations.
---

# Adversarial Resilience Cycle (Team Black / Team White)

Stage-6 QA verifies the build against its own spec — a cooperative check. This add-on is adversarial: **Team Black** attacks the build in good faith, trying its hardest to induce bugs and errors the spec-based tests never imagined; **Team White** fixes every confirmed break and, crucially, diagnoses the follow-on risks each break exposes. The cycle runs up to **2 rounds**, with an early-stop rule that turns "no bugs found" into a positive signal rather than a reason to keep hammering.

Both teams are played by Claude, but in strictly separated passes — Team Black must not soften its attack because it knows Team White has to clean up, and Team White must not dismiss a finding to save work. Keeping the two mindsets honest is the whole point.

## Position in the pipeline

Runs after stage 6 (QA), before stage 7 (Ops). Optional but recommended before a real launch. Its findings feed two places downstream: confirmed breaks that can't be fully closed become **stage-7 ops issue** candidates, and the risk diagnoses feed the **launch-readiness** audit.

## Inputs (required)

- `src/`, `supabase/migrations/*.sql` — **the attack target is the running build**, not the docs.
- `docs/03-spec/functional-spec.md`, `docs/03-spec/policies.md` — the boundary of "intended behavior." A break is behavior outside this boundary; Team Black aims *between and around* the spec's stated cases.
- `docs/06-qa/test-scenarios.md` — the already-covered ground. Team Black must attack *beyond* what stage 6 tested; re-finding a known bug doesn't count.
- `docs/06-qa/bug-report-template.md` — reuse this template for confirmed breaks (BUG-### numbering continues from stage 6).
- `docs/pipeline-state.json` — add a top-level `adversarial` block (`{"status": "in_progress"}`) on start. Do not touch numeric `stages` keys.

## Outputs

- `docs/065-adversarial/attack-log.md` — every Team Black attempt (successful or not), by round
- `docs/065-adversarial/round-report.md` — per-round result: breaks found, fixes applied, risk diagnosis, cycle decision
- `docs/06-qa/bug-reports/BUG-###.md` — one report per confirmed break (continues stage-6 numbering)

Read `references/templates.md` for templates and `references/attack-playbook.md` for Team Black's attack surface catalog.

## The cycle

### Round structure (repeat up to 2×)

**Phase 1 — Team Black attacks.** Adopt the attacker mindset fully. Work through `references/attack-playbook.md` attack surfaces, prioritizing what stage 6 did *not* cover: malformed/boundary input, concurrent/duplicate requests, auth and RLS bypass attempts, broken state transitions, resource exhaustion, injection, client-trusted values, error-path abuse. Log **every** attempt in `attack-log.md` with: attack ID (ATK-###), surface, hypothesis ("if I send X, the missing check at Y should let it through"), method, and result (broke / held). A break requires **evidence** — an error, wrong data, unauthorized access, a crash — with reproduction steps. "Feels fragile" is not a break.

**Phase 2 — early-stop check.**
- **If Team Black induced zero confirmed breaks this round**: stop the cycle. Record it as a resilience signal ("Round N: N attacks across M surfaces, 0 breaks — build held"). Do **not** run another round just to fill the quota — a clean adversarial round is a result worth reporting. Skip to Wrap-up.
- **If Team Black induced ≥1 break**: proceed to Phase 3, then decide on another round.

**Phase 3 — Team White fixes and diagnoses.** For each confirmed break:
1. **Fix** it in code per stage-5 conventions (server actions, zod, migrations-only schema changes, RLS on). File a BUG-### report with the fix documented.
2. **Re-verify**: reproduce the original attack — it must now hold — and re-run any stage-6 TCs touching the same FEAT (regression guard).
3. **Diagnose follow-on risk** — this is what separates this skill from ordinary bug-fixing. For each break ask: *what else does this failure mode imply?* One missing auth check usually means the pattern is missing elsewhere; one unhandled concurrency case implies siblings. Record each as a risk (RISK-###) with likelihood, blast radius, and whether it's closed here, needs a stage-7 ops scenario, or belongs in the launch-readiness audit. A fix that closes one symptom while leaving the class open is only half done.

**Phase 4 — cycle decision.** Write the round report. Then:
- If this was **round 1 and breaks were found**: run **round 2** — Team Black attacks again, now targeting the risk classes surfaced in Phase 3 (fixes often open new seams). 
- If this was **round 2**: end the cycle regardless. Two rounds is the cap; anything still open past round 2 is escalated to stage 7 / launch-readiness rather than chased here.

### Wrap-up

- Summarize the cycle: rounds run, total attacks, breaks found/fixed, open risks and where each was routed.
- Update `pipeline-state.json`: `adversarial.status: done`, rounds, counts, and a one-line `summary`. Add any RISK-### to the top-level `risks` array; add ops-bound items to `decisions` noting they route to stage 7.
- Propose stage 7 (pdlc-7-ops), pointing at the RISK entries flagged as ops candidates.

## Cautions

- **Separation of concerns is the method.** Run the full Team Black pass and log all attempts *before* switching to Team White. If you fix as you go, you stop attacking hard.
- Attack the build as it will ship (deployed config, real auth), not a mocked happy path — but never attack systems or accounts outside this project.
- Early-stop is a feature, not laziness. Do not manufacture weak "breaks" to justify a second round.
- Output language rule: deliverables in Korean, terms glossed in English on first mention.

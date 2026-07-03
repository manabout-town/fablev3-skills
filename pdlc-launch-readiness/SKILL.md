---
name: pdlc-launch-readiness
description: PDLC pipeline add-on (between stages 7 and 8) — pre-launch readiness audit and remediation across security/data, quality/performance, deploy/ops, and user/legal/marketing. Use this skill whenever the user asks to prepare a finished build for real launch, e.g. "출시 전 점검하자", "런치 준비됐는지 봐줘", "배포 전에 확인할 것", "보안 점검해줘", "출시 체크리스트", "Go/No-Go 판정", "실제로 배포해도 되나?", or after stages 5–7 are done and the user wants to ship for real (not just as a portfolio).
---

# Launch Readiness: Pre-Launch Audit + Remediation

The pipeline's stages 6–7 prove the system *behaves as specified* and that you *can respond to incidents*. This add-on answers a different question: **is this safe and complete enough to put in front of strangers?** QA tests against the spec; launch readiness tests against the real world — attackers, slow networks, app-store reviewers, privacy law, and users who never read instructions. The output is not just a report: findings get fixed, re-verified, and signed off with an explicit Go/No-Go call.

## Position in the pipeline

Runs after stage 6 (QA) — ideally after stage 7 — and before stage 8 (portfolio) or any real deployment. It is optional for portfolio-only projects but mandatory before a real launch. Audit findings and their fixes are prime stage-8 material ("we found an RLS hole before launch and here's how we closed it" beats any feature demo).

## Inputs (required)

- `src/`, `supabase/migrations/*.sql` — **the audit target is the actual code**, not the docs. Docs say what was intended; the code says what ships.
- `docs/05-build/db-schema.md`, `docs/05-build/build-notes.md` — FEAT ↔ file mapping.
- `docs/03-spec/policies.md` — every POL must be enforced in code, not just written down.
- `docs/06-qa/test-scenarios.md` — deferred bugs (P3–P4) get re-triaged: acceptable pre-portfolio ≠ acceptable pre-launch.
- `docs/07-ops/ops-issues.md` — each OPS prevention measure gets checked: was it actually implemented?
- `docs/pipeline-state.json` — add a top-level `launch_readiness` block (`{"status": "in_progress"}`) on start. Do not touch the numeric `stages` keys.

## Outputs

- `docs/09-launch/launch-audit.md` — findings (LR-###) across the 4 domains, each with severity, evidence (file:line), and back-references to FEAT/POL/OPS/TC IDs
- `docs/09-launch/remediation-log.md` — what was fixed, how, and re-verification results
- `docs/09-launch/launch-checklist.md` — Go/No-Go checklist with per-item sign-off

Read `references/check-catalog.md` for the full check catalog and `references/templates.md` for document templates.

## Workflow

### 1. Scope the launch

Interview briefly: real users or portfolio demo? Expected traffic? Handling personal data or payments? Target platform (web only / PWA / store submission)? This sets which catalog sections apply and how strict severity thresholds are. A payment flow turns "no rate limiting" from P2 into P0.

### 2. Audit — code first, docs second

Work through `references/check-catalog.md` domain by domain:

- **A. Security & data**: RLS coverage per table, service-role key exposure, server-action validation, authz on every mutation, secret hygiene, personal-data inventory
- **B. Quality & performance**: error/loading/empty states in the wild, N+1 and missing indexes, bundle size, Lighthouse pass, mobile viewport, cross-browser smoke
- **C. Deploy & ops readiness**: env-var completeness, migration reproducibility on a clean DB, rollback plan executability, monitoring/alerting actually wired (stage-7 preventions implemented?), backups
- **D. User, legal & marketing**: 404/500 pages, meta/OG/favicon, robots+sitemap, privacy policy & terms pages, analytics events for the core funnel, onboarding copy for a user with zero context

Every finding is an LR-### with severity **P0 (launch blocker) / P1 (fix before launch) / P2 (fix within first week) / P3 (backlog)**, concrete evidence (file path + line, or reproduction steps), and upstream ID references. No finding without evidence — "seems insecure" is not a finding.

### 3. Review findings with the user

Present the audit as a severity-sorted table. The user confirms severity calls and picks the remediation batch (default: all P0+P1). Severity disagreements are fine — record the decision and rationale in `pipeline-state.json`'s `decisions` array.

### 4. Remediate

Fix confirmed findings directly in code, following stage-5 stack conventions (server actions, zod, migrations-only schema changes, RLS on). Each fix gets a remediation-log entry: finding ID, change summary, files touched, commit message with LR ID. Doc-level gaps (missing policy, missing ops runbook) route back to the owning stage document — fix the source, not a copy.

### 5. Re-verify

For each remediated finding, re-run the check that produced it, plus any stage-6 TCs touching the same FEAT (regression guard). A fix without re-verification stays open. Update finding status: `open → fixed → verified`.

### 6. Go/No-Go

Fill `launch-checklist.md`: every catalog item marked pass / fail / waived-with-reason. **Go requires zero P0 and zero unwaived P1.** Record the verdict, date, and remaining P2/P3 backlog in the checklist and in `pipeline-state.json` (`launch_readiness.go_no_go`). If No-Go: list exactly what blocks, loop to step 4.

## Cautions

- Audit the deployed configuration, not just localhost — env vars, domains, and keys differ between the two, and that gap is where most launch-day incidents live.
- Don't gold-plate: this skill hardens the existing MVP scope. New features found "missing" during audit go to the stage-7 improvement plan, not into the remediation batch.
- Output language rule applies: deliverables in Korean, terms glossed in English on first mention.

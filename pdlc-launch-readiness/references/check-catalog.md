# Launch Readiness Check Catalog

Checks are written for the pipeline's fixed stack (Next.js App Router + Supabase + Tailwind + zod). Each check states **how to verify** — run the verification, don't eyeball. Default severity is a starting point; step 1 scoping can raise or lower it.

## A. Security & data (보안·데이터)

| # | Check | How to verify | Default severity if failing |
|---|-------|---------------|------------------------------|
| A1 | RLS enabled on **every** table | `grep -L "enable row level security" supabase/migrations/*.sql` cross-checked against table list; query `pg_tables`/`pg_policies` on live DB | P0 |
| A2 | RLS policies match POL permissions | For each POL-### permission rule, find the policy implementing it; test with a second user account attempting cross-user access | P0 |
| A3 | `service_role` key never client-side | `grep -rn "NEXT_PUBLIC" .env* src/` for service keys; `grep -rn "service_role" src/` outside server-only modules | P0 |
| A4 | Every mutation is a server action with auth check | List all `'use server'` functions; each must resolve the session server-side before writing. No client-trusted user IDs in arguments | P0 |
| A5 | zod validation on every server-action input | Each server action parses input with a zod schema before use; unvalidated `formData.get()` passed to DB is a finding | P1 |
| A6 | Secrets hygiene | `.env*` in `.gitignore`; `git log -p --all -- .env*` shows no committed secrets; deployment env vars set separately | P0 if leaked, P1 if only local |
| A7 | Personal-data inventory | List every column holding personal data (email, name, phone...); each must have: collection consent point, POL lifecycle rule, deletion path (계정 삭제 시 처리) | P1 |
| A8 | Rate limiting / abuse guard on public endpoints | Signup, login, and any unauthenticated write path have rate limiting (middleware or Supabase built-in) or a documented waiver | P2 (P0 if payments/SMS/email sending) |
| A9 | No sensitive data in logs or error messages | Trigger errors; confirm stack traces and DB errors are not rendered to users verbatim | P1 |
| A10 | Dependency audit | `npm audit --omit=dev`; critical/high vulns fixed or waived with reason | P1 for critical, P2 for high |

## B. Quality & performance (품질·성능)

| # | Check | How to verify | Default severity |
|---|-------|---------------|------------------|
| B1 | Error / loading / empty states on every screen | Per SCR-### the spec defines 4 states; walk each screen with network throttling and an empty account | P1 |
| B2 | 404 / 500 / global error boundary | `not-found.tsx`, `error.tsx`, `global-error.tsx` exist and render branded pages; visit a garbage URL | P1 |
| B3 | N+1 queries and missing indexes | Review data-fetch paths for per-row queries; `explain analyze` the 3 heaviest queries; FK and filter columns indexed | P1 if user-visible slowness, else P2 |
| B4 | Lighthouse pass on core pages | Run Lighthouse (mobile) on landing + 2 core screens; Performance ≥ 70, Accessibility ≥ 90, no failed Core Web Vitals | P2 |
| B5 | Mobile viewport | Walk the core journey at 375px width; no horizontal scroll, tap targets usable | P1 |
| B6 | Cross-browser smoke | Core journey on Chrome + Safari (+ mobile Safari); date inputs and form quirks are the usual suspects | P2 |
| B7 | Deferred bug re-triage | Every P3–P4 bug from stage 6 re-judged with launch eyes; user-facing data-loss or money bugs cannot stay deferred | varies |
| B8 | Accessibility basics | Keyboard-only pass of the core journey; form labels, alt text, focus visibility, color contrast (WCAG AA) | P2 |
| B9 | Build is clean | `next build` passes with zero errors; no `ts-ignore` masking real type holes in mutation paths | P1 |

## C. Deploy & ops readiness (배포·운영 준비)

| # | Check | How to verify | Default severity |
|---|-------|---------------|------------------|
| C1 | Clean-environment reproducibility | Fresh clone + `supabase db reset` + documented env vars + `npm run build` = running app. Follow build-notes.md verbatim; every undocumented step is a finding | P1 |
| C2 | Production env vars complete | Diff `.env.example` (must exist) against deployment platform settings; no localhost URLs in production config | P0 |
| C3 | Domain, HTTPS, auth redirect URLs | Production domain set in Supabase Auth allowed redirect URLs; OAuth callback works on the real domain, not just localhost | P0 |
| C4 | Rollback plan is executable | Stage-7 redeploy-plan.md rollback steps actually tried once (deploy platform rollback + migration down-path or restore point) | P1 |
| C5 | Monitoring & alerting wired | Every stage-7 OPS "prevention: add monitoring" item exists for real: error tracking (e.g., Sentry) receiving events, Supabase log alerts configured, uptime check on the production URL | P1 |
| C6 | Backups | Supabase backup schedule confirmed for the plan tier; restore procedure written in redeploy-plan.md | P1 |
| C7 | Launch-day runbook | Who deploys, when, smoke-test script for the first 30 minutes, rollback trigger conditions (from stage 7) | P2 |
| C8 | Cost guardrails | Supabase/Vercel plan limits vs expected traffic; spend alerts or hard caps set | P2 |

## D. User, legal & marketing (사용자·법무·마케팅)

| # | Check | How to verify | Default severity |
|---|-------|---------------|------------------|
| D1 | Privacy policy & terms pages | 개인정보처리방침 + 이용약관 pages exist, linked from footer and signup; content matches the A7 inventory (what's collected, why, retention) | P0 if collecting personal data |
| D2 | Consent at collection | Signup shows consent checkboxes where required; marketing consent separate from required consent | P1 |
| D3 | Meta / OG / favicon | Every public page has title+description; OG image renders in a link-preview debugger; favicon set | P2 |
| D4 | robots.txt + sitemap | Present, and auth-only routes excluded from indexing | P2 |
| D5 | Analytics on the core funnel | The stage-1 TO-BE success metric is measurable: events wired for signup → core action → retention proxy; verified events arrive in the dashboard | P1 (unmeasured launch = blind launch) |
| D6 | Zero-context onboarding | A person who has never seen the project completes the core journey unaided; every point of confusion is a finding | P1 |
| D7 | Support channel | A reachable contact (email/form) linked in the app; inbox actually monitored | P2 |
| D8 | Launch announcement asset | At minimum: one-paragraph launch post + screenshot set, aligned with the stage-8 portfolio hook | P3 |

## Severity definitions

- **P0 — launch blocker**: security hole, data-loss risk, legal exposure, or the app cannot run in production. No-Go until fixed.
- **P1 — fix before launch**: visible breakage or material risk on the core journey. Waivable only with explicit user sign-off and a written reason.
- **P2 — first week**: quality/polish gaps that won't hurt day one. Scheduled, not forgotten — goes into the stage-7 improvement plan.
- **P3 — backlog**: nice-to-have. Recorded and released.

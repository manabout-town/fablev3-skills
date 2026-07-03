---
name: pdlc-8-portfolio
description: PDLC pipeline stage 8 — build the Notion portfolio from stages 1–7 outputs, including a recruiter hook, live demo GIFs (8+ scenes), metrics, problem→solution mapping, and a phase-2 roadmap. Use this skill whenever the user asks to package the project, e.g. "포트폴리오 만들자", "노션에 정리해줘", "노션 포트폴리오", "프로젝트 정리해서 올리자", "이력서용 프로젝트 페이지", or after completing the pipeline. Creates pages directly when Notion MCP is connected; otherwise outputs import-ready markdown.
---

# Stage 8: Notion Portfolio

Recompose stages 1–7 into a story for the reader. This is not copying documents over — piles of documents go unread. The goal: a reviewer (recruiter, teammate, future you) confirms within minutes that "this person thinks from problem definition through operations," and can drill into any part they care about.

## Inputs

- All of `docs/` (00–07) — especially the `decisions` array in `pipeline-state.json`. **The record of decisions and pivots is this portfolio's differentiator.** A portfolio that only lists outputs and one that explains *why* are worlds apart.
- A locally runnable app from stage 5 — required for demo GIF recording.
- Set stage 8 `in_progress` on start.

## Outputs

- Notion page tree (when Notion MCP is connected) — structure per `references/notion-structure.md`
- `docs/08-portfolio/portfolio.md` — always generated (source of truth + fallback when MCP is absent)
- `docs/08-portfolio/gifs/` — demo GIF files (8+ scenes), produced per `references/demo-gif-guide.md`

## Mandatory sections (in this order)

The main page must contain sections 1–8 below. Detailed layouts and Korean copy patterns are in `references/notion-structure.md` — read it before writing anything.

1. **10-second hook** — the first screenful decides whether a recruiter keeps reading. One punchy headline (problem or outcome, not the service name), one sub-line, the hero GIF (the single most impressive scene), and a 4–6 cell project card (기간/역할/스택/성과). No greeting, no "안녕하세요" — lead with the substance.
2. **Live demo GIFs** — at least **8 scenes** covering distinct features or pages, each GIF with a one-line caption stating what to watch and the FEAT/SCR ID. With only 3 core features, reach 8+ by treating states and journeys as scenes: onboarding, each feature's happy path, a validation-error moment, an empty state, the integrated journey, responsive/mobile view. Production process: `references/demo-gif-guide.md`.
3. **30-second summary** — problem → approach → what was built → result, in 4–6 sentences of prose. Someone who reads only this section should still get the whole project.
4. **결과지표 (Outcome metrics)** — a table: stage-1 expected-impact metrics vs achieved (or measured-in-simulation) values, TC pass rate, bug find/fix counts, delivery vs plan. Label simulated numbers honestly — faking measurement kills trust in every other number.
5. **이 프로젝트에서 봐야 할 N가지** — 3–5 curated highlights, each linking to evidence: e.g., a contradicted hypothesis that changed the direction (INS/decision), an RLS design detail (db-schema), a boundary TC that caught a real bug (BUG report), an ops scenario that exposed a missing detection mechanism (OPS). Pick moments that show judgment, not effort.
6. **문제 → 해결 매핑 (full)** — the complete traceability table: every PP → the REQ it spawned → the FEAT/POL that addressed it → verification (TC result) → residual state (해소/부분/2차로 이월). Unresolved PPs stay in the table with their disposition — completeness here *is* the credibility.
7. **2차 로드맵 (Phase-2 roadmap)** — from the stage-7 improvement plan: next-sprint items with rationale, then backlog, each tied to the PP/OPS/BUG it serves. Frame as "what I would do next and why," proving the project is a living system, not a finished homework.
8. **단계별 여정 + 부록** — the 6-beat narrative (문제→검증→결정→실행→검증·운영→다음) as short highlights linking to per-stage subpages; appendix indexes all raw documents and the repo.

## Workflow

### 1. Preflight

- Unfinished stages: tell the user; choose finish-first vs publish-with-"진행 중" markers.
- Ask the audience: public (hiring) vs private archive. If public, sweep all artifacts (including GIFs) for real names, real data, API keys.
- Check Notion MCP connectivity; if connected, confirm target workspace/parent page.

### 2. Produce the demo GIFs

Follow `references/demo-gif-guide.md`: plan the 8+ scene list with the user first (scene table: scene / what it proves / FEAT·SCR / duration), bring up the stage-5 app locally, record, convert, compress, and name files `NN-feat-slug.gif`. This is the longest task in the stage — do it before writing copy, because captions come naturally after watching the GIFs.

### 3. Write the copy

Draft all 8 sections in `portfolio.md` first (single source of truth), following the layouts and **writing-voice rules** in `references/notion-structure.md`. The voice rules are non-negotiable: this must read like a 20–30대 developer who actually built the thing — formal but human, first-person experience, specifics over adjectives. Get user review on the draft *before* pushing to Notion; text iterations are 10x cheaper in markdown.

### 4. Build the Notion pages (MCP connected)

Follow the page tree and block-conversion rules in `references/notion-structure.md`. Key rules: main page carries sections 1–8; per-stage subpages carry depth; big tables show 5–10 representative rows with a link to the full doc; upload GIFs as file blocks (mind per-file size limits — see the GIF guide); Mermaid goes in as code blocks with a text summary alongside. After creation, send the user the URL list and ask them to open and verify no broken blocks.

### 5. Fallback (no MCP)

`portfolio.md` is already import-optimized (page splits marked with `---` + H1, no unsupported syntax). GIFs can't ride a markdown import — instruct the user to drag-drop them into the GIF section after import, in `NN-` filename order.

### 6. Wrap up

- Update `docs/pipeline-state.json`: stage 8 `done`, outputs include the Notion URL, `current_stage: 8` (pipeline complete).
- Deliver the completion summary: full deliverables list, elapsed time, decision highlights.
- Follow-up: if the phase-2 roadmap has next-sprint items, the cycle re-enters at stage 4 (plan update) → 5 (build). The pipeline is a loop, not a line.

## Output language rule

Deliverables in Korean, terms glossed in English on first mention. If public-facing, mask sensitive names/data.

## Completion criteria (self-check)

- [ ] All 8 mandatory sections present, in order
- [ ] 8+ demo GIFs, each captioned with FEAT/SCR ID; hero GIF in section 1
- [ ] Metrics table separates measured vs simulated honestly
- [ ] Problem→solution mapping covers 100% of PPs (including unresolved ones)
- [ ] Phase-2 roadmap items trace to PP/OPS/BUG sources
- [ ] Voice check passed (see notion-structure.md checklist) — no AI-flavored boilerplate
- [ ] (Public) sensitive-data sweep done; portfolio.md fallback exists

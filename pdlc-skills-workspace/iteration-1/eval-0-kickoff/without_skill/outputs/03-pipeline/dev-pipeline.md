# 개발 파이프라인 — RunMate

> 1주차(M0)에 이 문서대로 세팅한다. 목표: **"git push → 자동 검증 → 자동 배포"가 1주차 금요일 전에 동작.**

---

## 1. 저장소 & 브랜치 전략

- **GitHub 저장소**: `runmate` (public — 포트폴리오이므로 공개)
- **전략**: GitHub Flow (1인 프로젝트에 git-flow는 과함)

```
main ──────●────────●────────●──→  (항상 배포 가능, 머지 = 프로덕션 배포)
            \       /
   feat/crew-list ●   ← 이슈 단위 브랜치, PR로만 머지
```

- 브랜치 네이밍: `feat/…` `fix/…` `chore/…` `docs/…`
- **브랜치 보호 규칙 (main)**: PR 필수, CI 통과 필수. (1인이라 리뷰어 승인은 생략하되, PR 본문에 셀프 리뷰 체크리스트 작성 — 포트폴리오 증빙용)
- 커밋 컨벤션: Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `test:`) → 릴리스 노트 자동 생성 가능

## 2. 이슈/작업 관리

- **GitHub Issues** + **Projects (칸반)**: `Backlog → This Week → In Progress → Done`
- 이슈 라벨: `P0` `P1` `P2` / `epic:auth` `epic:crew` `epic:match` `epic:session` `epic:portfolio`
- 규칙: 커밋/PR에 `closes #12` 연결 → 추적 기록이 곧 포트폴리오
- 마일스톤: M0~M5를 GitHub Milestones로 등록 (roadmap-6weeks.md 날짜 기준)

## 3. 환경 구성 (3단)

| 환경 | 용도 | 인프라 |
|---|---|---|
| local | 개발 | `pnpm dev` + Supabase 로컬 (`supabase start`) 또는 dev 프로젝트 |
| preview | PR별 검증 | Vercel Preview 배포 (PR마다 고유 URL 자동 생성) |
| production | 시연/포트폴리오 | Vercel Production (main 머지 시 자동) + Supabase 프로젝트 |

- 환경변수: `.env.local` (git 제외) / Vercel 대시보드에 Preview·Production 분리 등록
- `.env.example` 파일을 저장소에 유지 (온보딩 문서 역할)

## 4. CI — GitHub Actions

PR과 main push마다 자동 실행. 파이프라인 정의는 [`ci.yml`](ci.yml) → 저장소의 `.github/workflows/ci.yml`로 복사.

```
PR 생성/업데이트
 ├─ 1. lint       (eslint + prettier check)
 ├─ 2. typecheck  (tsc --noEmit)
 ├─ 3. test       (vitest run — 도메인 로직 유닛)
 └─ 4. build      (next build — 빌드 깨짐 조기 발견)
      └─ Vercel이 병렬로 Preview 배포 → PR에 URL 코멘트
```

- E2E(Playwright)는 4주차부터 **main 머지 시에만** 실행 (매 PR 실행은 느려서 1인 프로젝트 리듬을 해침)
- README에 CI 배지 + Vercel 배포 배지 부착

## 5. CD — 배포

- **Vercel**: GitHub 연동만 하면 끝. `main` 머지 → 프로덕션, PR → 프리뷰
- **DB 마이그레이션**: Supabase CLI 마이그레이션 파일을 저장소에 커밋 (`supabase/migrations/`), 배포 전 수동 `supabase db push` (6주 규모에선 자동화 불필요 — 판단 근거를 ADR에 기록)
- **릴리스**: 마일스톤 완료 시 태그 (`v0.1.0`=M1 … `v1.0.0`=M4) + GitHub Release 노트

## 6. 품질 게이트

| 게이트 | 도구 | 시점 |
|---|---|---|
| 포맷/린트 | prettier + eslint | pre-commit (husky + lint-staged) & CI |
| 타입 | tsc strict | CI |
| 유닛 테스트 | vitest | CI (추천 점수·상태 전이·정원 검증은 필수 커버) |
| E2E 스모크 | playwright | main 머지 시 (4주차부터) |
| 에러 모니터링 | Sentry | production 상시 |
| 성능/접근성 | Lighthouse | M4 릴리스 전 수동 점검 |

## 7. 초기 세팅 절차 (1주차 체크리스트, 순서대로)

```bash
# 1. 저장소 & 스캐폴딩
pnpm create next-app@latest runmate --typescript --tailwind --eslint --app --src-dir
cd runmate && git remote add origin git@github.com:<me>/runmate.git

# 2. 코어 의존성
pnpm add @supabase/supabase-js @supabase/ssr @tanstack/react-query react-hook-form zod
pnpm add -D vitest @testing-library/react prettier husky lint-staged
npx shadcn@latest init

# 3. 품질 훅
npx husky init   # pre-commit: lint-staged (prettier + eslint --fix)

# 4. Supabase
npx supabase init && npx supabase login
npx supabase link --project-ref <ref>
# 스키마 v1 마이그레이션 작성 → npx supabase db push

# 5. CI/CD
mkdir -p .github/workflows && cp <이 폴더의 ci.yml> .github/workflows/ci.yml
# Vercel 대시보드: Import Git Repository → 환경변수 등록

# 6. GitHub 설정
# - main 브랜치 보호 규칙 (PR + CI 필수)
# - Issues 라벨/마일스톤 M0~M5 등록, Projects 칸반 생성
# - backlog.md의 P0 스토리를 이슈로 등록
```

**M0 완료 판정**: 위 절차 후, 빈 랜딩 페이지가 프로덕션 URL에서 열리고, 일부러 타입 에러를 낸 PR이 CI에서 빨간불이 되는 것을 확인하면 끝.

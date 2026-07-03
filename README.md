# PDLC Skills Pack for Claude Code

**기획부터 포트폴리오까지** — Claude Code에서 실행되는 8단계 제품 개발 파이프라인 스킬 모음.

아이디어 한 줄에서 시작해 AS-IS/TO-BE 분석 → 요구사항 정의 → 기능 명세 → 스프린트 계획 → 구현 → QA → 운영 → 포트폴리오까지, 실무 수준의 산출물을 일관된 문서 체계로 생성한다.

---

## 목차

- [개요](#개요)
- [파이프라인 구조](#파이프라인-구조)
- [스킬 목록](#스킬-목록)
- [설치 방법](#설치-방법)
- [사용 방법](#사용-방법)
- [문서 체계](#문서-체계)
- [기술 스택 규약](#기술-스택-규약)
- [벤치마크](#벤치마크)

---

## 개요

Claude Code의 [Skills](https://docs.anthropic.com/en/docs/claude-code/skills) 시스템을 활용한 제품 개발 파이프라인이다. 각 단계는 독립적인 스킬 파일로 존재하며, `pdlc-pipeline` 오케스트레이터가 상태를 추적하고 단계 간 이동을 관리한다.

**핵심 특징:**

- **추적성(Traceability)**: Pain Point → 요구사항 → 기능 → 화면 → 테스트 케이스까지 ID로 일관되게 연결
- **발산 후 수렴**: 킥오프와 1단계에서 Claude가 수익 모델·리텐션·커뮤니티 등 사용자가 말하지 않은 기회를 먼저 질문으로 제안(OPP-###), 2단계 인터뷰로 검증한 뒤에만 요구사항에 편입
- **게이트 검증**: 각 단계 완료 시 품질 체크리스트를 통과해야 다음 단계로 진행
- **단일 상태 파일**: `docs/pipeline-state.json`이 전체 파이프라인의 진행 상태를 관리
- **한국어 산출물**: 본문은 한국어, 업계 용어는 첫 등장 시 영어 병기

---

## 파이프라인 구조

```
아이디어
   │
   ▼
[1단계] AS-IS/TO-BE        문제를 정의한다
   │
   ▼
[2단계] 요구사항 정의       누구의 어떤 문제인지 확정한다
   │
   ▼
[3단계] 기능 명세           정확히 무엇을 어떻게 만들지 기술한다
   │
   ▼
[4단계] 스프린트 계획       언제까지 무엇을 만들지 일정을 확정한다
   │
   ▼
[5단계] 구현                동작하는 MVP를 만든다 (Next.js + Supabase)
   │
   ▼
[6단계] QA                  명세대로 동작하는지 검증한다
   │
   ▼
[7단계] 운영                출시 후 이슈에 대응하는 역량을 증명한다
   │
   ▼
[8단계] 포트폴리오          전체 과정을 이야기로 재구성한다
   │
   ▼
Notion 포트폴리오 페이지
```

---

## 스킬 목록

### `pdlc-pipeline` — 오케스트레이터

전체 파이프라인의 컨트롤 타워. 프로젝트 킥오프, 진행 상황 조회, 단계 간 게이트 검증을 담당한다.

**사용 시점:** `"프로젝트 시작하자"`, `"어디까지 했지?"`, `"다음 단계 뭐야?"`, `"파이프라인 현황 알려줘"`

---

### 1단계 `pdlc-1-asis-tobe` — AS-IS/TO-BE 프로세스 분석

현행 프로세스를 분해하고 문제를 정량적 근거와 함께 진단한 뒤, 개선된 미래 프로세스를 설계한다.

| 산출물 | 설명 |
|--------|------|
| `docs/01-process-analysis.md` | AS-IS 프로세스 분해, Pain Point(PP-###), TO-BE 설계, Mermaid 플로우차트 2개, 정량적 기대 효과 |

**Pain Point 기술 원칙:** "불편하다" 수준의 서술 금지. "건당 15분, 월 40건, 오입력률 10%" 형태의 수치 근거 필수.

---

### 2단계 `pdlc-2-requirements` — 요구사항 정의

페르소나를 정의하고, 인터뷰로 가설을 검증한 뒤, BRD(Business Requirements Document)로 요구사항을 확정한다.

| 산출물 | 설명 |
|--------|------|
| `docs/02-requirements/personas.md` | 최소 2개 페르소나 (목표·행동·불만·기술 친숙도) |
| `docs/02-requirements/interviews.md` | 페르소나별 인터뷰 설계 + 결과/시뮬레이션 |
| `docs/02-requirements/brd.md` | REQ ID + MoSCoW 우선순위 + 근거가 있는 요구사항 목록 |

---

### 3단계 `pdlc-3-spec` — 기능 명세 / 화면 설계 / 정책서

개발자가 그대로 구현할 수 있는 수준의 명세를 작성한다.

| 산출물 | 설명 |
|--------|------|
| `docs/03-spec/functional-spec.md` | FEAT-### ID, REQ 역참조, 입력·출력·예외 정의 |
| `docs/03-spec/screen-design.md` | SCR-### ID, 화면 흐름도, 구성요소·상태 4종 정의 |
| `docs/03-spec/policies.md` | POL-### ID, 권한·오류·데이터 생명주기 등 엣지케이스 규칙 |

---

### 4단계 `pdlc-4-sprint-plan` — 스프린트 계획

5단계에서 구현할 핵심 기능 3개를 확정하고 실행 가능한 일정을 만든다.

| 산출물 | 설명 |
|--------|------|
| `docs/04-plan/milestones.md` | 마일스톤별 완료 기준(exit criteria) |
| `docs/04-plan/gantt.md` | Mermaid 간트차트 (의존관계, 크리티컬 패스 포함) |
| `docs/04-plan/gantt.html` | 인터랙티브 HTML 간트 (발표용) |

---

### 5단계 `pdlc-5-build` — 구현 (Next.js + Supabase)

로컬에서 동작하는 MVP를 만든다. DB 마이그레이션 + 핵심 기능 3개.

| 산출물 | 설명 |
|--------|------|
| `docs/05-build/db-schema.md` | Mermaid ERD + 테이블 설계 문서 |
| `docs/05-build/build-notes.md` | 구현 범위, 실행 방법, 기술 의사결정 기록 |
| `supabase/migrations/*.sql` | 재현 가능한 DB 마이그레이션 파일 |
| `src/` | Next.js App Router 소스 |

→ [기술 스택 규약](#기술-스택-규약) 참조.

---

### 6단계 `pdlc-6-qa` — QA 테스트

명세(spec)를 오라클로 삼아 정상·오류·경계 시나리오를 설계하고 실행한다.

| 산출물 | 설명 |
|--------|------|
| `docs/06-qa/test-scenarios.md` | TS-### / TC-### ID, FEAT 역참조, 실행 결과 기록 |
| `docs/06-qa/bug-report-template.md` | 버그 리포트 표준 템플릿 |
| `docs/06-qa/bug-reports/BUG-###.md` | 개별 버그 리포트 |

---

### 7단계 `pdlc-7-ops` — 운영 이슈 시나리오

출시 후 실제로 일어날 수 있는 이슈 3개를 탐지→분석→대응→재발방지 형식으로 시뮬레이션한다.

| 산출물 | 설명 |
|--------|------|
| `docs/07-ops/ops-issues.md` | OPS-### ID, 3개 이슈 (서로 다른 카테고리) |
| `docs/07-ops/improvement-plan.md` | 이슈에서 도출한 개선 계획 |
| `docs/07-ops/redeploy-plan.md` | 재배포 계획 + 롤백 조건 |

---

### 애드온 `pdlc-feature-advisor` — 기능 전략 제안 (반복 스프린트 진입점)

기존 서비스를 분석해 "왜 이 기능이 이 서비스에 필요한가"를 근거와 함께 제안한다. 아이디어 나열이 아닌 문제→해결 연결이 핵심 — 근거 없는 제안은 출력하지 않는다.

**사용 시점:** `"어떤 기능 추가하면 좋을까"`, `"다음 스프린트 뭐 만들지"`, `"유저 리텐션 올리고 싶다"`, `"서비스 고도화 아이디어"`

| 산출물 | 설명 |
|--------|------|
| `FEAT-IDEA-###` 제안 목록 | 타깃 Pain Point + 페르소나 + 레퍼런스 사례 + 우선순위 점수 |
| 우선순위 요약표 | Impact × Effort 점수로 정렬된 제안 비교 |
| REQ 편입 | 선택한 제안을 BRD에 REQ-### 항목으로 자동 추가 |

**분석 기반:** 페르소나(2단계) + Pain Point(1단계) + 현재 기능(3·5단계) + 운영 이슈(7단계). 문서 없으면 코드베이스에서 직접 추론.

**도메인 패턴 라이브러리:** 커머스 / 소셜·커뮤니티 / 예약·매칭 / 생산성·SaaS / 콘텐츠·미디어 — 각 도메인별 검증된 기능 패턴을 근거로 제안.

---

### 애드온 `pdlc-seed-data` — 더미데이터 + 로직 스모크 테스트 (5단계 직후)

명세·페르소나에서 도출한 현실적인 더미데이터(시드)를 만들어 DB에 넣고, 그 데이터 위에서 로직을 실제로 한 번 돌려본다(스모크 테스트, SMK-###). 6단계 QA·적대적 검증·8단계 GIF 녹화가 빈 DB 위에서 돌아가는 일을 차단한다. 페르소나별 계정 + 지정 빈 계정(`empty@demo.test`), 경계값·에러 유발 행, RLS 경계 검증까지 포함하며 `supabase db reset` 한 번으로 동일 상태가 재현돼야 한다.

**사용 시점:** `"더미데이터 넣자"`, `"시드 데이터 만들어줘"`, `"테스트 데이터로 돌려보자"` — 또는 시드 없이 6·8단계를 요청하면 자동으로 먼저 라우팅

| 산출물 | 설명 |
|--------|------|
| `supabase/seed.sql` (또는 `scripts/seed.ts`) | 멱등(idempotent) 시드, 마이그레이션 옆에 버전 관리 |
| `docs/055-seed/seed-data.md` | 데이터셋 설계표 + SMK-### 스모크 테스트 로그 |

---

### 애드온 `pdlc-adversarial` — 적대적 검증 사이클 (6단계 ↔ 7단계 사이)

팀 블랙(최선을 다해 버그를 유발)과 팀 화이트(버그 수정 + 후속 위험 진단)가 최대 2라운드 진행한다. 한 라운드에서 팀 블랙이 버그를 하나도 못 내면 사이클을 조기 종료하고 다음 단계로 — "버그 없음"을 실패가 아닌 내구성 신호로 취급한다. QA가 명세 기반 협조적 검증이라면, 이 스킬은 명세 밖 틈을 노리는 적대적 검증이다.

**사용 시점:** `"팀 블랙 팀 화이트 돌리자"`, `"일부러 깨보자"`, `"레드팀 블루팀"`, `"내구성 검증"`

| 산출물 | 설명 |
|--------|------|
| `docs/065-adversarial/attack-log.md` | 라운드별 전 공격 시도 기록 (ATK-###, 성공·실패 무관) |
| `docs/065-adversarial/round-report.md` | 라운드별 결과 + 팀 화이트 수정 + 후속 위험 진단 + 사이클 결정 |
| `docs/06-qa/bug-reports/BUG-###.md` | 확정 버그 리포트 (6단계 번호 이어감) |

**핵심 규칙:** 팀 블랙 전체 공격을 먼저 로깅한 뒤 팀 화이트로 전환(선의의 전력 유지) / 확정 버그는 증거·재현 절차 필수 / 팀 화이트는 수정에 더해 "같은 실패 클래스가 다른 곳에도?"를 진단(RISK-###) → 미해결분은 7단계·launch-readiness로 이관.

---

### 애드온 `pdlc-launch-readiness` — 출시 전 점검 (7단계 ↔ 8단계 사이)

완성된 결과물(코드 + 문서)을 대상으로 출시 준비도를 감사(audit)하고, 발견사항을 직접 보완한 뒤 Go/No-Go를 판정한다. QA가 "명세대로 동작하는가"를 묻는다면, 이 스킬은 "낯선 사용자와 공격자 앞에 내놓아도 되는가"를 묻는다.

**사용 시점:** `"출시 전 점검하자"`, `"실제로 배포해도 되나?"`, `"출시 체크리스트"`, `"Go/No-Go 판정해줘"`

| 산출물 | 설명 |
|--------|------|
| `docs/09-launch/launch-audit.md` | LR-### 발견사항 (4개 영역 × 심각도 P0~P3, 증거 필수, FEAT/POL/OPS 역참조) |
| `docs/09-launch/remediation-log.md` | 보완 조치 + 재검증 기록 |
| `docs/09-launch/launch-checklist.md` | 전 항목 체크리스트 + Go/No-Go 판정 |

**4개 감사 영역:** ① 보안·데이터 (RLS, 키 노출, 개인정보) ② 품질·성능 (상태 화면, N+1, Lighthouse, 접근성) ③ 배포·운영 준비 (재현성, 롤백, 모니터링, 백업) ④ 사용자·법무·마케팅 (약관/개인정보처리방침, OG/SEO, 애널리틱스, 온보딩)

**Go 조건:** P0 0건 + 미승인 P1 0건. 포트폴리오 전용 프로젝트에는 선택, 실사용자 출시에는 필수.

---

### 8단계 `pdlc-8-portfolio` — 노션 포트폴리오

1~7단계의 산출물을 채용 담당자가 5분 안에 파악할 수 있는 이야기로 재구성한다.

| 산출물 | 설명 |
|--------|------|
| `docs/08-portfolio/portfolio.md` | 노션 임포트용 마크다운 원본 |
| `docs/08-portfolio/gifs/` | 라이브 데모 GIF 8개 이상 |

**필수 섹션 8개:** 10초 훅 / 라이브 데모 / 30초 요약 / 결과 지표 / 봐야 할 N가지 / 문제→해결 전체 매핑 / 2차 로드맵 / 단계별 여정

Notion MCP 연결 시 직접 페이지 생성, 미연결 시 마크다운 출력.

---

## 설치 방법

### 요구사항

- [Claude Code](https://claude.ai/code) 설치
- `~/.claude/skills/` 디렉토리 존재

### 설치

```bash
# 저장소 클론
git clone https://github.com/manabout-town/fablev3-skills.git

# 스킬 디렉토리로 복사
cp -r fablev3-skills/pdlc-pipeline ~/.claude/skills/
cp -r fablev3-skills/pdlc-1-asis-tobe ~/.claude/skills/
cp -r fablev3-skills/pdlc-2-requirements ~/.claude/skills/
cp -r fablev3-skills/pdlc-3-spec ~/.claude/skills/
cp -r fablev3-skills/pdlc-4-sprint-plan ~/.claude/skills/
cp -r fablev3-skills/pdlc-5-build ~/.claude/skills/
cp -r fablev3-skills/pdlc-6-qa ~/.claude/skills/
cp -r fablev3-skills/pdlc-7-ops ~/.claude/skills/
cp -r fablev3-skills/pdlc-8-portfolio ~/.claude/skills/
cp -r fablev3-skills/pdlc-feature-advisor ~/.claude/skills/
cp -r fablev3-skills/pdlc-seed-data ~/.claude/skills/
cp -r fablev3-skills/pdlc-adversarial ~/.claude/skills/
cp -r fablev3-skills/pdlc-launch-readiness ~/.claude/skills/
```

설치 후 Claude Code 세션을 재시작하면 스킬이 활성화된다.

---

## 사용 방법

### 새 프로젝트 시작

Claude Code에서 아래와 같이 말하면 오케스트레이터가 킥오프 인터뷰를 진행한다.

```
프로젝트 시작하자 / 파이프라인 시작해줘
새 서비스 기획부터 만들어보자
```

킥오프가 완료되면 프로젝트 디렉토리에 다음 파일이 생성된다.

```
docs/
├── 00-project-brief.md      ← 프로젝트 브리프
└── pipeline-state.json      ← 파이프라인 상태 (단일 진실 공급원)
```

### 단계별 진행

각 단계는 자연어로 요청하면 된다.

```
AS-IS 분석해줘               → 1단계 (pdlc-1-asis-tobe)
요구사항 정의하자             → 2단계 (pdlc-2-requirements)
기능명세서 만들어줘           → 3단계 (pdlc-3-spec)
간트차트 만들어줘             → 4단계 (pdlc-4-sprint-plan)
이제 구현하자                 → 5단계 (pdlc-5-build)
더미데이터 넣고 돌려보자      → 애드온 (pdlc-seed-data)
테스트 시나리오 작성해줘      → 6단계 (pdlc-6-qa)
팀 블랙 팀 화이트 돌리자      → 애드온 (pdlc-adversarial)
운영 이슈 시나리오 만들자     → 7단계 (pdlc-7-ops)
기능 제안해줘                 → 애드온 (pdlc-feature-advisor)
출시 전 점검하자              → 애드온 (pdlc-launch-readiness)
포트폴리오 정리해줘           → 8단계 (pdlc-8-portfolio)
```

### 진행 상황 확인

```
어디까지 했지? / 파이프라인 현황 알려줘
```

파이프라인 오케스트레이터가 단계별 상태, 완료 일시, 산출물 링크를 표로 정리해준다.

### 단계 건너뛰기

단계는 건너뛸 수 있다. 단, 상류 단계 산출물이 없으면 하류 품질이 떨어진다는 경고를 스킬이 명시적으로 안내한다.

---

## 문서 체계

### 디렉토리 구조

```
project-root/
├── docs/
│   ├── pipeline-state.json
│   ├── 00-project-brief.md
│   ├── 01-process-analysis.md
│   ├── 02-requirements/
│   │   ├── personas.md
│   │   ├── interviews.md
│   │   └── brd.md
│   ├── 03-spec/
│   │   ├── functional-spec.md
│   │   ├── screen-design.md
│   │   └── policies.md
│   ├── 04-plan/
│   │   ├── milestones.md
│   │   ├── gantt.md
│   │   └── gantt.html
│   ├── 05-build/
│   │   ├── db-schema.md
│   │   └── build-notes.md
│   ├── 06-qa/
│   │   ├── test-scenarios.md
│   │   ├── bug-report-template.md
│   │   └── bug-reports/
│   ├── 07-ops/
│   │   ├── ops-issues.md
│   │   ├── improvement-plan.md
│   │   └── redeploy-plan.md
│   └── 08-portfolio/
│       ├── portfolio.md
│       └── gifs/
├── supabase/
│   └── migrations/
└── src/                     ← Next.js 앱 소스
```

### ID 추적성 체계

모든 요구사항은 ID로 연결된다. 하류 문서는 반드시 상류 ID를 역참조한다.

```
PP-### (Pain Point)
  └→ INS-### (인터뷰 인사이트)
       └→ REQ-### (요구사항 / BRD)
              └→ FEAT-### (기능 명세)
                    ├→ SCR-### (화면 설계)
                    ├→ POL-### (정책)
                    └→ TC-###  (테스트 케이스)
                          └→ BUG-### (버그 리포트)
                                └→ OPS-### (운영 이슈)
```

이 체계 덕분에 "이 테스트 케이스는 어떤 요구사항을 검증하는가", "이 운영 이슈는 어느 Pain Point에서 비롯된 것인가"에 즉시 답할 수 있다.

### 상태 파일 스키마

```json
{
  "version": 1,
  "project": {
    "name": "서비스명",
    "one_liner": "무엇을 누구에게 왜",
    "type": "work | side-project",
    "created_at": "2026-07-02"
  },
  "current_stage": 3,
  "stages": {
    "1": { "status": "done", "completed_at": "2026-07-02", "summary": "..." },
    "2": { "status": "done", "completed_at": "2026-07-03", "summary": "..." },
    "3": { "status": "in_progress", "completed_at": null, "summary": "" },
    "4": { "status": "pending", ... }
  },
  "decisions": [...],
  "risks": [...]
}
```

`status` 값: `pending` → `in_progress` → `done` (예외적으로 `skipped`)

---

## 기술 스택 규약

5단계 구현에 적용되는 스택 고정 규약이다.

| 영역 | 기술 |
|------|------|
| 프레임워크 | Next.js App Router + TypeScript |
| 데이터베이스 | Supabase (Postgres + Auth) |
| 스타일 | Tailwind CSS |
| 입력 검증 | zod (서버 액션) |

**주요 규칙:**

- 데이터 변경(mutation)은 반드시 서버 액션(`'use server'`)으로 처리
- DB 스키마 변경은 마이그레이션 파일로만 (`supabase/migrations/*.sql`)
- 모든 테이블에 RLS(Row Level Security) 활성화
- `service_role` 키는 서버 환경변수 전용 — `NEXT_PUBLIC_` 접두사 절대 금지
- 커밋 메시지에 FEAT ID 포함: `feat(auth): implement login flow (FEAT-001)`

---

## 벤치마크

3개 시나리오(킥오프 / AS-IS 분석 / 스프린트 계획)를 각 3회씩, 스킬 적용 여부를 비교한 실행 결과.

| 지표 | 스킬 적용 | 스킬 미적용 | 차이 |
|------|-----------|------------|------|
| 통과율 | **100%** ± 0% | 28% ± 25% | **+72%p** |
| 실행 시간 | 237.5s ± 102.5s | 232.8s ± 71.9s | +4.7s |
| 토큰 | 57,560 ± 9,545 | 51,030 ± 4,297 | +6,530 |

스킬 없이는 게이트 체크리스트(ID 추적, Mermaid 다이어그램, 정량 근거 등)를 대부분 누락한다. 스킬 적용 시 시간과 토큰이 소폭 증가하지만 품질 일관성이 보장된다.

---

## 라이선스

MIT

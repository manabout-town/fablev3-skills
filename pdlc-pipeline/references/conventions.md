# PDLC 파이프라인 공통 규약

모든 단계 스킬이 따르는 단일 규약. 이 문서가 규약의 원본(source of truth)이며, 각 단계 스킬에 요약된 규약과 충돌하면 이 문서가 우선한다.

## 1. 산출물 디렉토리 구조

프로젝트 루트 기준:

```
docs/
├── pipeline-state.json          # 파이프라인 상태 (아래 스키마)
├── 00-project-brief.md          # 킥오프 산출물 (오케스트레이터)
├── 01-process-analysis.md       # 1단계: AS-IS/TO-BE
├── 02-requirements/
│   ├── personas.md              # 페르소나
│   ├── interviews.md            # 인터뷰 설계 + 결과/가상 시뮬레이션
│   └── brd.md                   # 요구사항 정의서 (REQ ID 부여)
├── 03-spec/
│   ├── functional-spec.md       # 기능명세서 (FEAT ID, REQ 역참조)
│   ├── screen-design.md         # 화면설계서 (SCR ID)
│   └── policies.md              # 정책서 (POL ID)
├── 04-plan/
│   ├── milestones.md            # 마일스톤 정의
│   ├── gantt.md                 # Mermaid 간트차트
│   └── gantt.html               # 인터랙티브 HTML 간트
├── 05-build/
│   ├── db-schema.md             # DB 설계 문서 + Mermaid ERD
│   └── build-notes.md           # 구현 범위, 실행 방법, 기술 의사결정
├── 06-qa/
│   ├── test-scenarios.md        # 테스트 시나리오 (TS/TC ID)
│   ├── bug-report-template.md   # 버그 리포트 템플릿
│   └── bug-reports/             # BUG-###.md 개별 리포트
├── 07-ops/
│   ├── ops-issues.md            # 운영 이슈 시나리오 3개 (OPS ID)
│   ├── improvement-plan.md      # 개선 계획
│   └── redeploy-plan.md         # 재배포 계획 (롤백 포함)
└── 08-portfolio/
    ├── portfolio.md             # 노션 업로드용 원본 (MCP 실패 시 폴백)
    └── gifs/                    # 라이브 데모 GIF (NN-feat-slug.gif, 8개 이상)
```

5단계 코드는 `docs/`가 아니라 프로젝트 루트에 둔다: Next.js 앱 소스 + `supabase/migrations/*.sql`.

## 2. pipeline-state.json 스키마

```json
{
  "version": 1,
  "project": {
    "name": "서비스명",
    "one_liner": "무엇을 누구에게 왜",
    "type": "work | side-project",
    "created_at": "2026-07-02"
  },
  "current_stage": 1,
  "stages": {
    "1": { "name": "asis-tobe",     "status": "pending", "completed_at": null, "outputs": [], "summary": "", "stale": false },
    "2": { "name": "requirements",  "status": "pending", "completed_at": null, "outputs": [], "summary": "", "stale": false },
    "3": { "name": "spec",          "status": "pending", "completed_at": null, "outputs": [], "summary": "", "stale": false },
    "4": { "name": "sprint-plan",   "status": "pending", "completed_at": null, "outputs": [], "summary": "", "stale": false },
    "5": { "name": "build",         "status": "pending", "completed_at": null, "outputs": [], "summary": "", "stale": false },
    "6": { "name": "qa",            "status": "pending", "completed_at": null, "outputs": [], "summary": "", "stale": false },
    "7": { "name": "ops",           "status": "pending", "completed_at": null, "outputs": [], "summary": "", "stale": false },
    "8": { "name": "portfolio",     "status": "pending", "completed_at": null, "outputs": [], "summary": "", "stale": false }
  },
  "opportunities": [
    { "id": "OPP-001", "hypothesis": "커뮤니티 기능이 재방문율을 높인다", "lens": "참여·리텐션", "status": "accepted | rejected | pending", "req_id": "REQ-005" }
  ],
  "decisions": [
    { "date": "2026-07-02", "stage": 2, "decision": "결제 기능을 2차 범위로 이동", "reason": "MVP 검증에 불필요" }
  ],
  "risks": [
    { "id": "RISK-001", "description": "", "mitigation": "", "status": "open | closed" }
  ]
}
```

- `status` 값: `pending` → `in_progress` → `done`, 예외적으로 `skipped`
- 각 단계 스킬은 시작 시 자기 단계를 `in_progress`로, 완료 시 `done` + `completed_at` + `outputs` + `summary`(한 줄)를 기록하고 `current_stage`를 다음 단계 번호로 올린다.
- 상류 단계가 수정되면 하류 단계에 `stale: true`를 표시한다.

## 3. ID 체계와 추적성(Traceability)

실무 문서의 핵심은 추적성이다. 모든 요구는 ID로 연결되어야 "이 테스트가 어느 요구를 검증하는가"에 답할 수 있다.

| ID | 대상 | 부여 단계 | 예시 |
|---|---|---|---|
| OPP-### | 확장 기회 가설 | 0 (킥오프, 발산 단계) | OPP-001 |
| PP-### | Pain Point | 1 (프로세스 분석) | PP-001 |
| INS-### | 인터뷰 인사이트 | 2 (인터뷰) | INS-001 |
| REQ-### | 요구사항 (FR 001~, NFR 101~) | 2 (BRD) | REQ-001 |
| FEAT-### | 기능 | 3 (기능명세) | FEAT-001 |
| SCR-### | 화면 | 3 (화면설계) | SCR-001 |
| POL-### | 정책 | 3 (정책서) | POL-001 |
| MS-# | 마일스톤 | 4 | MS-1 |
| TS-### / TC-### | 테스트 시나리오 / 케이스 | 6 | TS-001 / TC-001 |
| BUG-### | 버그 | 6 | BUG-001 |
| OPS-### | 운영 이슈 | 7 | OPS-001 |
| RISK-### | 리스크 | 전 단계 | RISK-001 |

연결 방향: `PP/INS → REQ → FEAT → SCR/POL → TC → BUG → OPS`. 하류 문서는 반드시 상류 ID를 역참조 컬럼으로 갖는다 (예: 기능명세 표에 "관련 요구사항: REQ-003" 컬럼).

OPP는 PP와 나란히 REQ의 근거가 될 수 있는 별도 진입점이다: `OPP → (2단계 인터뷰 검증) → REQ`. 검증 없이 OPP에서 곧바로 Must REQ가 되는 것은 금지 — 검증 전 OPP 기반 REQ는 Could 이하로만 편입한다.

## 4. 언어 규칙

한국어 본문 + 업계 용어 첫 등장 시 영어 병기. 예: "비기능 요구사항(NFR, Non-Functional Requirements)". 코드 주석·커밋 메시지·변수명은 영어.

## 5. 단계별 게이트 체크리스트 (오케스트레이터 검증용)

- **1단계**: AS-IS 프로세스가 단계별로 분해되어 있고, 각 pain point가 정량/정성 근거를 가지며, TO-BE가 pain point와 1:1 이상 매핑되는가. Mermaid 플로우차트 2개(AS-IS, TO-BE) 존재.
- **2단계**: 페르소나 최소 2개, 인터뷰 질문이 페르소나별 가설 검증형인가, BRD의 모든 요구에 REQ ID + 우선순위(MoSCoW) + 근거(인터뷰/분석 출처)가 있는가.
- **3단계**: 모든 FEAT가 REQ를 역참조하는가, 화면설계에 화면 흐름도와 화면별 구성요소/상태 정의가 있는가, 정책서가 엣지케이스(오류·권한·데이터 생명주기)를 다루는가.
- **4단계**: 마일스톤에 완료 기준(exit criteria)이 있는가, 간트에 의존관계와 크리티컬 패스가 표시되는가, 5단계 구현 범위(핵심 기능 3개)가 확정되었는가.
- **5단계**: 마이그레이션 SQL이 실행 가능한가, 핵심 기능 3개가 로컬에서 동작하는가, build-notes에 실행 방법과 기술 의사결정 기록이 있는가.
- **6단계**: 핵심 기능 3개 각각에 정상/오류/경계 시나리오가 있는가, TC가 FEAT를 역참조하는가, 실제 실행 결과가 기록되었는가.
- **7단계**: 이슈 3개가 서로 다른 카테고리(예: 장애/데이터/사용자 불만)인가, 각 이슈에 탐지→분석→대응→재발방지가 있는가, 재배포 계획에 롤백 조건이 있는가.
- **8단계**: 필수 섹션 8개(10초 훅 / 라이브 데모 GIF 8개+ / 30초 요약 / 결과지표 / 봐야 할 N가지 / 문제→해결 전체 매핑 / 2차 로드맵 / 단계별 여정)가 순서대로 있는가. 문제→해결 매핑이 모든 PP를 커버하는가. 의사결정 기록(decisions)이 스토리로 녹아 있고, 말투 체크(voice check)를 통과했는가.

## 6. 00-project-brief.md 템플릿

```markdown
# [서비스명] Project Brief

## 한 줄 정의
[무엇을] [누구에게] [어떤 가치를] 제공하는 서비스

## 배경과 문제의식
(3~5문장. 왜 지금 이 문제인가)

## 목표
- 비즈니스 목표:
- 사용자 목표:
- 학습/포트폴리오 목표: (side-project인 경우)

## 범위 밖 (Out of Scope)
(처음부터 하지 않기로 한 것 — 이후 단계에서 범위 방어에 사용)

## 확장 가설 (Expansion Hypotheses) — 있는 경우만
| ID | 가설 | 렌즈 | 기대 효과(메커니즘) | 비용/리스크 | 검증 방법 |
|----|------|------|---------------------|-------------|-----------|
| OPP-001 | 커뮤니티 기능이 재방문율을 높인다 | 참여·리텐션 | 구매 전 정보 탐색이 스토어 안에서 일어나 이탈 감소 | 모더레이션 부담 | 2단계 인터뷰에서 검증 |

(제안했으나 기각된 항목은 한 줄 사유와 함께 범위 밖 섹션에 기록)

## 제약 조건
- 기간:
- 팀:
- 기술: Next.js + Supabase (파이프라인 고정)
- 도메인 규제:

## 성공 기준
(측정 가능한 형태로 2~3개)
```

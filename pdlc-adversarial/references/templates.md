# 적대적 검증 사이클 산출물 템플릿

산출물은 한국어, 용어는 첫 등장 시 영어 병기 (파이프라인 언어 규칙).

## docs/065-adversarial/attack-log.md

```markdown
# [서비스명] 적대적 공격 로그 (Team Black Attack Log)

| 대상 커밋 | 기반 문서 | 환경 |
|---|---|---|
| <hash> | functional-spec v1.x, test-scenarios v1.x | <배포 환경 또는 로컬> |

## 라운드 1

| ATK ID | 공격 표면 | 가설 | 방법 | 결과 | 증거 | 비고 |
|--------|-----------|------|------|------|------|------|
| ATK-001 | 2. 인가/RLS | orders의 타 사용자 id 직접 지정 시 수정될 것 | 세션 A로 세션 B 주문 PATCH | **broke** | 200 + 데이터 변경됨, 재현 100% | → BUG-014 |
| ATK-002 | 1. 입력 경계 | 수량 음수 주문 통과 | qty=-5 서버액션 호출 | held | zod가 차단, 400 | |

라운드 1 요약: 공격 N건 / 표면 M종 / 확정 break K건

## 라운드 2 (라운드 1에서 break가 있었던 경우만)

(동일 형식. 1라운드 RISK 클래스 집중)
```

## docs/065-adversarial/round-report.md

```markdown
# 적대적 검증 라운드 리포트

## 라운드 1

- 공격 요약: N건 시도 / M개 표면 / 확정 break K건
- 발견 break: BUG-014, BUG-015
- Team White 조치:
  | BUG | 수정 요약 | 변경 파일 | 재검증 | 회귀 확인 TC |
  |-----|-----------|-----------|--------|--------------|
  | BUG-014 | orders RLS에 owner 조건 추가 | migrations/0009.sql | ATK-001 재시도 → 차단 | TC-021 pass |
- 후속 위험 진단 (Risk diagnosis):
  | RISK | 원 break | 함의(같은 실패 클래스) | 가능성 | 영향 범위 | 처리 |
  |------|----------|------------------------|--------|-----------|------|
  | RISK-003 | BUG-014 | 다른 소유자 테이블(carts, reviews)도 동일 패턴 누락 가능 | 높음 | 데이터 유출 | carts 확인·수정(닫음), reviews는 7단계 OPS 후보 |
- **사이클 결정**: break 발견 → 라운드 2 진행 (RISK-003 클래스 집중)

## 라운드 2

(동일 형식)

- **사이클 결정**: 라운드 2 종료 (2회 상한). 미해결 RISK-### → 7단계/launch-readiness로 에스컬레이션

## 조기 종료된 경우

> 라운드 1: 공격 N건 / M개 표면 / 확정 break 0건 — 빌드가 버텼다.
> 사이클을 조기 종료한다. 이는 내구성 신호이며, 억지 2라운드는 진행하지 않는다.
```

## pipeline-state.json 추가 블록

```json
{
  "adversarial": {
    "status": "done",
    "run_at": "YYYY-MM-DD",
    "rounds_run": 2,
    "attacks_total": 23,
    "breaks_found": 3,
    "breaks_fixed": 3,
    "open_risks": ["RISK-005"],
    "early_stopped": false,
    "summary": "2라운드 진행, break 3건 수정 완료. RISK-005(reviews RLS)는 7단계 OPS로 이관."
  }
}
```

숫자 `stages` 키는 건드리지 않는다. break는 BUG-###로 6단계 번호를 이어서, 위험은 RISK-###로 상태파일 `risks` 배열에 기록한다.

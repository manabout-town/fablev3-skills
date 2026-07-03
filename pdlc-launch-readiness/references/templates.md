# Launch Readiness Templates

Deliverables in Korean; terms glossed in English on first mention (pipeline output language rule).

## docs/09-launch/launch-audit.md

```markdown
# 출시 준비도 감사 보고서 (Launch Readiness Audit)

- 감사일: YYYY-MM-DD
- 대상: <서비스명> — 커밋 <hash> / 배포 환경 <URL 또는 "미배포">
- 출시 범위: <실사용자 대상 여부, 트래픽 예상, 개인정보/결제 취급 여부 — 1단계 스코핑 결과>

## 요약

| 심각도 | 건수 | 상태 |
|--------|------|------|
| P0 (출시 차단) | n | open n / verified n |
| P1 (출시 전 수정) | n | ... |
| P2 (첫 주 내) | n | ... |
| P3 (백로그) | n | ... |

## 발견사항 목록

| ID | 영역 | 심각도 | 제목 | 관련 ID | 상태 |
|----|------|--------|------|---------|------|
| LR-001 | A. 보안·데이터 | P0 | ... | POL-003, FEAT-002 | open |

---

### LR-001: <제목>

- **영역/체크**: A2 — RLS 정책과 POL 권한 일치
- **심각도**: P0 — <이 심각도인 이유 한 줄>
- **증거(Evidence)**: `supabase/migrations/0002_xxx.sql:34` — <무엇이 어떻게 잘못되어 있는지> / 재현 절차: 1) ... 2) ...
- **영향**: <공격자/사용자 관점에서 실제로 벌어지는 일>
- **관련 ID**: POL-###, FEAT-###, (있다면 OPS-###, TC-###)
- **권고 조치**: <구체적 수정 방향>
```

## docs/09-launch/remediation-log.md

```markdown
# 보완 조치 기록 (Remediation Log)

| LR ID | 조치 요약 | 변경 파일 | 커밋 | 재검증 방법 | 결과 | 상태 |
|-------|-----------|-----------|------|-------------|------|------|
| LR-001 | tasks 테이블 RLS에 owner 조건 추가 | migrations/0007_fix_rls.sql | fix(db): close RLS gap (LR-001) | 타 계정으로 조회 재시도 + TC-012 재실행 | 차단 확인, TC-012 pass | verified |

## 미조치 항목 (waived / deferred)

| LR ID | 심각도 | 사유 | 승인자 | 후속 계획 |
|-------|--------|------|--------|-----------|
```

## docs/09-launch/launch-checklist.md

```markdown
# 출시 체크리스트 (Go/No-Go Checklist)

- 판정일: YYYY-MM-DD
- **판정: GO / NO-GO**
- 근거: P0 0건, 미승인 P1 0건 (P2 n건 → improvement-plan 반영, P3 n건 → 백로그)

## A. 보안·데이터
- [ ] A1 전 테이블 RLS 활성화 — pass / fail(LR-###) / waived(사유)
- [ ] A2 RLS ↔ POL 권한 일치
- ... (check-catalog.md의 전 항목을 나열, 하나도 생략하지 않는다)

## B. 품질·성능
- [ ] B1 ...

## C. 배포·운영 준비
- [ ] C1 ...

## D. 사용자·법무·마케팅
- [ ] D1 ...

## 출시 후 첫 주 계획
- P2 항목 처리 일정: ...
- 모니터링 확인 주기: ...
- 롤백 발동 조건 (redeploy-plan.md 참조): ...
```

## pipeline-state.json addition

```json
{
  "launch_readiness": {
    "status": "done",
    "audited_at": "YYYY-MM-DD",
    "go_no_go": "go",
    "findings": { "p0": 0, "p1": 0, "p2": 3, "p3": 5 },
    "summary": "P0 2건(RLS 구멍, 프로덕션 env 누락) 발견 후 수정·재검증 완료. GO 판정."
  }
}
```

Numeric `stages` keys are untouched; this block is additive so the orchestrator's stage logic keeps working.

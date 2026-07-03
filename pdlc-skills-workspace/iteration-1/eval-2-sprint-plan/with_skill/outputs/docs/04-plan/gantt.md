# 단골도장 간트차트

기준 문서: `milestones.md` v1.0 · 기간 2026-07-06 ~ 2026-08-14 (주말 제외, 근무일 30일, 버퍼 5일 포함)

```mermaid
gantt
    title 단골도장 개발 일정 (2026-07-06 ~ 2026-08-14)
    dateFormat YYYY-MM-DD
    excludes weekends

    section MS-1 계획 확정
    스프린트 계획 확정              :crit, t1, 2026-07-06, 1d
    계획 확정                       :milestone, m1, after t1, 0d

    section MS-2 MVP 구현
    프로젝트 셋업(Next.js+Supabase) :crit, t2, after t1, 1d
    DB 스키마 설계·마이그레이션     :crit, t3, after t2, 1d
    인증 기반 작업(로그인·RLS)      :crit, t4, after t3, 2d
    FEAT-001 매장 등록·설정         :crit, t5, after t4, 2d
    FEAT-002 QR 도장 적립           :crit, t6, after t5, 3d
    FEAT-003 쿠폰 발행·사용 처리    :crit, t7, after t6, 3d
    여정 통합·E2E 점검              :crit, t8, after t7, 2d
    오류 처리·UI 정리(POL-001~005)  :t9, after t8, 2d
    구현 버퍼                       :b1, after t9, 3d
    MVP 시연 가능                   :milestone, m2, after b1, 0d

    section MS-3 QA
    테스트 시나리오·TC 작성         :crit, t10, after b1, 1d
    TC 실행 1차                     :crit, t11, after t10, 2d
    버그 수정·회귀 테스트           :crit, t12, after t11, 1d
    QA 버퍼                         :b2, after t12, 1d
    QA 완료                         :milestone, m3, after b2, 0d

    section MS-4 운영 준비
    운영 시나리오·재배포 계획       :t13, after b2, 2d
    운영 준비 완료                  :milestone, m4, after t13, 0d

    section MS-5 포트폴리오
    포트폴리오 작성·발행            :t14, after t13, 2d
    최종 버퍼                       :b3, after t14, 1d
    포트폴리오 공개                 :milestone, m5, after b3, 0d
```

- 크리티컬 패스: t1→t2→t3→t4→t5→t6→t7→t8→t10→t11→t12 (`crit` 마킹).
- 버퍼 3개(b1 3d, b2 1d, b3 1d) = 근무일의 16.7%.
- 주말 제외(`excludes weekends`): 1인 사이드 프로젝트지만 지속 가능성을 위해 주말은 계획에서 제외하고, 지연 시 버퍼로 흡수한다.

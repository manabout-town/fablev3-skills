# 러닝 크루 매칭 앱 — "RunMate" (가칭)

> 6주 완성 1인 사이드 프로젝트 · 최종 목표: 배포된 서비스 + 포트폴리오 문서화

- **킥오프일**: 2026-07-02 (수)
- **최종 마감**: 2026-08-12 (수) — 6주차 종료
- **진행자**: 박효균 (기획 · 디자인 · 개발 · 운영 1인)

## 문서 인덱스

| 단계 | 문서 | 내용 |
|---|---|---|
| 킥오프 | [01-kickoff/project-charter.md](01-kickoff/project-charter.md) | 프로젝트 헌장: 목표, 범위, 성공 기준, 리스크 |
| 킥오프 | [01-kickoff/prd-mvp.md](01-kickoff/prd-mvp.md) | MVP 제품 요구사항 (페르소나, 유저 플로우, 기능 명세) |
| 계획 | [02-plan/roadmap-6weeks.md](02-plan/roadmap-6weeks.md) | 주차별 로드맵 + 마일스톤 + 주간 운영 리듬 |
| 계획 | [02-plan/backlog.md](02-plan/backlog.md) | 우선순위화된 유저 스토리 백로그 (P0/P1/P2) |
| 파이프라인 | [03-pipeline/tech-stack.md](03-pipeline/tech-stack.md) | 기술 스택 결정 기록 (ADR) |
| 파이프라인 | [03-pipeline/dev-pipeline.md](03-pipeline/dev-pipeline.md) | Git 전략, CI/CD, 배포, 환경 구성 |
| 파이프라인 | [03-pipeline/ci.yml](03-pipeline/ci.yml) | GitHub Actions CI 워크플로 템플릿 |
| 포트폴리오 | [04-portfolio/portfolio-plan.md](04-portfolio/portfolio-plan.md) | 포트폴리오 산출물 목록 + 작성 일정 |

## 한눈에 보는 계획

```
1주차  기획 확정 + 디자인 + 프로젝트 스캐폴딩/파이프라인 가동
2주차  인증 + 프로필 (온보딩 완성)
3주차  크루 생성/탐색/필터 (핵심 도메인)
4주차  매칭(가입 신청/승인) + 러닝 세션 일정
5주차  버퍼 + 폴리싱 + 배포 안정화
6주차  포트폴리오 문서화 + 데모 영상 + 회고
```

## 첫 액션 (오늘 할 일)

1. `project-charter.md` 읽고 범위에 동의하는지 스스로 확인 — 빼고 싶은 것 있으면 지금 빼기
2. GitHub 저장소 생성 (`runmate`) → `03-pipeline/dev-pipeline.md`의 초기 세팅 절차 실행
3. 1주차 백로그(P0) 티켓을 GitHub Issues/Projects에 등록

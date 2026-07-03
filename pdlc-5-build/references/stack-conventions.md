# Next.js + Supabase 기술 컨벤션

## 스택 고정

- Next.js (App Router) + TypeScript
- Supabase: Postgres, Auth, (필요시) Storage
- 스타일: Tailwind CSS
- 검증: zod (서버 액션 입력 검증)

버전은 셋업 시점의 안정 최신(latest stable)을 사용하고 build-notes에 기록한다. 학습 곡선을 줄이려고 상태관리 라이브러리·ORM을 추가하지 마라 — 이 규모에서는 서버 컴포넌트 + Supabase 클라이언트로 충분하다.

## 디렉토리 구조

```
src/
├── app/                  # 라우트 (SCR ID와 매핑)
│   ├── (auth)/           # 인증 관련 라우트 그룹
│   └── [feature]/
│       ├── page.tsx
│       └── actions.ts    # 해당 기능의 서버 액션
├── components/
│   ├── ui/               # 범용 (Button, Input...)
│   └── [feature]/        # 기능 종속 컴포넌트
├── lib/
│   ├── supabase/
│   │   ├── server.ts     # 서버용 클라이언트 (cookies 기반)
│   │   └── client.ts     # 브라우저용 클라이언트
│   └── validations/      # zod 스키마 (명세 유효성 표의 번역본)
└── types/
supabase/
└── migrations/           # NNNN_description.sql
```

## Supabase 사용 규칙

- **로컬 개발 우선**: `supabase init` + `supabase start`(Docker)로 로컬 스택을 쓴다. Docker가 없으면 Supabase 무료 프로젝트를 쓰되 build-notes에 명시. Supabase MCP가 연결되어 있으면 스키마 적용·확인에 활용한다.
- **스키마 변경은 마이그레이션으로만**. 대시보드에서 손으로 고치지 않는다. 파일명: `0001_create_profiles.sql` 형식.
- **모든 테이블에 RLS 활성화**. 정책은 마이그레이션에 포함. 패턴:
  ```sql
  alter table public.items enable row level security;
  create policy "owner can read own items"
    on public.items for select
    using (auth.uid() = user_id);
  ```
- `service_role` 키는 서버 환경변수로만. 클라이언트 번들에 절대 노출 금지 (`NEXT_PUBLIC_` 접두사 붙이지 마라).
- 인증 필요 서비스면 `profiles` 테이블 + `auth.users` 트리거 패턴 사용:
  ```sql
  create table public.profiles (
    id uuid primary key references auth.users(id) on delete cascade,
    display_name text,
    created_at timestamptz not null default now()
  );
  ```

## 코드 컨벤션

- 데이터 변경(mutation)은 서버 액션(`'use server'`)으로. 클라이언트에서 supabase 직접 insert/update 금지 — 검증 우회 경로가 된다.
- 서버 액션 입력은 zod로 파싱 후 사용. 명세의 유효성 규칙 표와 1:1 매핑.
- 페이지 데이터 로딩은 서버 컴포넌트에서. `loading.tsx`(로딩 상태), `error.tsx`(오류 상태) 파일을 라우트마다 두어 명세의 상태 4종 중 2종을 구조적으로 커버.
- 커밋 단위: 기능 슬라이스별. 메시지: `feat(coupon): implement issue flow (FEAT-002)` 형태로 FEAT ID 포함.

## 환경변수

`.env.local.example`을 저장소에 포함:
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=   # server only, 필요시
```

## docs/05-build/db-schema.md 템플릿

```markdown
# [서비스명] DB 설계서

## 1. ERD

​```mermaid
erDiagram
    profiles ||--o{ items : owns
    profiles { uuid id PK
               text display_name }
    items    { uuid id PK
               uuid user_id FK
               timestamptz deleted_at }
​```

## 2. 테이블 정의

### items
| 컬럼 | 타입 | 제약 | 설명 | 관련 정책 |
|---|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | | |
| deleted_at | timestamptz | nullable | soft delete | POL-004 |

**인덱스**
| 인덱스 | 대상 쿼리 | 근거 |
|---|---|---|

**RLS 정책**
| 정책명 | 명령 | 조건 | 근거 POL |
|---|---|---|---|

## 3. 정책서 반영 요약
(POL → 스키마 반영 매핑)

## 4. 향후 확장
(백로그 FEAT를 위해 예약된 설계 여지 — 테이블은 만들지 않음)
```

## docs/05-build/build-notes.md 템플릿

```markdown
# [서비스명] 구현 노트

## 1. 구현 범위
| FEAT | 상태 | 구현 파일 | 비고 |
|---|---|---|---|
| FEAT-001 | 완료 | src/app/coupons/, actions.ts | |

## 2. 실행 방법 (처음부터)
1. `git clone ...` / 의존성 설치
2. Supabase 로컬 스택 기동 및 마이그레이션
3. `.env.local` 구성
4. `npm run dev` → 확인 시나리오

## 3. 기술 의사결정 기록
| 날짜 | 결정 | 대안 | 사유 | 영향 문서 |
|---|---|---|---|---|

## 4. 수동 확인 결과
| FEAT | 확인 케이스 | 결과 |
|---|---|---|

## 5. 알려진 한계 / 백로그
```

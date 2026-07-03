# 라이브 데모 GIF 제작 가이드 (8개 이상)

## 1. 장면 기획 — 기능 3개로 8+ 장면 만들기

기능 수가 아니라 **보여줄 가치가 있는 순간** 단위로 장면을 센다. 표준 구성:

| # | 장면 | 증명하는 것 | 참조 | 길이 |
|---|---|---|---|---|
| 01 | 온보딩/가입 → 첫 설정 | 첫인상, 진입 장벽 낮음 | SCR-001~002 | 6~8s |
| 02 | 기능 1 정상 플로우 | 핵심 가치 1 | FEAT-001 | 5~8s |
| 03 | 기능 2 정상 플로우 | 핵심 가치 2 | FEAT-002 | 5~8s |
| 04 | 기능 3 정상 플로우 | 핵심 가치 3 | FEAT-003 | 5~8s |
| 05 | 유효성 오류 → 안내 → 복구 | 오류 UX를 설계했음 | FEAT-###, POL-### | 5~7s |
| 06 | 빈 상태(empty state) → 첫 데이터 생성 | 상태 4종을 실제 구현했음 | SCR-### | 4~6s |
| 07 | 통합 여정 (기능 3개 관통) | 서비스가 하나로 동작함 | TS-900 | 10~15s |
| 08 | 모바일/반응형 뷰 | 반응형 대응 | - | 4~6s |
| 09+ | 권한 차단, 실시간 반영, 관리 화면 등 | 프로젝트 특성에 맞게 추가 | | |

기획 표를 먼저 사용자와 확정하고 녹화에 들어간다. 히어로 GIF(§1용)는 보통 07 통합 여정의 하이라이트 컷이다.

## 2. 녹화 — 두 가지 방법

### 방법 A: Playwright 자동 녹화 (권장 — 재현 가능)
장면별 시나리오를 스크립트로 만들어 두면 UI가 바뀔 때마다 재녹화가 공짜다.

```bash
npm i -D playwright && npx playwright install chromium
```

```ts
// scripts/record-demo.ts — 장면 1개 = 함수 1개
import { chromium } from 'playwright';

async function scene01_onboarding() {
  const browser = await chromium.launch();
  const ctx = await browser.newContext({
    viewport: { width: 1280, height: 800 },
    recordVideo: { dir: 'demo-raw/', size: { width: 1280, height: 800 } },
  });
  const page = await ctx.newPage();
  await page.goto('http://localhost:3000/signup');
  // 사람이 보는 속도로: 입력 사이에 waitForTimeout(600~900)을 넣는다
  await page.fill('[name=email]', 'demo@example.com');
  await page.waitForTimeout(800);
  await page.click('button[type=submit]');
  await page.waitForURL('**/onboarding');
  await page.waitForTimeout(1200); // 결과 화면을 읽을 시간
  await ctx.close(); // 비디오 저장됨
  await browser.close();
}
```

주의: 시드 데이터를 장면 전에 리셋하는 스크립트를 같이 두면 (`supabase db reset` + seed) 몇 번이든 동일 화면이 나온다. 시드는 `pdlc-seed-data` 애드온의 산출물을 그대로 쓴다 — 없으면 녹화 전에 그 스킬부터 실행. 빈 상태 장면(06)은 시드의 지정 빈 계정(`empty@demo.test`)으로 찍는다. 모바일 장면은 `viewport: { width: 390, height: 844 }`.

### 방법 B: 수동 녹화
- macOS: `Cmd+Shift+5` 영역 녹화 → .mov / Windows: `Win+G`
- 창 크기를 1280px 폭으로 고정하고, 마우스를 천천히, 클릭 전 0.5초 멈춤 (시청자의 눈이 따라오게)
- 개인정보·실데이터가 화면에 없는지 녹화 전 확인

## 3. GIF 변환 — ffmpeg 2-pass 팔레트

```bash
# 고품질 GIF: 팔레트 생성 → 적용 (1280px, 12fps)
ffmpeg -i scene01.webm -vf "fps=12,scale=1280:-1:flags=lanczos,palettegen" -y palette.png
ffmpeg -i scene01.webm -i palette.png \
  -filter_complex "fps=12,scale=1280:-1:flags=lanczos[x];[x][1:v]paletteuse" \
  -y 01-onboarding.gif
```

### 용량이 크면 (순서대로 시도)
1. fps 12 → 10
2. scale 1280 → 960
3. 장면을 더 짧게 자르기 (`-ss 시작 -t 길이`)

### 용량 기준
- **노션 무료 플랜: 파일당 5MB 제한.** 유료면 여유 있지만, 페이지 로딩을 위해 **장면당 3~5MB 이하**를 목표로.
- 8개 합계 30MB를 넘기면 페이지가 무거워진다 — 히어로만 고품질, 나머지는 960px/10fps로.

## 4. 파일 규약과 업로드

- 파일명: `NN-feat-slug.gif` (예: `02-coupon-issue.gif`) — NN이 §2 배치 순서
- 저장 위치: `docs/08-portfolio/gifs/`
- 캡션(볼드 1줄 + FEAT/SCR ID)을 파일명과 함께 표로 정리해 두면 노션 배치가 기계적으로 끝난다
- Notion MCP 연결 시: 파일 업로드 블록으로 삽입. 업로드가 지원되지 않는 MCP 버전이면 페이지에 자리표시 콜아웃("여기에 01-onboarding.gif")을 넣고 사용자에게 드래그 업로드 안내
- 폴백(markdown 임포트) 시: GIF는 임포트에 실리지 않음 — `NN-` 순서대로 수동 업로드 안내

## 5. 품질 체크리스트

- [ ] 8개 이상, 기획 표의 장면과 1:1
- [ ] 각 GIF가 한 가지만 보여준다 (한 GIF에 기능 2개 욱여넣기 금지, 통합 여정 제외)
- [ ] 첫 프레임만 봐도 어느 화면인지 안다 (중간부터 시작하는 GIF 금지)
- [ ] 텍스트가 읽힌다 (1280px 기준 폼 라벨이 선명)
- [ ] 개인정보·API 키·실데이터 없음
- [ ] 장면당 5MB 이하, 루프가 어색하지 않음

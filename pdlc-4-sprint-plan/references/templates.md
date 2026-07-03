# 4단계 산출물 템플릿

## docs/04-plan/milestones.md

```markdown
# [서비스명] 마일스톤 및 스프린트 계획

## 1. 확정 사항

- **핵심 구현 기능 3개**: FEAT-001, FEAT-003, FEAT-005
- **탈락 후보와 사유**: FEAT-002 — (사유)
- **전체 기간**: YYYY-MM-DD ~ YYYY-MM-DD (버퍼 N일 포함)

## 2. 마일스톤

| ID | 마일스톤 | 목표일 | 완료 기준 (Exit Criteria) | 산출물 |
|---|---|---|---|---|
| MS-1 | 설계 확정 | | 기능명세·화면설계·정책서 리뷰 완료 | 03-spec/* |
| MS-2 | MVP 구현 | | 핵심 기능 3개 로컬 시연 가능, 마이그레이션 재현 가능 | 코드, 05-build/* |
| MS-3 | QA 완료 | | TC 통과율 90%+, Critical 버그 0건 | 06-qa/* |
| MS-4 | 운영 준비 | | 운영 시나리오·재배포 계획 리뷰 완료 | 07-ops/* |
| MS-5 | 포트폴리오 공개 | | 노션 페이지 발행 | 08-portfolio/* |

## 3. WBS

| # | 작업 | 마일스톤 | 역할 | 예상 소요 | 의존 | 비고 |
|---|---|---|---|---|---|---|
| 1 | DB 스키마 설계 | MS-2 | 개발 | 1d | - | crit |
| 2 | 인증 기반 작업 | MS-2 | 개발 | 1d | 1 | |

## 4. 일정 리스크

| ID | 리스크 | 영향 | 완화 방안 |
|---|---|---|---|
| RISK-002 | | | |
```

## docs/04-plan/gantt.md (Mermaid)

```markdown
# [서비스명] 간트차트

​```mermaid
gantt
    title [서비스명] 개발 일정
    dateFormat YYYY-MM-DD
    excludes weekends

    section MS-2 MVP 구현
    DB 스키마 설계        :crit, t1, 2026-07-06, 1d
    인증 기반 작업        :t2, after t1, 1d
    FEAT-001 구현        :crit, t3, after t2, 3d
    MVP 시연 가능        :milestone, m2, after t3, 0d

    section MS-3 QA
    테스트 시나리오 작성   :t4, after t3, 1d
​```
```

- 각 마일스톤을 `section`으로, 마일스톤 자체는 `:milestone`으로 표시.
- 크리티컬 패스는 `crit`. 버퍼는 마지막에 `버퍼 :buffer, after tN, Nd`로 명시적 작업으로 넣는다.

## docs/04-plan/gantt.html 골격

아래 골격을 사용하고 `TASKS`, `MILESTONES`, `PROJECT` 만 프로젝트 데이터로 교체한다. 외부 CDN 없이 동작한다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>간트차트</title>
<style>
  :root { --bar:#4f7cf7; --crit:#e5534b; --done:#57ab5a; --line:#e2e5ea; }
  body { font-family:'Pretendard','Apple SD Gothic Neo',sans-serif; margin:24px; color:#1f2328; }
  h1 { font-size:20px; } .legend { font-size:12px; color:#57606a; margin-bottom:12px; }
  .chip { display:inline-block; width:10px; height:10px; border-radius:2px; margin:0 4px 0 12px; }
  #gantt { position:relative; border:1px solid var(--line); overflow-x:auto; }
  .row { display:flex; height:32px; border-bottom:1px solid var(--line); align-items:center; }
  .label { position:sticky; left:0; background:#fff; width:220px; min-width:220px; font-size:12px;
           padding:0 8px; border-right:1px solid var(--line); z-index:2;
           white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
  .track { position:relative; height:100%; flex:1; }
  .bar { position:absolute; top:7px; height:18px; border-radius:4px; background:var(--bar); cursor:pointer; }
  .bar.crit { background:var(--crit); } .bar.done { background:var(--done); }
  .ms { position:absolute; top:9px; width:14px; height:14px; background:#8250df; transform:rotate(45deg); }
  .grid { position:absolute; top:0; bottom:0; border-left:1px dashed var(--line); font-size:10px; color:#8b949e; padding-left:2px; }
  .today { border-left:2px solid #d29922; z-index:1; }
  #tip { position:fixed; display:none; background:#1f2328; color:#fff; font-size:12px;
         padding:8px 10px; border-radius:6px; z-index:9; max-width:280px; pointer-events:none; }
  .section-row { background:#f6f8fa; font-weight:600; }
</style>
</head>
<body>
<h1 id="title"></h1>
<div class="legend">
  <span class="chip" style="background:var(--bar)"></span>일반
  <span class="chip" style="background:var(--crit)"></span>크리티컬 패스
  <span class="chip" style="background:var(--done)"></span>완료
  <span class="chip" style="background:#8250df"></span>마일스톤
  <span class="chip" style="background:#d29922"></span>오늘
</div>
<div id="gantt"></div>
<div id="tip"></div>
<script>
const PROJECT = { title: "[서비스명] 개발 일정", start: "2026-07-06", end: "2026-08-14" };
// section: 마일스톤 그룹, crit: 크리티컬, done: 완료, dep: 의존 작업명
const TASKS = [
  { section:"MS-2 MVP 구현", name:"DB 스키마 설계", start:"2026-07-06", end:"2026-07-07", crit:true,  done:false, owner:"개발", dep:"-" },
  { section:"MS-2 MVP 구현", name:"인증 기반 작업", start:"2026-07-07", end:"2026-07-08", crit:false, done:false, owner:"개발", dep:"DB 스키마 설계" },
];
const MILESTONES = [ { name:"MS-2 MVP 시연", date:"2026-07-18" } ];

const day = 86400000, p0 = new Date(PROJECT.start), p1 = new Date(PROJECT.end);
const total = Math.round((p1 - p0) / day) + 1, pxPerDay = Math.max(18, Math.floor(1000/total));
const x = d => (Math.round((new Date(d) - p0) / day)) * pxPerDay;
const g = document.getElementById('gantt'), tip = document.getElementById('tip');
document.getElementById('title').textContent = PROJECT.title;

function addRow(labelText, cls) {
  const row = document.createElement('div'); row.className = 'row' + (cls ? ' ' + cls : '');
  const label = document.createElement('div'); label.className = 'label'; label.textContent = labelText;
  const track = document.createElement('div'); track.className = 'track'; track.style.width = total*pxPerDay + 'px';
  row.append(label, track); g.append(row); return track;
}
function decorate(track, withDates) {
  for (let i = 0; i <= total; i += 7) {
    const gr = document.createElement('div'); gr.className = 'grid'; gr.style.left = i*pxPerDay + 'px';
    if (withDates) gr.textContent = new Date(p0.getTime() + i*day).toISOString().slice(5,10);
    track.append(gr);
  }
  const t = Math.round((new Date().setHours(0,0,0,0) - p0) / day);
  if (t >= 0 && t <= total) {
    const td = document.createElement('div'); td.className = 'grid today'; td.style.left = t*pxPerDay + 'px'; track.append(td);
  }
}
decorate(addRow('', ''), true);
let cur = null;
for (const t of TASKS) {
  if (t.section !== cur) { cur = t.section; decorate(addRow(cur, 'section-row'), false); }
  const track = addRow('  ' + t.name, ''); decorate(track, false);
  const bar = document.createElement('div');
  bar.className = 'bar' + (t.crit ? ' crit' : '') + (t.done ? ' done' : '');
  bar.style.left = x(t.start) + 'px';
  bar.style.width = Math.max(pxPerDay, x(t.end) - x(t.start) + pxPerDay) + 'px';
  bar.onmousemove = e => { tip.style.display='block'; tip.style.left=(e.clientX+12)+'px'; tip.style.top=(e.clientY+12)+'px';
    tip.innerHTML = `<b>${t.name}</b><br>${t.start} ~ ${t.end}<br>담당: ${t.owner} / 의존: ${t.dep}`; };
  bar.onmouseleave = () => tip.style.display = 'none';
  track.append(bar);
}
const msTrack = addRow('마일스톤', 'section-row'); decorate(msTrack, false);
for (const m of MILESTONES) {
  const d = document.createElement('div'); d.className = 'ms'; d.style.left = x(m.date) + 'px'; d.title = `${m.name} (${m.date})`;
  msTrack.append(d);
}
</script>
</body>
</html>
```

## 작성 유의

- mermaid 코드블록 앞 제로폭 문자는 표기용 — 실제 문서에선 일반 코드블록.
- Mermaid, HTML, milestones.md 세 곳의 날짜·작업이 일치해야 한다. 데이터 확정 → 3개 파일 생성 순서로 작업하라.

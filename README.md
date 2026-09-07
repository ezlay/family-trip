# 추석 가족여행 준비물 체크리스트

세 집(엄마 · 혀비 · 혀니)이 2026년 추석 가족여행 준비물을 나눠 챙기기 위한 한 장짜리 웹 체크리스트입니다.

- 2026-09-23(수)~25(금) 2박 3일 / 인천 용유도 / 어른 6 · 초등 2 · 유아 2
- 갯벌체험 준비물과 다섯 끼 식재료를 항목별로 나눠 담습니다
- 항목별 **완료 체크**, **담당 집 지정**, **이름 수정**, **메모** 지원
- 항목 추가 · 삭제, 담당별 필터, 진행률 표시
- 맨 위에 **일정 · 식단 · 숙소 메모** 카드가 접었다 펼 수 있게 들어 있습니다
- **세 집이 같은 목록을 실시간으로 공유합니다.** 한 집에서 체크하면 다른 집 화면에 바로 반영되고, 새로고침할 필요가 없습니다

공개 주소: <https://ezlay.github.io/waterpark-checklist/>

## 구성

| 경로 | 설명 |
| --- | --- |
| `index.html` | 앱 전체. 빌드 없이 GitHub Pages 에서 그대로 서빙 |
| `worker/` | 동기화 서버. Cloudflare Worker + Durable Object (SQLite) |
| `docs/superpowers/specs/` | 설계 문서 |

화면은 GitHub Pages 가 내려주고, 체크·메모 같은 상태는 Cloudflare Worker 가 들고 있습니다.
브라우저는 Worker 와 WebSocket 으로 연결돼 변경을 주고받습니다.

`index.html` 상단의 `WORKER_HOST` 가 Worker 주소를, `ROOM` 이 방 이름을 가리킵니다.
방 이름을 바꾸면 완전히 새 목록으로 시작합니다.

## 동기화 서버 배포

```bash
cd worker && npx wrangler deploy
```

처음이라면 `npx wrangler login` 으로 한 번 로그인해야 합니다.
설정과 번들만 확인하려면 로그인 없이 `npm run check` 를 쓰면 됩니다.

로컬에서 돌려보려면 `npx wrangler dev` 로 띄운 뒤,
`index.html` 의 `WORKER_HOST` 를 `localhost:8787` 로 두면 됩니다. (`ws://` 로 자동 전환됩니다)

## 저장 방식

- 정본은 Durable Object 안의 SQLite 에 있습니다
- 브라우저의 localStorage 는 첫 화면을 즉시 그리기 위한 캐시일 뿐이라, 서버 상태가 오면 항상 덮어씁니다
- 연결이 끊긴 동안의 변경은 큐에 쌓였다가 재연결 시 순서대로 전송됩니다

## 접근 범위

저장소가 Public 이라 Worker 주소도 공개됩니다. 주소를 아는 사람은 목록을 보고 고칠 수 있습니다.
준비물 목록이라 민감도는 낮다고 보고 이렇게 두었고, 대신 `worker/src/index.js` 의 `EXPIRES_AT`
이후로는 서버가 연결을 거부합니다. 지금은 2026-09-27 로 맞춰져 있습니다.

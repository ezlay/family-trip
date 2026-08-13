# 아마존 워터파크 준비물 체크리스트

두 가족(은영네 · 현주네) 워터파크 나들이 준비물을 나눠 챙기기 위한 한 장짜리 웹 체크리스트입니다.

- 어른 4 · 초등 3 · 중학생 1 / 13:00–22:00 / 카바나 + 취사(버너·불판 대여)
- 항목별 **완료 체크**, **담당 집 지정**, **메모** 지원
- 항목 추가 · 삭제, 담당별 필터, 진행률 표시
- **두 집이 같은 목록을 실시간으로 공유합니다.** 한쪽에서 체크하면 다른 쪽 화면에 바로 반영되고, 새로고침할 필요가 없습니다

공개 주소: <https://ezlay.github.io/waterpark-checklist/>

## 구성

| 경로 | 설명 |
| --- | --- |
| `index.html` | 앱 전체. 빌드 없이 GitHub Pages 에서 그대로 서빙 |
| `worker/` | 동기화 서버. Cloudflare Worker + Durable Object (SQLite) |

화면은 GitHub Pages 가 내려주고, 체크·메모 같은 상태는 Cloudflare Worker 가 들고 있습니다.
브라우저는 Worker 와 WebSocket 으로 연결돼 변경을 주고받습니다.

`index.html` 상단의 `WORKER_HOST` 가 Worker 주소를 가리킵니다. Worker 를 새로 배포하면
이 값을 배포된 주소로 바꿔야 합니다.

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
이후로는 서버가 연결을 거부합니다. 나들이 날짜가 정해지면 그 값을 앞당기면 됩니다.

---
name: apps-in-toss
description: 인앱토스(Apps in Toss) 미니앱 개발·제출·심사 대응. **착수 전 금지 업종(복권·로또 등 사행성) 확인**, granite.config.ts 설정, ait build / ait deploy, 콘솔 등록(브랜드 아이콘·스크린샷·개인정보처리방침), 심사 반려 사유 대응, TossAds 광고 그룹 ID 발급·연동, 한국어 UI 다해상도 QA, 급여·금액 계산 로직 검증에 사용. 트리거 - "인앱토스", "Apps in Toss", "토스 미니앱", "granite.config", "ait build", "ait deploy", "토스 콘솔", "심사 반려", "미니앱 제출", "TossAds", "광고 그룹 ID", "attachBanner", "로또", "복권", "사행성".
---

# Apps in Toss 미니앱 — 실전 체크리스트

실제로 심사 반려를 맞고 고친 항목들이다. 새 미니앱을 만들거나 제출할 때 이 순서로 확인한다.

## -1. 코드를 한 줄도 쓰기 전에 — 애초에 낼 수 있는 아이템인가

**이걸 안 물어봐서 앱 하나를 통째로 버렸다.** 완성해 놓고 나서야 "복권 앱은 못 낸다"는
답을 들었다. 데이터 수집기, 코어 로직, 테스트 86개, 디자인, 아이콘, Figma 파일까지 전부
폐기했다. **아이디어를 들은 그 순간에 이 항목부터 확인한다.**

### 낼 수 없는 것 (2026-07 확인)

- **복권·로또 관련 일체** — 번호 조회, 통계, 번호 생성 전부. "구매 링크가 없다",
  "통계 조회일 뿐이다", "예측하지 않는다고 명시했다"로 우회되지 않는다.
  앱의 **소재 자체**가 걸린다.
- 그 밖의 사행성 소재(도박·베팅·경마·경륜 등)도 같은 취급으로 보는 게 안전하다.

### 확인 방법 — 추정하지 말고 물어본다

토스 콘솔의 운영 정책/금지 업종 문서를 보거나, **콘솔 문의로 먼저 물어본다.**
"이런 소재로 미니앱을 만들려는데 가능한가요"는 개발 착수 전에 답을 받을 수 있는 질문이다.
**앱 하나를 만드는 데 드는 시간에 비하면 문의 답변을 기다리는 비용은 0에 가깝다.**

### 판단이 애매하면

사용자가 사행성·금융·의료·성인 등 규제 소재를 꺼내면, **설계를 시작하기 전에
"이 소재가 인앱토스에서 허용되는지 콘솔에 확인하셨나요?"를 먼저 묻는다.**
"기능을 순화하면 통과할 것 같다"는 식의 자체 판단은 하지 않는다 — 실제로 그 판단으로
사행성 리스크를 낮추는 설계까지 해줬지만, 소재 자체가 막혀 있어 전부 무의미했다.

## 0-0. ⚠️ "고쳤다"고 말하기 전에 라이브 번들 버전을 확인한다

**이 실수가 형태를 바꿔가며 세 번 반복됐다.** 공통점은 "작업은 다 해놓고, 사용자 화면은
몇 시간~며칠간 옛날 그대로"였다는 것이다.

| 시점 | 빠진 칸 | 결과 |
|---|---|---|
| 2026-07-30 | 광고 키 발급 후 **재배포** 안 함 | 이틀간 광고 노출 0 |
| 2026-08-01 | `.env` 넣고 **번들 검증** 안 함 | 옛 키로 폴백, 노출 0 |
| 2026-08-08 | UI 버그 수정 후 **검수 통과를 안 기다림** | 사용자는 6일 전 번들을 계속 봄. 고쳤다고 보고한 버그가 그대로 재제보됨 |

**출시 동작 (2026-08-08 실측 정정):** 이미 출시된 앱의 **업데이트 번들은 검수 승인되면
자동으로 라이브가 된다**(`deployed: true`, live version == 방금 만든 번들). 콘솔 웹에서
'출시하기'를 눌러야 하는 것은 **첫 출시**뿐이다. 같은 날 제출한 12개 중 승인된 6개는
전부 자동 배포됐고, 나머지 6개는 그냥 아직 `REVIEWING`이었다.

즉 진짜 함정은 "출시 버튼"이 아니라 **검수에 걸리는 시간**이다. 제출 직후에는 사용자가
여전히 옛 번들을 본다.

**그래서 "고쳤다"고 보고하기 전에 반드시:**

```
bundle_get_live_version(miniAppId)   ← 라이브 versionName 확인
```

방금 만든 번들의 `versionName`과 **다르면 아직 아무것도 안 바뀐 것이다.**
사용자가 "아직 그대로인데"라고 하면 **버그를 다시 파기 전에 이것부터 확인한다** —
재현 안 되는 버그를 몇 시간 쫓다가 원인이 "출시를 안 했다"인 경우가 실제로 있었다.

**보고 문구도 구분해서 쓴다.** "수정했습니다"(코드) / "배포했습니다"(번들 업로드) /
"검수 넣었습니다"(REVIEWING) / **"사용자에게 나갔습니다"(승인 후 라이브)** 는 전부 다른 상태다.
사용자에게 영향이 가는 건 마지막 하나뿐이다. 앞의 셋을 마치고 "다 됐다"고 말하지 마라.

**검수 대기 중에 사용자가 직접 확인하게 하려면 `bundle_test_push`의 `privateLink`를 준다.**
`intoss-private://{appName}?_deploymentId={id}` 는 라이브와 무관하게 그 번들만 바로 실행하므로,
승인을 기다리지 않고 수정이 맞는지 검증받을 수 있다. 재현이 안 되는 버그를 고쳤을 때는
특히 이 링크로 먼저 확인받고 나서 다음 작업으로 넘어가라.

## 0. 가장 먼저 — 배포는 워킹트리가 아니라 커밋 기준으로 생각한다

**심사 반려 1순위 원인이 이거였다.** 아이콘을 새 디자인으로 교체해 놓고 커밋하지 않은 채
배포해서, 콘솔에 등록한 아이콘과 배포된 `brand.icon`이 서로 다른 세대가 됐다.

배포 전 항상:

```bash
git status --short     # 비어 있어야 한다
npm run build
npm run deploy
```

에셋(아이콘·이미지)은 특히 위험하다. 텍스트 diff와 달리 눈에 안 띈다.
`git show HEAD:assets/icon.png | md5sum` 과 `md5sum assets/icon.png` 를 비교하면 확실하다.

## 1. granite.config.ts

```ts
export default defineConfig({
  appName: 'myapp',              // src/config.ts APP_NAME과 반드시 동일
  brand: {
    displayName: '앱 이름',
    primaryColor: '#1F5FDB',
    icon: 'https://static.toss.im/appsintoss/{앱ID}/{uuid}.png',  // 콘솔 업로드 로고의 URL
  },
  web: {
    host: 'localhost',
    port: 5173,                  // vite 기본 포트와 일치시킬 것
    commands: { dev: 'vite', build: 'vite build' },
  },
  permissions: [],
  outdir: 'dist',
})
```

### appName ↔ APP_NAME 동기화

`appName`은 딥링크 `intoss://{appName}` 해석에 쓰인다. `src/config.ts`의 `APP_NAME`과
**문자 단위로 같아야** 한다. 두 곳에 흩어져 있어서 한쪽만 고치기 쉽다.

짧고 좋은 이름은 대부분 **이미 선점돼 있다.** (원하던 이름이 막혀 접미사를 붙여야 했다)
이름이 바뀌면 `granite.config.ts`, `src/config.ts`, 문서, 빌드 산출물명(`{appName}.ait`)이
전부 따라 움직인다. **개발 초기에 콘솔에서 이름부터 확보**하고 시작하는 게 낫다.

`localStorage` 키 프리픽스까지 같이 바꿀 필요는 없다. 이미 배포된 앱이라면 프리픽스를
바꾸는 순간 기존 사용자 데이터가 날아간다.

### ⚠️ brand.icon은 로컬 경로가 아니라 **콘솔 로고의 URL**이다

> 반려 사유 원문: "브랜드 아이콘 설정이 정상적으로 되어 있는지 확인해주세요.
> granite.config.ts에 콘솔에 등록한 아이콘과 동일한 아이콘을 등록해야 해요."

이 한 줄로 세 번 반려당했다. `'./assets/icon.png'` 같은 **로컬 경로를 넣으면 절대 통과하지
못한다.**

> 재발 사례 (2026-07-28, `moimsplitter` / 정산해 - 모임 정산): 반려 사유가 이 문구 그대로
> 다시 나왔다. 원인은 동일 — `icon: './assets/icon.png'`. **새 앱을 스캐폴딩할 때
> 템플릿에서 로컬 경로가 그대로 복사돼 넘어오는 것이 진짜 원인이다.**
> 새 프로젝트를 만들면 `brand.icon`부터 URL로 바꾸거나, 콘솔 업로드 전이면
> `icon: 'TODO://upload-to-console-first'` 같은 눈에 띄는 플레이스홀더를 둔다.
> 값이 문자열이기만 하면 빌드·배포가 다 성공하므로, 로컬 경로는 조용히 살아남는다.
>
> 전 프로젝트 일괄 점검:
> ```bash
> grep -rn "icon:" /c/dev/*/granite.config.ts | grep -v "https://static.toss.im"
> ```
> 출력이 있으면 그 앱들은 전부 반려 예정이다.

### ⚠️ displayName은 콘솔 앱 이름 **그 자체**여야 한다 — 부제를 붙이면 반려된다

> 반려 사유 원문: "미니앱 이름이 앱 정보등록에 제출된 이름과 동일해야 해요."

`brand.icon`과 **함께 오는 단짝 반려 사유**다. 아이콘만 고치고 배포하면 이걸로 또 반려당한다.
두 개는 항상 같이 확인한다.

실제 사례 (2026-07-28, `moimsplitter`):

| | 값 |
|---|---|
| 콘솔 '앱 정보 등록' 이름 | `모임 정산해` |
| `brand.displayName` | `정산해 - 모임 정산` ← **반려** |

원인은 **스토어 최적화 욕심**이다. 검색 노출을 노리고 `{이름} - {설명}` 꼴로 부제를 붙였는데,
콘솔에는 순수 이름만 등록돼 있어서 불일치가 됐다. `displayName`은 마케팅 문구를 넣는 곳이
아니다. 키워드는 콘솔의 키워드 입력란에서 따로 넣는다.

- 하이픈·중점·괄호로 부제를 붙이지 않는다 (`정산해 - 모임 정산`, `가계부 | 지출관리` 전부 X)
- **띄어쓰기 한 칸이 반려 사유가 된다.** `earningscalendar`는 콘솔 등록명이 `실적캘린더`(붙임)인데
  `실적 캘린더`(띄움)로 넣어서 이것만으로 또 반려당했다. 한국어 앱 이름은 맞춤법상 띄어 쓰는 게
  자연스러워 보여도 **콘솔에 붙여 등록했으면 붙여 쓴다.** 눈으로 비교하면 놓치기 쉬우니
  콘솔 값을 복사해 붙여넣는다.
- 조사 하나, 어미 하나까지 콘솔과 같아야 한다
- 어순도 다르면 안 된다 (`모임 정산해` ≠ `정산해 모임`)

두 번째 사례 (2026-07-28, `earningscalendar`)는 **반대 방향**이었다. `displayName`은
`어닝 캘린더`인데 **인앱 헤더만 `실적 캘린더`**였고, 콘솔 등록명이 `실적 캘린더`였다.
즉 틀린 쪽이 `displayName`이었다. 개발 중 화면 문구만 새 이름으로 바꾸고 config·문서를
안 따라 바꾼 전형적인 케이스다.

**교훈: 어느 한 곳을 기준으로 삼지 말고, 콘솔 등록명을 확인한 뒤 전 파일을 거기에 맞춘다.**
`granite.config.ts`가 항상 옳다고 가정하면 이 방향의 불일치를 놓친다.

**앱 이름은 한 군데서만 바뀌지 않는다.** 함께 갱신할 곳:

```bash
grep -rn "{옛 이름}" --exclude-dir=node_modules .
```

`granite.config.ts` / `index.html` `<title>` / `README.md` / `docs/STORE.md` / `docs/PRIVACY.md` /
`docs/SUBMISSION.md` / **인앱 헤더·온보딩 `<h1>`** / 스토어 썸네일 HTML. 특히 인앱 `<h1>`은
심사자가 실제로 보는 화면이라 여기가 어긋나면 또 지적받는다.

E2E가 앱 이름을 `waitText`로 검증한다면 같이 확인한다. 새 이름이 옛 이름을 부분 문자열로
포함하면(`모임 정산해` ⊃ `정산해`) 테스트는 통과해버리므로 **테스트 통과가 이름 일치의
증거가 되지 않는다.** 반드시 `granite.config.ts`를 눈으로 본다.

**눈으로 비교하지 말고 스크립트를 돌린다.** 위 규칙을 다 알고 있었는데도 2026-08-03에
`hikingtime`·`marathonpace`·`babyvaccine` 3개가 **같은 사유로 동시에 자동 반려**됐다.
`등산 소요시간 계산기` / `아기 예방접종 알리미`처럼 '계산기', '아기' 한 단어가 붙은 건데,
앱을 연달아 만들다 보면 이 차이가 눈에 안 들어온다. 제출 전 반드시:

```bash
# miniapp_list MCP 결과를 파일로 저장한 뒤
node _scaffold/check-names.mjs <miniapp_list.json>
```

콘솔 쪽 이름 필드는 `title`이다 (`name`이 아니다 — `name`은 아예 없다).
불일치가 있으면 종료 코드 1로 떨어진다.

### ⚠️ 콘솔에 넣은 한글은 종성이 깨져 저장될 수 있다 — 넣은 뒤 반드시 읽어서 확인

2026-08-03에 앱 4개가 "오타"로 반려됐는데, **로컬 코드에는 그 오타가 하나도 없었다.**
콘솔 저장값만 종성 한 글자가 어긋나 있었다.

| 보낸 값 | 저장된 값 |
|---|---|
| 꿈해**몽** | 꿈해**몸** |
| 뽑는 / 흩어져 / 셉니다 / 네겔레 / 쉥겐 | 뵑는 / 흔어져 / 셀니다 / 네겤레 / 셰겐 |

전부 초성·중성은 맞고 **종성만** 틀린다. 심사는 이걸 맞춤법 오류로 보고 반려한다.

- `miniapp_update_basic_info` 등으로 한글 텍스트를 넣은 뒤에는 **반드시 `miniapp_get`으로
  읽어서 눈으로 대조**한다. 넣었다는 사실이 제대로 저장됐다는 증거가 아니다.
- 다시 깨지면 그 음절을 피한 표현으로 우회한다. 억지로 같은 단어를 재시도하지 마라.
- `update_*` 호출은 그 자체가 앱정보 검토 재요청이라 **앱당 1회만 가능**하다
  (2회째는 `REVIEW_ALREADY_REQUESTED`). 한 앱의 수정은 전부 모아 한 번에 넘긴다.
- `miniapp_list`는 상당수 앱의 `detailDescription`을 `null`로 내려준다(50개 중 34개 실측).
  **상세설명 점검은 앱별 `miniapp_get`으로 해야 한다** — list만 훑으면 오염을 놓친다.

**재발 (2026-08-07, 전월세 도우미·포트폴리오 진단):** `review_submit`에 한글을 `\uXXXX`
이스케이프로 넣었더니 **제출 즉시 자동 반려**됐다. 반려 문구가 깨진 글자를 그대로 인용한다 —
`'꾭'`(꼭), `'쉼렸는지'`(쏠렸는지), `'잎을 수 있는지'`(잃을), `'도넫'`(도넛). 같은 세션의
`bundle_submit_review`는 멀쩡히 통과해서 "이스케이프가 안전하다"는 오판을 하기 쉽다 —
**한 번 통과한 것이 다음 번 안전을 보장하지 않는다.**

제출 전 검증 절차(앱정보는 1회성이라 필수):

```bash
# 1) 상세설명을 파일로 먼저 쓴다
# 2) 길이와 등장 음절을 덤프해 눈으로 훑는다 — 깨진 글자는 반드시 희귀 음절로 나타난다
node -e "
const t=require('fs').readFileSync('desc.txt','utf8').trim();
console.log('len',t.length);
console.log([...new Set([...t].filter(c=>c>='가'&&c<='힣'))].sort().join(''));
"
```

`꾭 넫 쉼 뵑 흔 셀 겤` 같은 낯선 음절이 목록에 있으면 그 문장을 다시 쓴다. 흔한 어휘로만
쓰면 사고 확률 자체가 내려간다(`도넛` → `원형 그래프`).

**릴리즈 노트는 `bundle_submit_review`에 바로 넣지 말고 `bundle_set_release_note`로 먼저
넣고 `bundle_list`로 읽어 대조한 뒤 제출한다.** `submit_review`는 노트 등록과 REVIEWING 전환을
한 번에 하므로, 응답을 읽고 오타를 발견해도 이미 잠긴 뒤다(2026-08-07 꿈해몽 사전에서
`펼쳐지는지`가 `펌쳐지는지`로 굳었다). 노트만 따로 넣으면 CREATED 상태라 몇 번이든 고칠 수 있다.

**4번째 재발 (2026-08-08, 리네임 6개 일괄 재제출).** `miniapp_update_basic_info` 페이로드에
한글을 `\uXXXX`로 넣어 세 군데가 깨졌다 — `흩어진`→`흔어진`(U+D6E9→U+D754),
`끝`→`끕`(U+B05D→U+B055), `담배 끊기`→`담배 끓기`(U+B04A→U+B053). **전부 종성만 한 칸 어긋난다.**
제출 응답은 `{miniAppId}` 하나뿐이라 그 자리에서는 절대 알 수 없고, 뒤늦게 `miniapp_get`으로
읽어야 보인다. 셋 다 흔한 단어인데도 틀렸다 — 어휘를 쉽게 쓰는 걸로는 못 막는다.

**5번째 재발 (2026-08-08, 배당 캘린더): `캘린더`가 `캠린더`로 나갔다.** U+CE98 → U+CEA0.
앱 **이름**이 깨진 첫 사례다 — 상세설명 오타와 달리 검색 결과·홈·공유 문구에 전부 노출된다.
사용자가 "배당 캠린더는 뭐야"라고 물어서 알았다.

**그때 내가 한 짓이 더 나쁘다: 확인하지 않고 "콘솔에는 정확히 들어가 있고 제 답변 텍스트만
틀렸습니다"라고 단정했다.** 실제로는 콘솔이 깨져 있었다. 사용자가 한 번 더 지적하지 않았으면
그대로 남았다. **검증하지 않은 것을 검증했다고 말하지 마라.**

**규칙: 콘솔에 들어가는 한글은 예외 없이 리터럴로 쓴다.** 이스케이프는 편해 보이지만
사람이 검산할 수 없고, 틀려도 조용히 저장된다. 이 사고는 릴리즈 노트에서 3번,
앱정보에서 1번 — 총 4번 반복됐다.

**`reviewState: IN_REVIEW`면 `miniapp_get`은 옛 값을 준다 — 캐시가 아니다.**
`miniapp_get`은 **승인된 스냅샷**을 돌려준다. 그래서 재제출 직후 조회하면 옛 값이 나오고,
자동 검토가 몇 초~수십 초 뒤 끝나면 그때부터 새 값이 나온다. 필드별로 다르게 보이는 것도
같은 이유다(검토 통과 시점 차이). **읽기를 반복하기 전에 `miniapp_meta_status`로
`reviewState`부터 확인할 것** — IN_REVIEW면 아무리 다시 읽어도 새 값은 안 나온다.

**앱정보는 반려 상태(REJECTED)면 재제출이 가능하다.** `isEditable: true`를 확인하고 다시
`review_submit`을 부르면 되고, 제출 후 `miniapp_meta_status`로 `reviewState`를 즉시 확인한다
— 자동 검사는 몇 초 만에 끝나므로 반려면 바로 잡을 수 있다. 단 `miniapp_get`은 **검토 중인
값이 아니라 승인된 스냅샷**을 돌려주므로 방금 제출한 문구 대조에는 쓸 수 없다.

**반려 사유 확인은 `review_list`가 아니라 `bundle_list`로 한다.** `review_list`는
`AUTO_REJECTED`라는 상태만 주고 이유를 안 준다. 실제 문구는 `bundle_list`의
`reviewReason` / `rejectMessages[]`에 있다. 요청 2초 만에 `reviewedAt`이 찍혔으면
사람 심사가 아니라 자동 검사에서 걸린 것이다.

### 반려는 세트로 온다 — 배포 전 3종 동시 확인

1. `brand.icon` = 콘솔 로고 URL (로컬 경로 X)
2. `brand.displayName` = 콘솔 앱 이름 (부제 X)
3. `appName` = `APP_NAME` = 콘솔 앱 아이디

하나만 고쳐 배포하면 나머지로 다음 라운드에 반려된다. 실제로 아이콘만 고쳐 배포했다가
곧바로 이름으로 다시 반려당했다. 파일 내용이 콘솔 업로드본과 바이트 단위로 같아도 소용없다 — 실제로 md5까지
맞춰봤지만 그대로 반려됐다.

[미니앱 브랜딩 가이드](https://developers-apps-in-toss.toss.im/design/miniapp-branding-guide.html) 원문:

> 앱인토스 콘솔에 로고 파일을 업로드하고, `granite.config.ts` 파일 상단의 `appsInToss`
> 함수에서 `brand.icon` 속성에 **동일한 로고 링크**를 입력해요.

올바른 순서:

1. 콘솔에 로고를 먼저 업로드한다
2. 콘솔 > 앱 정보에서 그 이미지의 **링크를 복사**한다
   (`https://static.toss.im/appsintoss/{앱ID}/{uuid}.png` 형태)
3. 그 URL을 `brand.icon`에 그대로 넣는다
4. 재빌드 → `ait deploy`

**아이콘 규격** (같은 문서):

> 크기는 600×600px의 **각진 정사각형**이어야 해요. 모서리가 둥근 형태는 사용할 수 없어요.
> 로고 뒤에는 반드시 배경이 있어야 하며, 라이트 모드와 다크 모드 모두에서 잘 보이는
> 배경 색상을 사용해요.

`primaryColor`는 여섯 자리 헥스 코드. 아이콘에서 실제 색을 뽑아 맞춘다. 눈대중 말고 픽셀 샘플링:

```bash
python -c "
import zlib,struct,sys
d=open(sys.argv[1],'rb').read(); i=8; idat=b''
while i<len(d):
    ln=struct.unpack('>I',d[i:i+4])[0]; t=d[i+4:i+8]; data=d[i+8:i+8+ln]
    if t==b'IHDR': w,h,bd,ct=struct.unpack('>IIBB',data[:10])
    if t==b'IDAT': idat+=data
    i+=12+ln
raw=zlib.decompress(idat); ch={0:1,2:3,3:1,4:2,6:4}[ct]; stride=w*ch
out=bytearray(); prev=bytearray(stride); p=0
for y in range(h):
    f=raw[p]; p+=1; line=bytearray(raw[p:p+stride]); p+=stride
    for x in range(stride):
        a=line[x-ch] if x>=ch else 0; b=prev[x]; c=prev[x-ch] if x>=ch else 0
        if f==1: line[x]=(line[x]+a)&255
        elif f==2: line[x]=(line[x]+b)&255
        elif f==3: line[x]=(line[x]+(a+b)//2)&255
        elif f==4:
            pp=a+b-c; pa,pb,pc=abs(pp-a),abs(pp-b),abs(pp-c)
            pr=a if (pa<=pb and pa<=pc) else (b if pb<=pc else c)
            line[x]=(line[x]+pr)&255
    out+=line; prev=line
def px(x,y):
    o=y*stride+x*ch; return '#%02X%02X%02X'%(out[o],out[o+1],out[o+2])
print('TL',px(5,5),'BR',px(w-6,h-6),'center',px(w//2,h//2))
" assets/icon-600.png
```

### brand.icon 값은 아무도 검사하지 않는 그냥 문자열이다

이 사실을 알면 위의 URL 규칙이 왜 중요한지 이해된다.

- 타입 검증은 `"string" === typeof input.icon` 이 전부다. 경로인지 URL인지, 파일이
  실제로 있는지 **아무것도 확인하지 않는다.**
- 프레임워크·CLI 어디에도 이 파일을 읽거나 업로드하는 코드가 없다. 값은 그대로
  번들 메타데이터에 실려 나간다.
- `.ait`는 protobuf 헤더 + zip 블록 구조인데, 헤더 brand 블록에 그 문자열이 그대로
  들어가고 zip 쪽엔 아이콘 이미지가 아예 없다(번들 JS + web 에셋뿐).

즉 **잘못된 값을 넣어도 빌드도 배포도 전부 성공한다.** 오류가 나지 않으니 심사에서
반려당하기 전까지 틀린 줄 모른다. 그래서 배포 후 헤더를 직접 확인하는 습관이 필요하다.

이미지는 콘솔 UI 업로드로만 등록된다. `ait deploy`는 이미지를 올리지 않는다.

헤더 내용은 이렇게 확인한다:

```bash
python -c "
import re
d=open('{appName}.ait','rb').read(); z=d.find(b'PK\x03\x04')
for s in re.findall(rb'[ -~]{4,}', d[:z]): print(s.decode())
"
```

### 배포가 'Code: 4097 이미 해당 앱 번들이 업로드되어 있어요'로 막힐 때

같은 내용의 번들이 이미 올라가 있으면 배포가 **취소**된다(`■ Canceled`). `.ait`를 콘솔에
수동 업로드한 뒤 `ait deploy`를 돌리면 반드시 만난다.

`granite.config.ts`만 고치면 이 함정에 걸리기 쉽다. config 변경은 번들 해시를 바꾸므로
보통은 통과하지만, 애매하면 `package.json`의 `version`을 올려 확실히 새 번들로 만든다.

```bash
# package.json version 0.1.0 → 0.1.1
npm run build && npm run deploy
```

## 1-1. 리텐션 — 심사 통과보다 이게 어렵다 (2026-08-02 실측)

콘솔 실데이터: 대표 앱 **D1 리텐션 1.1%, D2/D3 0%, 중앙 세션 14~34초.**
검색으로 유입은 되는데 전원이 한 번 보고 나간다. 원인은 광고나 키워드가 아니라
앱이 **"질문 1개 → 답 1개" 단발 구조**라 다시 올 이유가 없는 것이다.

새 앱을 만들 때 5개 축을 설계 단계에서 넣어라. 자세한 기준은
`C:\dev\.claude\RETENTION_RUBRIC.md`.

1. **첫 10초** — 빈 입력폼·전용 온보딩 화면 금지. 합리적 기본값으로 **열자마자 결과**가 보여야 한다.
   (일괄 점검한 37개 중 온보딩 벽이 있던 앱이 7개였고 전부 제거했다.)
2. **답 이후** — 결과만 있고 할 게 없으면 나간다. 조건 바꿔 즉시 재계산 / 비교 / 항목별 분해 / 저장 중 최소 하나.
3. **내일 올 이유** — streak, D-day, 저장값 복원, 매일 바뀌는 콘텐츠. 계산기라도 마지막 입력은 저장한다.
4. **광고가 결과를 가리지 않게** — 전면광고를 탭 전환이나 첫 결과 액션에 걸지 마라.
   **핵심 기능을 리워드 광고 뒤에 잠그지 마라** — 배당 캘린더는 다음 달 일정 전체가,
   퇴사 플래너는 체크리스트 10개 중 7개가 잠겨 있었다. 재방문 루프 자체를 광고가 막고 있었다.
5. **이모지 금지 · 숫자 위계** — 결과 수치 28px+ bold, 라벨은 작게.

## 1-2. ⚠️ 배너를 문서 맨 아래 두면 아무도 못 본다 (2026-08-02 실측)

**수익이 안 나는 1순위 원인이 이거였다.** 리텐션만의 문제가 아니다.

워크스페이스 61245, 7일(7/27~8/2) 총 광고 수익 **1,837원**. 그중 실적캘린더가 **69%**.
이유는 단순하다 — **그 앱만 목록형이라 유저가 스크롤을 한다.** 계산기·결과형 앱은 답이
첫 화면에 다 나와서 스크롤할 이유가 없고, 중앙 세션이 16초다. 그래서 문서 맨 아래
배너는 **존재조차 모른 채** 끝난다.

- 적금 메이트: **DAU 166명인 날 노출 1건.** 165명이 배너를 본 적이 없다.
- 전기요금·타로·음력·몇퍼센트: 전부 노출 1~2건.
- 광고 그룹 설정은 멀쩡했다(ENABLED, iOS/Android placement 정상). **순전히 위치 문제.**

### 해법 — `position: sticky; bottom: 0` (fixed 아님)

fixed는 문서 흐름에서 빠져 콘텐츠를 덮으므로 하단 여백을 따로 계산해야 한다.
sticky는 자기 자리를 그대로 차지해 **CTA를 안 덮고 여백 계산도 필요 없다.**

**함정 4개가 있고 전부 밟아야 통과한다. 하나만 빠져도 깨진다.**

1. **`margin-top: auto`가 반드시 필요하다.**
   sticky는 화면 밖으로 나가려는 요소를 **붙잡아 둘 뿐, 이미 보이는 요소를 아래로
   밀지 못한다.** 스크롤이 안 생기는 화면(`scrollHeight == innerHeight`)에서는
   배너가 콘텐츠 끝, 화면 중간에 그냥 남는다.
   실측: 모임 정산해 2인 1지출 결과에서 **256px**, 배당 캘린더 관심종목 1개에서 **537px**.
   부모가 flex column이어야 하고(아니면 `display:flex; flex-direction:column` 추가),
   `.app`에 `min-height: 100dvh`도 필요하다. 콘텐츠가 길면 auto가 0이 되어 sticky가 넘겨받는다.
   **shorthand `margin` 뒤에 와야 한다** — 앞에 두면 조용히 덮인다.

2. **부모의 `padding-bottom`을 상쇄해야 한다.**
   상쇄하지 않으면 `margin-top: auto`를 넣어도 **정확히 그만큼 뜬 채로** 끝난다.
   배너에 `margin-bottom: -{padding값}`. 미디어쿼리로 패딩이 바뀌면 거기서도 오버라이드.

3. **하단 탭바가 있으면 배너 `bottom` 오프셋 == 부모 `padding-bottom`.**
   두 값이 어긋나면 짧은 화면에서 딱 그 차이만큼 틈이 생긴다(배당 캘린더에서 5px).
   `calc(탭바높이 + env(safe-area-inset-bottom))`로 양쪽을 통일한다.

4. **`body { overflow-x: hidden }`이 sticky를 통째로 죽인다.**
   body가 스크롤 컨테이너가 되는데 세로 스크롤은 뷰포트가 가져가므로, body는
   "스크롤되지 않는 컨테이너"가 되고 그 안의 sticky가 전부 무력화된다.
   **`html`에만 걸어라.** html의 overflow-x는 뷰포트로 전파돼 효과는 동일하다.

### 검증은 반드시 실제 폰 높이로

**390×700에서 통과해도 390×844에서 깨진다.** 뷰포트가 클수록 스크롤이 안 생겨
1번 함정에 걸린다. 실제로 700px에서만 검증했다가 844px에서 56px 뜬 걸 놓쳤다.
빈 목록·항목 1개 같은 **가장 짧은 도달 가능 상태**를 반드시 확인할 것.
토스 앱 밖에서는 배너가 null이므로, 같은 클래스의 더미를 주입해 레이아웃만 검증한다.
확인 항목: `배너.bottom == innerHeight`(gap 0), 탭바와 안 겹침, 가로 스크롤 0.

## 2. 한국어 UI 다해상도 QA

토스 사용자 기기 폭은 320\~430px에 걸쳐 있다. 320px에서 깨지는 게 기본값이라고 생각하고 시작한다.

실제로 걸렸던 것들:

- **`<input type="time">`의 크롬 시계 아이콘** — 좁은 화면에서 시간 값을 잘라먹는다.
  `::-webkit-calendar-picker-indicator { display: none }`로 숨기고 좌우 여백을 줄인다.
- **한글 레이블 중간 줄바꿈** — "야간수당"이 "야간/수당"으로 쪼개진다.
  `word-break: keep-all`을 한글 레이블에 기본으로 깐다. 배지가 붙는 행은 `flex-wrap`으로
  배지가 다음 줄로 내려가게 한다.
- **터치 타깃** — 44×44px 미만이 없는지 자동 검출.

### ⚠️ Pretendard는 토스 앱에만 있다 — 로컬 레이아웃 QA가 통째로 헛돈다 (2026-08-08 실측)

`styles.css`는 `font-family: 'Pretendard', ...`로 **이름만** 지정하고 `@font-face`로 싣지 않는다.
그래서 **토스 앱 안에서만 Pretendard로 그려지고, 개발 PC에서는 더 좁은 대체 글꼴**(맑은 고딕 등)로
그려진다. 글자 폭이 다르니 **글꼴 폭에 의존하는 넘침 버그는 로컬에서 절대 재현되지 않는다.**

연차금고에서 사용자가 "연차 부여 기준 버튼이 카드 밖으로 넘친다"고 제보했는데,
320/360/390/430px × 초기·결과 화면 × 한국어 로캘 × 글꼴 확대까지 다 돌려도 **전부 0px**이었다.
원인은 로컬에 Pretendard가 없다는 것 하나였다.

**교훈: "로컬에서 안 넘친다"는 안 넘친다는 증거가 아니다.** 폭에 민감한 레이아웃은
재현을 기다리지 말고 **구조적으로 넘칠 수 없게** 만들어라.

### ⚠️ `flex: 1` + `word-break: keep-all` = 넘침 (min-width가 범인)

flex 아이템의 `min-width` 기본값은 `auto`라 **min-content 아래로 줄지 않는다.**
`body { word-break: keep-all }`이 깔려 있으면 `회계연도(1/1)` 같은 한글 덩어리가 통째로
안 쪼개지고, 그 폭이 제 몫보다 크면 형제를 오른쪽으로 밀어 카드 밖으로 나간다.

```css
.seg-btn, .tab, .chip {
  flex: 1 1 0;
  min-width: 0;            /* 이 한 줄이 하한을 없앤다 */
  overflow-wrap: anywhere; /* 좁으면 두 줄로 접히게 */
}
```

**긴 라벨은 버튼 안에 넣지 마라.** `회계연도(1/1) 기준`처럼 괄호 보조정보가 붙으면 폭만
잡아먹는다. 버튼은 `회계연도 기준`으로 줄이고 `(1/1)` 설명은 아래 도움말 문구로 뺀다.

전 프로젝트 점검 — `min-width: 0` 없는 `flex: 1`을 찾는다:
```bash
grep -n "flex: 1;" /c/dev/*/src/styles.css | grep -v "min-width"
```

자동화가 답이다. `scripts/visual-qa.mjs` 패턴 — **라우트 × 상태 × 뷰포트(320/360/390/430)**
전조합을 돌면서 가로 오버플로 / 요소 겹침 / 터치 타깃 미달 / 텍스트 오버플로를 검출하고
스크린샷을 남긴다. 산출물 디렉터리는 `.gitignore`에 넣는다.

## 3. 런타임 E2E 스모크

`vite preview` + `puppeteer-core`(로컬 Chrome 재사용, 다운로드 불필요)로 실제 클릭·타이핑을
돌린다. 단위 테스트가 잡아주지 않는 것들:

- **첫 사용자 플로우 완주** — 온보딩부터 결과 화면까지, 금액을 exact match로 검증
- **콘솔 에러 0건** — uncaught error / React 에러 포함
- **`NaN` / `undefined` / `Infinity` / `null원` 노출 0건** — 전 화면 텍스트 스캔.
  금액 계산 앱에서 가장 흔한 사고이고, 심사에서 바로 눈에 띈다
- **손상 `localStorage` 내성** — 쓰레기 JSON을 주입한 뒤에도 크래시 없이 초기 상태로 복구되는지.
  구버전 스키마가 남아 있는 실제 사용자를 흉내내는 것
- **빈 상태 직접 진입** — 데이터 없이 결과 라우트로 바로 들어갔을 때
- **해시 라우트 전수 방문** + 미지의 해시 → 홈 폴백
- **영속성** — 입력 후 reload

## 4. 금액 계산 로직

- **정수 최소단위로 합산하고 마지막에 한 번만 환산한다.** 근무별로 `분/60`을 해서 더하면
  부동소수점 오차로 주 900분(=15시간)이 14.999…가 되어 **주휴수당 대상에서 빠지는**
  경계 버그가 난다. 주 40시간 연장 경계도 같다. 경계값은 반드시 회귀 테스트로 고정.
- **매직 넘버를 상수로.** 월 환산 계수 4.345 같은 값이 여러 화면에 하드코딩되면 반드시 어긋난다.
- 계산은 **순수 함수로 UI와 분리**한다. `src/core/*.ts`에 몰아두면 그대로 단위 테스트된다.

### ⚠️ 출시된 앱 37개 일괄 점검에서 실제로 나온 계산 버그 (2026-08-02 실측)

전부 **심사를 통과하고 유저에게 서비스되던 중** 발견된 것들이다. 새 앱을 만들 때
아래를 체크리스트로 돌려라. 같은 유형이 반복해서 나온다.

1. **법정 요율·단가를 해마다 갱신했는지.** netpay-calc·alba-pay가 2026년인데 **2025년
   4대보험 요율**을 쓰고 있었다(국민연금 4.5%→4.75%, 건강 3.545%→3.595%, 장기요양 산식 변경).
   elec-bill은 **한전 고압 전력량요금 3구간 전부 오단가** — 아파트 사용자에게 과대 청구액을 보여줬다.
   → 요율은 `RATES` 한 곳에 모으고 **화면 문구를 `RATE_LABELS`로 그 상수에서 파생**시켜라.
   요율과 안내문이 따로 놀면 반드시 어긋난다(두 앱 모두 이 형태로 틀려 있었다).

2. **`toISOString()` 금지.** UTC라 **KST 00~09시에 날짜가 하루 어긋난다.** dividend-cal·bongtu가
   이것 때문에 새벽에 D-day가 전부 틀렸다. 항상 로컬 기준 `todayKey()`를 쓴다.

3. **`Date.setMonth`/`setDate` 오버플로.** dday-mil은 8/31 입대 +18개월이 3/01로,
   date-calc는 1/31 +1개월이 3/02로 나왔다. 민법 160조③(해당일 없으면 그 달 말일)에 맞춰
   **말일 클램프하는 `addMonths`를 직접 만들어라.** 윤년·말일 경계는 회귀 테스트로 고정.

4. **결측을 0으로 뭉개지 마라.** dividend-cal이 수집 실패한 ETF 수익률 0을 실제 "0%"로 표시했다.
   점수 모듈은 `> 0`으로 결측 취급하는데 화면은 `== null`만 봐서 **같은 데이터를 두 곳이
   다르게 해석**했다. null과 0을 타입으로 구분하고 판정 로직을 한 곳에 둔다.

5. **화면 라벨이 실제 동작과 반대인 경우.** stock-journal은 수수료를 반영해 계산하면서
   화면엔 "(수수료·세금 미반영)"이라고 하드코딩돼 있었다. 조건부 문구는 반드시 계산 플래그에서 파생.

6. **나머지(잔돈) 배분.** moim-split이 N빵 나머지 r원을 총무 한 명에게 몰아줘 지출 건마다
   최대 (인원-1)원씩 누적됐다. **합계는 맞아도 편차가 쌓인다** — 1원씩 순서대로 배분하고,
   "합계 보존 + 1인당 편차 ≤ 1원"을 랜덤 불변식 테스트로 고정해라.

7. **UI가 이미 안내하는 규칙을 계산이 안 지키는 경우.** chungyak-score는 "미성년 기간 최대 5년"이라
   안내하면서 `scoreAccount`는 가입일 그대로 계산해 점수를 과다 산출했다.

8. **없는 데이터를 지어내지 마라.** 주기로 계산한 미래 일정에는 **"추정" 배지**를 붙이고,
   과거 내역이 없으면 "아직 제공하지 않아요"라고 쓴다. 국내 배당락일처럼 종목마다 규칙이
   제각각이면 하나로 단정하지 말고 중립 표기 + 설명을 병기한다.

## 5. 콘솔 등록 · 심사 제출

- **스크린샷** — 390×844 @2x. 앱 실제 화면을 자동 생성하는 스크립트(`scripts/store-shots.mjs`)를
  두면 UI 변경 때마다 다시 뽑기 쉽다.
- **키워드** — 콘솔에 넣은 **등록 순서가 유지**되므로 중요한 것부터.
- **개인정보처리방침** — 별도 입력란이 있다. `docs/PRIVACY.md` 내용을 그대로 넣는다.
  서버 전송이 없고 `localStorage`만 쓴다면 그 사실을 명시하는 게 심사에 유리하다.
- **네이티브 권한** — 안 쓰면 `permissions: []`로 비워 둔다. 불필요한 권한은 반려 사유가 된다.
- **심사 코멘트에 테스트 시나리오를 첨부한다.** 심사자가 그대로 따라 하면 전 기능을 볼 수 있는
  번호 매긴 순서. 특수 입력이 필요한 기능(예: 자정 넘김 근무)은 이게 없으면 확인조차 안 된다.

### ⚠️ 릴리즈 노트에 한글을 `\uXXXX` 이스케이프로 넣지 마라 (2026-08-01, 두 번 연속)

MCP `bundle_submit_review`의 `releaseNotes`에 한글을 유니코드 이스케이프로 써서 **한 세션에
두 번** 글자가 깨졌다: `껐다`가 `꾔다`로, `띠별`이 `띄별`로. 눈으로는 안 보이고 서버 응답을
읽어야만 발견된다. 과거에 오탈자(`컸디션`) 하나로 앱정보가 반려된 전례가 있어 무시할 수 없다.

**한글은 항상 리터럴로 넣는다.** 이스케이프는 손으로 못 쓰고, 틀려도 조용히 통과한다.

**3번째 재발 (2026-08-08, mydailyfortune): 또 `띠별` → `띄별`이었다.** `bundle_set_release_note`로
리터럴 한글을 넣어 정상 저장된 것까지 확인해 놓고, 뒤이은 `bundle_submit_review`에 같은 본문을
`\uXXXX`로 다시 넣어 **멀쩡한 노트를 깨진 것으로 덮어썼다.** 취소 → version 범프 → 재빌드 →
재배포 전 과정을 다시 치렀다.

**해법: `bundle_submit_review`의 `releaseNotes`를 아예 넣지 마라.**
`releaseNotes`는 필수가 아니고, **생략하면 `bundle_set_release_note`로 넣어둔 노트가 그대로
유지된다**(2026-08-08 실측 확인). 즉 올바른 순서는:

```
bundle_set_release_note(리터럴 한글)  →  응답의 releaseNote를 눈으로 대조  →
bundle_test_push  →  bundle_submit_review({deploymentId})   ← releaseNotes 생략
```

이러면 한글이 지나가는 경로가 한 곳뿐이고, 그 한 곳은 제출 전에 몇 번이든 고칠 수 있다.
같은 본문을 두 tool에 두 번 쓰는 것 자체가 사고 원인이다.

### 앱정보·릴리즈 노트에 한글을 넣기 전 필수 절차 (5번 당하고 만든 것)

눈으로 읽는 것으로는 못 잡는다. `캘`과 `캠`은 폰트에 따라 거의 같아 보인다. **기계로 검산해라.**

```bash
# 1) 넣을 문구를 스크립트에 리터럴로 적고
# 2) 등장 음절을 전부 뽑아 눈으로 훑고
# 3) 틀리기 쉬운 짝을 명시적으로 검사한 뒤에야 MCP를 부른다
node -e "
const t = '배당 캘린더';   // 여기에 리터럴로
console.log('음절:', [...new Set([...t].filter(c=>c>='가'&&c<='힣'))].sort().join(''));
for (const [ok, bad] of [['캘','캠'],['흩','흔'],['끝','끕'],['끊','끓'],['몽','몸'],['띠','띄']])
  if (t.includes(bad)) console.error('의심:', bad, '→', ok);
"
```

**그리고 넣은 뒤에는 반드시 `miniapp_get`으로 읽어 대조한다.** 저장이 됐다는 사실이
제대로 저장됐다는 증거가 아니다. `reviewState`가 IN_REVIEW면 아직 옛 값이 보이니
APPROVED가 된 뒤에 볼 것.

또한 `띠별`은 이미 **두 번** 깨진 단어다. 세 번째를 노리지 말고 `열두 동물 운세` 같은
다른 표현으로 우회한다.

그리고 **제출 직후 응답의 `releaseNote`를 반드시 눈으로 읽어라.** 여기서 못 잡으면 비용이
급격히 커진다 — 아래 때문이다.

### ⚠️ 검수 취소하면 그 번들은 끝이다 — 릴리즈 노트가 영구 잠긴다

`review_cancel` 후에는 `bundle_set_release_note`가 `릴리즈 노트 변경이 가능한 상태가 아닙니다`로
거부한다. `bundle_submit_review`도 CREATED/REJECTED만 받으므로 CANCELED 번들은 재사용 불가다.
**릴리즈 노트 오탈자 하나를 고치려면 `package.json` version 범프 → 재빌드 → 재업로드 전 과정을
다시 해야 한다.** 그러니 제출 전에 노트를 확정하는 게 훨씬 싸다.

### `bundle_submit_review`가 `이미 검수 중인 배포가 있어요`로 막힐 때

한 미니앱은 동시에 하나의 배포만 검수할 수 있다. `review_list`에 `WAITING_FOR_AUTO_APPROVE`
같은 상태로 **이전 제출이 아직 살아 있는 경우**다. `statuses` 필터에 이 값을 빼면 안 보이니
필터 없이 조회해 확인한다. 새 번들이 이전 것을 완전히 포함한다면 이전 건을 `review_cancel`하고
새 것을 낸다(취소된 쪽은 위 규칙대로 폐기된다).

### 금융·계산 도구라면 면책 고지가 필수다

결과 화면과 스토어 설명 양쪽에 **"모의 계산이며 법적 효력이 없다"**를 명시한다.
금융 상품 판매·중개가 아니라 참고용 계산 도구임이 드러나야 한다.

## 5-0. ⚠️ 전면·리워드가 "붙어는 있는데 안 뜨는" 상태를 의심하라 (2026-08-09 실측)

배너가 잘 나가고 있어도 **수익의 절반 이상이 전면광고에서 나온다.** 적금 메이트 5일 실측:

| 지면 | 노출 | eCPM | 수익 |
|---|---|---|---|
| 배너 | 2,358 | 1,222원 | 2,882원 |
| 전면 | 122 | **23,857원** | 2,911원 |

**전면 eCPM이 배너의 19배**다. 배너 노출을 두 배로 늘리는 것보다 전면 노출률을
올리는 쪽이 훨씬 크다. 그런데 실제로는 근로장려금·오늘의 운세가 **노출 0건**이었다.

### 진단 순서 — 순서를 지켜야 헛수고를 안 한다

1. `iaa_placement_group_list(includeReport=true, startDate, endDate)`로 **지면별로** 본다.
   앱 단위 리포트만 보면 배너에 묻혀서 전면이 0인 걸 못 본다.
2. `report`가 `null`이거나 `impression: 0`이면 **먼저 키가 라이브 번들에 있는지** 본다.
   `grep -oh "ait\.v2\.live\.[a-f0-9]*" dist/web/assets/*.js | sort -u`
   그리고 `bundle_get_live_version`의 배포 시각이 그 키를 넣은 커밋보다 뒤인지 확인한다.
3. 키가 멀쩡하면 **트리거 도달률**을 의심한다. 실제 사례:
   - 근로장려금: 전면을 부르는 곳이 `결과 복사하기` **하나뿐**이고 `threshold 2`였다.
     한 세션에 결과를 두 번 복사하는 사용자는 없다 → 구조적으로 영원히 0.
   - 적금 메이트: 트리거가 과세방식 칩뿐이라 **슬라이더만 만지는 대다수가 빠졌다.**

### ⚠️ 슬라이더 onChange를 그대로 세면 드래그 도중에 광고가 터진다

`type="range"`의 `onChange`는 픽셀마다 불린다. 카운터를 그대로 올리면 드래그 한 번에
문턱을 순식간에 넘겨 **손가락을 떼기도 전에 전면광고가 뜬다.** 사용자에겐 명백한
오작동이고, 콘솔 기준으로는 어뷰징 판정(지면 제한 → 앱 전체 제한) 대상이다.

정착(debounce) 후에만 1회로 세라:

```ts
let settleTimer: ReturnType<typeof setTimeout> | undefined;
export function bumpInterstitialSettled(threshold: number, delay = 900): void {
  if (settleTimer !== undefined) clearTimeout(settleTimer);
  settleTimer = setTimeout(() => { settleTimer = undefined; bumpInterstitial(threshold); }, delay);
}
```

드래그 한 번 = 1회, 숫자 입력 한 번 = 1회로 수렴한다. 테스트로 고정할 것 —
"16ms 간격 50번 호출 후에도 노출 0", "조작 중엔 안 뜨고 손 뗀 뒤에 뜬다".

**같은 값을 두 경로에서 세지 않도록 주의한다.** 정착 카운터의 의존성 배열에 이미 들어 있는
상태를 UI 핸들러에서 또 bump하면 한 번 조작에 두 번 센다.

### 노출 빈도를 올리는 변경은 한 앱씩 낸다

`abuseLevel`이 LEVEL_1이면 그 지면만 차단이지만 **LEVEL_2는 그 미니앱의 광고 서빙
전체가 30일 정지**다. 매출 상위 앱 여러 개에 동시에 넣으면 정확히 이 판정을 부른다.
**노출 0이라 잃을 baseline이 없는 앱부터** 내고, 하루 뒤 `abuseLevel`과 eCPM을 보고 넓힌다.

### `impressionPerUser`를 도달률로 읽지 마라

이 값은 **지면별 평균**이라 지면 수가 늘면 낮아진다. 0.5가 "절반만 봤다"가 아니다.
총 도달률은 `dashboard_dau`의 AU로 직접 나눠서 구한다
(적금 메이트 08-07: DAU 604 / 배너 노출 630 → 사용자당 1.04회).

## 5-1. 광고(TossAds) 키

### 광고 유형마다 키를 따로 발급받아야 한다

배너와 전면·리워드는 **각각 광고 그룹을 만들어 각각 ID를 받는다.** 하나를 받아서 유형을
바꿔가며 돌려 쓸 수 없다. 배너만 붙였다가 나중에 리워드를 추가하면 그때 또 발급받아야 한다.

- 콘솔 > 광고 > 광고 그룹에서 생성
- 형식은 `ait.v2.{채널}.{16자리 hex}` 꼴 (예: `ait.v2.live.…`)
- **발급한 미니앱에서만 동작한다.** 다른 앱에 넣으면 광고가 뜨지 않는다
- 사업자 정보·정산 정보 인증이 끝나야 실제(live) ID가 나온다
- 광고 그룹 자체도 **심사에서 반려될 수 있다.** 앱 심사와 별개 절차다

코드에서 호출부가 유형별로 다르다 — 배너는 `TossAds.attachBanner(adGroupId, el, opts)`,
전면·리워드는 `loadFullScreenAd` 계열. `TossAds.initialize`는 앱 전체에서 한 번만 부르고,
`isSupported()`로 구버전 토스 앱을 걸러낸다.

### 키는 환경변수로만, 소스에 박지 않는다

```bash
# .env  (.gitignore에 반드시 포함)
VITE_AD_GROUP_ID=ait.v2.live.xxxxxxxxxxxxxxxx
```

```ts
export const AD_GROUP_ID = (import.meta.env.VITE_AD_GROUP_ID as string | undefined) ?? '';
```

빈 문자열이면 광고 컴포넌트가 **아무것도 렌더하지 않게**(zero footprint) 만든다. 그러면
ID 미발급 상태로도 제출할 수 있고, 빈 회색 박스가 남아 고장난 것처럼 보이는 일도 없다.
`.env.example`에 키 없이 형식과 발급 방법만 적어 커밋한다.

이 ID는 클라이언트 번들에 그대로 실려 나가므로 API 시크릿 같은 비밀은 아니다. 그래도
공개 저장소에 커밋하지는 않는다 — 도용 시 내 앱의 광고 지표가 오염된다.

### ⚠️ isSupported()는 토스 앱 밖에서 동기로 throw한다 — 광고 ID 주입 직후 크래시 주의

실제 사례 (2026-07-29, `moimsplitter`): `adGroupId !== '' && TossAds.attachBanner.isSupported()`
패턴에서 ID가 빈 문자열일 때는 단락평가로 `isSupported()`가 호출되지 않아 멀쩡했는데,
**광고 ID를 주입하는 순간** 일반 브라우저(E2E·프리뷰)에서 `fetchTossAd_isSupported is not a
constant handler` 에러가 **동기로 throw**되어 해당 화면 렌더가 통째로 죽었다(흰 화면).

- `isSupported()` 호출은 반드시 try/catch로 감싸고 catch에서 `false`를 돌려준다
- `Analytics.screen/click` 등 브리지 호출도 브라우저에서는 reject된다 —
  `?.catch(() => {})`로 흡수하고, E2E 허용 콘솔 패턴에 `/ReactNativeWebView is not available/` 추가
- **광고 ID를 주입했으면 반드시 E2E를 다시 돌린다.** ID 없이 통과한 E2E는
  광고 코드 경로를 한 줄도 실행하지 않은 것이다

## 6. 제출 전 최종 점검

```bash
git status --short              # 비어 있어야 한다 (0번 항목)
npm test
npx tsc -b --noEmit
node scripts/e2e-smoke.mjs
node scripts/visual-qa.mjs
npm run build                   # {appName}.ait 생성 확인
npm run deploy                  # 아이콘·브랜드 변경은 이걸 해야 반영된다
```

그리고 콘솔에서 **브랜드 아이콘이 업로드본과 같게 보이는지 눈으로 확인**한 뒤 제출한다.

## 프로젝트에 남겨둘 것

`docs/SUBMISSION.md`에 기본 정보 표(appName / displayName / primaryColor / 아이콘 경로 /
권한 / 카테고리), 심사용 테스트 시나리오, 규정 체크리스트, 최종 점검 체크박스를 두고
**설정을 바꿀 때마다 같이 갱신한다.** 이 문서가 config와 어긋나기 시작하면 반려로 이어진다.

## 7. 콘솔 작업은 MCP로 한다 (2026-07-30 확립)

toss.im은 브라우저 자동화(Claude in Chrome)에서 안전정책상 차단된다. 대신 공식 콘솔 MCP를 쓴다:
`claude mcp add --transport http apps-in-toss-console https://mcp.toss.im/adapters/apps-in-toss-console/mcp --client-id mcp-gateway`
→ 세션 재시작 → `/mcp`에서 OAuth 인증.

- 광고 그룹 발급: `iaa_placement_group_create` — 반환 **groupId(`ait.v2.live.{hex}`)가 곧 SDK 키**다.
  BANNER는 `adStyles: ["NORMAL"]`만, INTERSTITIAL은 `categoryId` 필수(3820=생활/편의).
- 번들 검수 제출: `ait deploy` → `bundle_set_release_note` → `bundle_test_push`(isTested=true 필수)
  → `bundle_submit_review`.
- ⚠️ `bundle_submit_review`의 `releaseNotes`가 **기존 릴리즈 노트를 덮어쓴다.** 제출 페이로드에
  전체 노트(기능 목록 + 테스트 시나리오)를 그대로 넣을 것. REVIEWING 상태에선 노트 수정 불가.
- E2E 프리뷰 서버는 `--host 127.0.0.1`을 명시한다 — vite 8이 ::1에만 바인딩하면 Node fetch가
  127.0.0.1로 붙어 "preview server did not start"로 오탐된다. 실패한 E2E가 남긴 프리뷰 프로세스가
  포트를 점유해 다음 실행까지 연쇄로 깨뜨리니 종료 시 반드시 정리한다.

### ⚠️ 광고 키는 "발급"이 아니라 "발급 후 재배포"까지가 한 세트다 (2026-07-30 재발)

`moimsplitter`: 광고 그룹 3개 발급·`.env` 주입까지 해놓고 **재배포를 안 해서** 라이브 번들에
키가 빠진 채 이틀간 노출 0이었다. 유저는 들어오는데 수익 리포트에 앱이 아예 안 뜨면
이걸 의심한다. 검증: `strings dist/web/assets/*.js | grep -o "ait\.v2\.live\.[a-f0-9]*"` —
빌드 산출물에 키가 박혀 있는지 직접 확인 후 배포한다.

또한 무가드 `isSupported()` 크래시 패턴이 `mydailyfortune`·`earningscalendar`에서도 재발견됐다.
새 앱 스캐폴딩 시 BannerAd 템플릿이 복사되며 퍼진 것 — **전 프로젝트 일괄 점검**:
```bash
grep -rn "isSupported()" /c/dev/*/src --include=*.tsx | grep -v "try\|isAdSupported"
```

### ⚠️ 새로 만든 광고 그룹은 한동안 `REGISTERING`이다 — 그 키로 빌드하면 노출 0 (2026-08-01)

`iaa_placement_group_create`가 groupId를 즉시 돌려주지만 `state`는 `REGISTERING`이고, 이 상태에서는
**광고가 나가지 않는다.** 발급 직후 그 키로 빌드해 제출하면 "번들 grep은 통과하는데 노출 0"이라는,
이미 두 번 겪은 사고와 증상이 완전히 같아서 원인을 또 헤매게 된다.

**검수 제출 전에 `iaa_placement_group_list`로 `state == ENABLED`를 확인한다.** 같은 앱에 같은 형식의
ENABLED 그룹이 이미 있으면 새로 만들지 말고 그걸 쓴다(실제로 4개 앱 중 3개가 그랬다).
`iaa_placement_group_list` 응답은 미디에이션 라인까지 딸려와 수만 자에 달하니 서브에이전트로
`groupId/name/adFormat/state`만 뽑아오게 하는 편이 낫다.

### ⚠️ `.env`의 변수명이 소스와 다르면 조용히 옛 키로 폴백한다 (2026-08-01)

`sub-keeper`의 `.env.production`에 `VITE_FULLSCREEN_AD_ID`가 있었는데 소스(`src/ads/config.ts`)는
`VITE_INTERSTITIAL_AD_ID`만 읽었다. **아무도 안 읽는 죽은 변수**라 프로덕션 빌드가 `.env`의 옛 키로
폴백했고, 오류는 한 줄도 안 났다. `.env`에 키를 "넣었다"는 사실이 그 키가 쓰인다는 증거가 아니다.

검증은 항상 번들 실측으로:
```bash
grep -oh "ait\.v2\.live\.[a-f0-9]*" dist/web/assets/*.js | sort -u
```
기대한 키가 나오고 **옛 키가 사라졌는지** 둘 다 본다. `.env*`는 보통 `.gitignore` 대상이라
이 변경은 커밋에 안 남는다 — 다른 머신에서 클론해 빌드하면 키가 비어 조용히 광고 0이 된다.

### ⚠️ "유사한 -계산기" 워크스페이스 정책 반려 (2026-07-30 실측)

> 반려 원문: "동일 워크스페이스 내 유사한 앱이 확인됩니다. 같은 워크스페이스 내에서는
> 유사한 서비스/ 앱 이름 / 앱 아이콘으로 출시 어려우며" + 채널톡: "유사한 -계산기로 출시 어려움"

한 워크스페이스에 "~계산기" 이름의 앱이 여러 개면 **앱정보 단계에서 정책 반려**된다
(퇴사·청약·전월세 계산기 3종 동시 반려). 코드·문구 수정으로 해결 불가.
- 새 앱 이름에 "계산기"를 쓰지 않는다 — 플래너/노트/도우미/디데이/질문형 이름으로 차별화
- **아이콘도 서로 색·모티프가 확실히 달라야** 한다 (유사 아이콘도 명시된 반려 사유)
- 앱정보 반려는 review_list에 안 뜬다 — **반려 통지는 이메일**(noreply@business.toss.im)로 오므로
  "리젝 확인" 요청을 받으면 Gmail에서 "[앱인토스] ... 반려" 메일부터 검색한다

### image_upload_url 복구 (2026-08-01) — contentLength 필수
"존재하지 않는 정보" 에러의 정체는 어댑터 스키마 개편이었다. 이제 `contentLength`(파일 실제 byte, 최대 5MB)가 필수며
서명에 포함되므로 정확해야 한다. 플로우: image_upload_url(workspaceId, extension, contentLength) →
curl PUT (Content-Type + x-amz-acl: public-read) → publicUrl을 iconUri/images에 사용.
miniapp_create/review_submit까지 전부 MCP로 가능해짐. detailDescription 500자 제한 주의.
⚠️ 한글을 유니코드 이스케이프로 쓰다 "몽(\ubabd)"을 "몸(\ubab8)"으로 오타낸 사고 발생 — 페이로드에 한글을 직접 쓸 것.

### ⚠️ .env.local의 빈 값이 .env를 조용히 덮어쓴다 (2026-08-01, deowidaepiso 실측)

`.env`에 `VITE_AD_GROUP_ID=키`를 넣어도 `.env.local`에 `VITE_AD_GROUP_ID=`(빈 값) 줄이
있으면 **빈 값이 이긴다**(vite 우선순위: .env.local > .env). 빌드는 성공하고 광고만 조용히
빠진다. 광고 키 주입 후 번들 검증(`grep -o "ait\.v2\.live\.[a-f0-9]*" dist/web/assets/*.js`)이
실패하면 `.env.local`부터 의심할 것.

### ⚠️ 데이터 자동 갱신 레포 — "Actions 성공"이 갱신을 보장하지 않는다 (2026-08-01, earnings-data 실측)

스크립트의 출력 경로(`ROOT = join(scriptdir, '..')` 등)가 레포 구조 변경 후 어긋나면,
매시간 워크플로가 **성공하면서도 영원히 "no changes"**로 끝난다(파일을 레포 밖에 씀).
데이터 레포는 커밋 로그가 아니라 **파일의 asOf 날짜**로 신선도를 검증할 것.
앱 쪽은 remote.asOf > seed.asOf일 때만 교체하므로, 시드가 원격보다 새로우면 원격 갱신이 무력화된다.

### MCP 광고 그룹 발급 요령 (2026-08-01 추가 확인)
- INTERSTITIAL은 `adStyles`를 넣으면 "배너 광고에서만 광고 스타일을 설정할 수 있습니다"로 거부 — categoryId+displayName만.
- REWARDED는 adStyles ["MOMENT_VIDEO"] + rewardSettings{unitAmount,unitType} + categoryId 필수.
- ait CLI가 4031(권한 없음)일 때: MCP bundle_upload→S3 PUT(Content-Type: application/zip)→bundle_upload_complete→bundle_test_push로 완전 대체 가능. deploymentId는 `ait build` 로그의 값을 그대로 사용.
  - **`application/zip`이 아니면 403 `SignatureDoesNotMatch`가 뜬다.** presign이 `content-type`을 서명에 포함하기 때문에 값이 1글자만 달라도 실패한다. `application/octet-stream`은 안 된다(2026-08-03 실측: zip만 200, octet-stream·gzip·x-tar·빈 값 전부 403).
  - MCP 파라미터는 평평하지 않고 **`request` 객체**로 감싼다: `bundle_upload{request:{deploymentId,memo}}`, `bundle_submit_review{request:{deploymentId}}`. `bundle_set_release_note`의 필드명은 `releaseNote`가 아니라 **`releaseNotes`**(복수형)다.
  - `bundle_submit_review`의 `featureList`는 선택 항목이고 **객체 배열**이다(`{linkUri,title}` 필수). 문자열을 넣으면 거부된다 — 안 쓸 거면 아예 빼는 게 맞다.
- ⚠️ `.env`는 gitignore라 유실돼도 티가 안 남 — 재배포 전 `iaa_placement_group_list`로 콘솔 그룹과 대조할 것. 실제로 소라고동·구독지킴이 .env가 유실된 채 발견됨.

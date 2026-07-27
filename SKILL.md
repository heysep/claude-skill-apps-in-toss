---
name: apps-in-toss
description: 인앱토스(Apps in Toss) 미니앱 개발·제출·심사 대응. granite.config.ts 설정, ait build / ait deploy, 콘솔 등록(브랜드 아이콘·스크린샷·개인정보처리방침), 심사 반려 사유 대응, 한국어 UI 다해상도 QA, 급여·금액 계산 로직 검증에 사용. 트리거 - "인앱토스", "Apps in Toss", "토스 미니앱", "granite.config", "ait build", "ait deploy", "토스 콘솔", "심사 반려", "미니앱 제출".
---

# Apps in Toss 미니앱 — 실전 체크리스트

실제로 심사 반려를 맞고 고친 항목들이다. 새 미니앱을 만들거나 제출할 때 이 순서로 확인한다.

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
못한다.** 파일 내용이 콘솔 업로드본과 바이트 단위로 같아도 소용없다 — 실제로 md5까지
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

## 2. 한국어 UI 다해상도 QA

토스 사용자 기기 폭은 320\~430px에 걸쳐 있다. 320px에서 깨지는 게 기본값이라고 생각하고 시작한다.

실제로 걸렸던 것들:

- **`<input type="time">`의 크롬 시계 아이콘** — 좁은 화면에서 시간 값을 잘라먹는다.
  `::-webkit-calendar-picker-indicator { display: none }`로 숨기고 좌우 여백을 줄인다.
- **한글 레이블 중간 줄바꿈** — "야간수당"이 "야간/수당"으로 쪼개진다.
  `word-break: keep-all`을 한글 레이블에 기본으로 깐다. 배지가 붙는 행은 `flex-wrap`으로
  배지가 다음 줄로 내려가게 한다.
- **터치 타깃** — 44×44px 미만이 없는지 자동 검출.

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

## 5. 콘솔 등록 · 심사 제출

- **스크린샷** — 390×844 @2x. 앱 실제 화면을 자동 생성하는 스크립트(`scripts/store-shots.mjs`)를
  두면 UI 변경 때마다 다시 뽑기 쉽다.
- **키워드** — 콘솔에 넣은 **등록 순서가 유지**되므로 중요한 것부터.
- **개인정보처리방침** — 별도 입력란이 있다. `docs/PRIVACY.md` 내용을 그대로 넣는다.
  서버 전송이 없고 `localStorage`만 쓴다면 그 사실을 명시하는 게 심사에 유리하다.
- **네이티브 권한** — 안 쓰면 `permissions: []`로 비워 둔다. 불필요한 권한은 반려 사유가 된다.
- **광고** — 유닛 ID를 환경변수로만 주입하고 미주입 시 광고 UI가 **아예 렌더되지 않게**
  (zero footprint) 만들어 두면, ID 미발급 상태로도 제출할 수 있다.
- **심사 코멘트에 테스트 시나리오를 첨부한다.** 심사자가 그대로 따라 하면 전 기능을 볼 수 있는
  번호 매긴 순서. 야간 근무처럼 특수 입력이 필요한 기능은 이게 없으면 확인조차 안 된다.

### 금융·계산 도구라면 면책 고지가 필수다

결과 화면과 스토어 설명 양쪽에 **"모의 계산이며 법적 효력이 없다"**를 명시한다.
금융 상품 판매·중개가 아니라 참고용 계산 도구임이 드러나야 한다.

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

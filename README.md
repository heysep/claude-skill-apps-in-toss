# apps-in-toss — Claude Code Skill

인앱토스(Apps in Toss) 미니앱을 만들고 심사에 통과시키기까지 **실제로 막혔던 것들**을
정리한 [Claude Code](https://claude.com/claude-code) 스킬입니다.

문서를 요약한 게 아니라, 심사 반려를 세 번 맞고 원인을 찾아가며 얻은 내용입니다.

## 무엇이 들어 있나

| 항목 | 요약 |
| --- | --- |
| **`brand.icon`은 URL이다** | 로컬 파일 경로를 넣으면 아이콘 심사에서 반드시 반려된다. 콘솔 로고의 링크를 넣어야 한다 |
| **잘못된 값도 빌드·배포가 성공한다** | `brand.icon`은 타입 검증만 거치는 opaque string. 오류가 안 나서 반려 전까지 모른다 |
| **배포 전 `git status`** | 에셋을 교체하고 커밋 안 한 채 배포하면 콘솔 등록본과 어긋난다 |
| **`appName` ↔ `APP_NAME`** | 딥링크 해석에 쓰이며 두 파일에 흩어져 있다. 짧은 이름은 대부분 선점돼 있다 |
| **Code 4097** | 동일 번들이 이미 올라가 있으면 배포가 조용히 취소된다 |
| **광고 키는 유형별 발급** | 배너와 전면·리워드는 광고 그룹을 따로 만들어 각각 ID를 받는다. 돌려 쓸 수 없다 |
| **한국어 UI 다해상도 QA** | 320\~430px 스윕, 크롬 시계 아이콘, 한글 `word-break: keep-all` |
| **런타임 E2E 스모크** | `NaN`/`undefined` 노출 0건, 손상 `localStorage` 내성, 해시 라우트 전수 |
| **금액 계산의 부동소수점 경계** | 정수 최소단위로 합산 후 한 번만 환산. 주 15시간이 14.999…가 되면 주휴수당이 사라진다 |
| **콘솔 등록·심사 제출** | 스크린샷 규격, 키워드 순서, 개인정보처리방침, 금융 계산기 면책 고지 |

## 설치

```bash
git clone https://github.com/heysep/claude-skill-apps-in-toss.git \
  ~/.claude/skills/apps-in-toss
```

Windows(PowerShell):

```powershell
git clone https://github.com/heysep/claude-skill-apps-in-toss.git `
  "$env:USERPROFILE\.claude\skills\apps-in-toss"
```

유저 레벨(`~/.claude/skills/`)에 두면 모든 프로젝트에서 자동으로 잡힙니다.
특정 프로젝트에만 적용하려면 그 저장소의 `.claude/skills/` 아래에 두세요.

## 언제 발동하나

Claude Code가 `description`의 트리거를 보고 알아서 로드합니다 — "인앱토스",
"Apps in Toss", "토스 미니앱", "granite.config", "ait build", "ait deploy",
"토스 콘솔", "심사 반려", "미니앱 제출" 등.

직접 부를 수도 있습니다:

```
/apps-in-toss
```

## 검증된 환경

`@apps-in-toss/web-framework` 2.10.x 기준입니다. 플랫폼 규격(아이콘 크기, 콘솔 UI,
에러 코드)은 바뀔 수 있으니 숫자는 **콘솔과 [공식 문서](https://developers-apps-in-toss.toss.im/)에서
확인**하세요. 스킬 본문도 그렇게 쓰여 있습니다.

## 기여

같은 함정을 만나셨거나 규격이 바뀐 걸 발견하셨다면 이슈·PR 환영합니다.
**추측이 아니라 실제로 확인한 내용**만 담는 것이 이 스킬의 원칙입니다 —
근거(문서 링크, 에러 원문, 재현 방법)를 함께 남겨 주세요.

## 라이선스

[MIT](LICENSE)

---
name: store-screenshots
description: 앱스토어·플레이스토어 등록용 마케팅 스크린샷 자동 생성. 원본 앱 스크린샷에 기기 프레임(iPhone, iPad, Galaxy, Fold, Flip)을 씌우고 배경 그라데이션과 홍보 문구를 얹어 스토어 규격 PNG로 출력한다. "스토어 스크린샷 만들어줘", "앱스토어 이미지", "플레이스토어 스크린샷" 같은 요청에 사용.
argument-hint: [원본 스크린샷 폴더 경로]
---

# 스토어 스크린샷 생성기

원본 앱 스크린샷을 받아 → 기기 프레임 + 브랜드 배경 + 홍보 문구를 얹은 스토어 등록용 이미지를 만든다.

## 절대 규칙

1. **컨펌 전에는 전체 렌더링을 하지 않는다.** 문구 초안 표 + 시안 1장(첫 슬라이드)을 보여주고 사용자 OK를 받은 뒤에만 나머지를 렌더링한다.
2. 최종 PNG는 전부 픽셀 크기를 검증(`sips` 또는 ImageMagick `identify`)하고, 규격 표와 다르면 다시 렌더링한다.
3. 기기 프레임은 외부에서 다운로드하지 않고 HTML/CSS로 직접 그린다(라이선스·네트워크 문제 없음). 실사 프레임을 원하면 아래 "실사 프레임" 참고.
4. 중간 작업 파일(html 등)은 스크래치패드에 두고, 출력 폴더에는 최종 PNG만 남긴다.
5. **사용자에게 손을 쓰게 하지 않는다.** 화면 전환·탭 같은 기기 조작은 Claude가 adb/idb/AppleScript로 직접 한다(아래 "자동 캡처"). 사용자의 개입은 AskUserQuestion 선택지 고르기나 짧은 타이핑까지만. "화면 띄우고 메시지 보내주세요" 같은 진행은 금지 — 자동화 수단이 전부 실패했을 때만 선택지로 수동 진행 여부를 묻는다.

## 워크플로

### 1. 원본 수집

- 인자(`$ARGUMENTS`)로 폴더를 받았으면 거기서, 아니면 현재 디렉토리에서 png/jpg를 스캔한다:
  `find . -maxdepth 2 \( -iname '*.png' -o -iname '*.jpg' \) -not -path '*/store-screenshots/*'`
- 파일이 없으면 **Claude가 직접 기기를 조작해서 캡처한다**(아래 자동 캡처). 사용자에게 화면 전환을 부탁하지 않는다.
- **각 원본을 Read로 열어 무슨 화면인지 파악한다.** 문구는 실제 화면 내용에 근거해야 한다. 직접 캡처한 경우에도 캡처 직후 Read로 열어 의도한 화면이 맞는지 확인하고, 아니면 다시 탐색해서 재캡처한다.
- 앱 이름·아이콘을 찾을 수 있으면 참고한다(Info.plist, strings.xml, ic_launcher, 앱 아이콘 파일 등).
- 캡처할 화면 목록(홈, 핵심 기능 화면 등)은 Claude가 앱 구조를 보고 스스로 정하되, 애매하면 2단계 질문에 선택지로 끼워서 같이 묻는다.

#### 자동 캡처 — Android (adb, 완전 자동)

1. `adb devices`로 기기/에뮬레이터 확인. 앱 실행: `adb shell monkey -p <package> 1` 또는 `adb shell am start -n <package>/<activity>`.
2. 현재 화면의 UI 구조를 덤프해서 탭·버튼 좌표를 얻는다:
   `adb exec-out uiautomator dump /dev/tty` → XML의 `bounds="[x1,y1][x2,y2]"`에서 중심 좌표 계산.
3. 이동 → 안정화 → 캡처를 한 번에 (sleep은 기기 쪽에서 실행):
   `adb shell "input tap X Y; sleep 1" && adb exec-out screencap -p > 02-mypet.png`
4. 화면마다 2~3을 반복. 스크롤이 필요하면 `adb shell input swipe`, 텍스트 입력은 `adb shell input text`.

#### 자동 캡처 — iOS 시뮬레이터

캡처는 `xcrun simctl io booted screenshot 01.png`. 탭은 `simctl`로 불가능하므로 도구 우선순위대로:

1. **idb**가 있으면(`command -v idb`): `idb ui describe-all`로 요소 좌표를 얻고 `idb ui tap X Y`.
2. **Maestro**가 있으면(`command -v maestro`): `tapOn`/`takeScreenshot`으로 flow yaml을 만들어 `maestro test`.
3. 둘 다 없으면 AppleScript로 Simulator 창을 직접 클릭(손쉬운 사용 권한 필요):
   System Events로 Simulator 창의 position/size를 얻고, 기기 해상도 → 창 좌표로 비례 변환해 `click at {x, y}`, 사이사이 `delay 1`.
4. 전부 실패했을 때만 AskUserQuestion으로 묻는다: "idb 설치 후 자동 진행 (Recommended)" / "내가 직접 화면을 넘길게". 수동을 골랐을 때만 사용자에게 화면 전환을 부탁한다.

### 2. 질문 플로우 (전부 선택지로)

이미 대화에서 답이 나온 항목은 다시 묻지 않는다. 사용자가 타이핑 없이 **선택만으로** 답할 수 있게 AskUserQuestion으로 묻는다.

**1차 — 플랫폼** (질문 1개):
- App Store (iOS) / Google Play (Android) / 둘 다

**2차 — 세부** (질문 4개를 한 번에, 기종 선택지는 1차 답에 맞춰 구성):

| 질문 | 선택지 |
|---|---|
| 기종 (multiSelect) | iOS: iPhone(Recommended), iPad · Android: Galaxy 폰(Recommended), Fold, Flip, 태블릿 · 둘 다: iPhone, iPad, Galaxy 폰, Fold·Flip |
| 스타일 | 기존 앱 디자인에 맞추기(Recommended) / 모던·미니멀 / 밝고 팝하게 / 다크·세련 |
| 장수 | 4장 / 5장(Recommended) / 6장 / 스토어 최대 |
| 어필 방향 | 알아서 뽑아줘(Recommended) / 핵심 기능 직접 지정 — 직접 지정이면 Other로 기능을 적게 한다 |

### 3. 배경 디자인 결정

스타일 답에 따라 배경·장식·톤을 정한다.

- **기존 앱 디자인에 맞추기** (기본): 앱의 실제 디자인 시스템에서 뽑는다.
  - 색: Android는 `res/values/colors.xml`·`themes.xml`, iOS는 `Assets.xcassets`의 Color Set, Flutter는 `ThemeData`, 웹 기반이면 CSS 변수·tailwind config. 소스가 없으면 앱 아이콘·스크린샷에서 주조색과 포인트색을 추출한다.
  - 톤: 앱 UI가 라이트면 밝게, 다크면 어둡게. 앱의 코너 radius가 크면 배경 장식도 둥글게, 버튼이 각지면 장식도 각지게.
- **모던·미니멀**: 밝은 단색 또는 아주 옅은 그라데이션, 여백 크게, 장식 없음. 헤드라인 볼드 + 서브카피 옅게.
- **밝고 팝하게**: 채도 높은 그라데이션(주조색 ± 인접색), 큰 블롭·도형 장식 1~2개, 기울임 적극 사용.
- **다크·세련**: `#0b0b0f` 계열 어두운 배경 + 주조색 글로우·그라데이션 포인트, 프레임 그림자 강하게.

공통: 텍스트는 상단 12~18%에 배경과 대비 확실한 색, 폰트 `-apple-system, "Apple SD Gothic Neo", "Malgun Gothic", sans-serif`. 슬라이드마다 같은 hue에서 명도만 살짝 바꾸면 세트 느낌이 난다. 기기는 하단 중앙 + bottom bleed(화면 아래로 살짝 잘리게) + `box-shadow`, 홀수 장 -2도 / 짝수 장 +2도 기울이면 리듬감(기울임 여부는 컨펌 때 언급).

### 4. 시안 + 문구 초안 ← 컨펌 게이트

- 슬라이드 구성 (2차 질문의 장수 답 반영, App Store 최대 10장·Play 폼팩터당 2~8장):
  - 1장: 훅 — 앱의 핵심 가치 한 줄. 목록에서 1~2장만 보이므로 가장 중요하다.
  - 중간: 자랑할 기능 하나씩 (어필 방향 반영).
  - 마지막: 신뢰·마무리(리뷰, 다운로드 수, CTA) — 근거가 있을 때만.
- 문구 규칙:
  - 헤드라인: 한글 6~14자, 혜택 중심. "할 일을 잊지 않게" > "알림 기능".
  - 서브카피: 0~1줄, 20자 이내.
  - "1위", "최고" 같은 표현은 근거가 있을 때만.
- **문구 표 + 시안 1장을 함께 제시한다**:
  - 표: | # | 원본 파일 | 헤드라인 | 서브카피 |
  - 시안: 1번 슬라이드만 실제 규격으로 렌더링해서 macOS면 `open <파일>`로 띄워 보여준다. 배경·프레임·문구를 실물로 확인시키기 위함.
- 수정 요청이 오면 반영해 다시 제시한다(시안도 다시 렌더링). **OK를 받으면** 전체 렌더링으로.

### 5. 렌더링

- 타깃 × 슬라이드마다 HTML 파일을 생성한다(아래 템플릿). body 크기 = 스토어 규격 픽셀.
- 원본 이미지는 상대경로 `<img src="./01.png">`로 참조한다(data URI 불필요 — 같은 폴더에 복사해 두면 된다).
- 렌더 명령 (기본: 헤드리스 크롬):

  ```bash
  # 크롬 실행 파일 찾기 (OS에 맞게)
  CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"  # macOS
  command -v google-chrome >/dev/null 2>&1 && CHROME=google-chrome       # Linux
  command -v chromium >/dev/null 2>&1 && CHROME=chromium

  "$CHROME" --headless=new --screenshot="$OUT" --window-size=1320,2868 \
    --force-device-scale-factor=1 --hide-scrollbars --virtual-time-budget=2000 "$HTML"
  ```

- 크롬이 없으면 폴백: `npx -y playwright screenshot --viewport-size=1320,2868 "$HTML" "$OUT"` (첫 실행 시 `npx playwright install chromium` 필요할 수 있음).

### 6. 검증·보고

- 픽셀 크기 전수 확인 후 규격 표와 대조: macOS는 `sips -g pixelWidth -g pixelHeight <파일>`, 그 외 OS는 `identify -format '%wx%h\n' <파일>`(ImageMagick).
- 슬라이드 최소 1장을 Read로 직접 열어 확인: 글자 잘림, 프레임 비율 왜곡, 스크린샷 상하단 잘림.
- 출력 구조를 보여주고, 스토어 어느 슬롯에 올리면 되는지 안내하며 마무리한다.

출력 구조:

```
store-screenshots/
├── appstore-iphone-69/01.png …   # 1320×2868
├── appstore-iphone-65/           # 1284×2778 — iPhone은 항상 두 세트
├── appstore-ipad-13/
├── play-phone/
└── play-foldable/
```

## 스토어 규격 (2025년 기준)

| 타깃 | 크기(px, 세로) | 장수 | 비고 |
|---|---|---|---|
| App Store iPhone 6.9" | 1320×2868 | 최대 10 | 1290×2796도 허용 |
| App Store iPhone 6.5" | 1284×2778 | 최대 10 | 1242×2688도 허용 |
| App Store iPad 13" | 2064×2752 | 최대 10 | iPad 지원 앱이면 필수. 2048×2732 허용 |
| Play 폰 | 1080×1920 | 2~8 | 9:16. 피처링 조건: 최소 1080px, 4장 이상 |
| Play 태블릿 7"/10" | 1080×1920 또는 1920×1080 | 최대 8 | 태블릿 지원 시 |
| Play 폴더블 | 폰 슬롯에 포함 | — | 별도 슬롯 없음. Fold 펼침 화면은 태블릿 슬롯에도 활용 가능 |

- **iPhone은 6.9"와 6.5" 두 세트를 모두 렌더링한다.** App Store Connect는 앱의 기기 지원 범위에 따라 6.5" 규격(1284×2778 / 1242×2688)을 요구하기도 해서, 6.9" 한 세트만 만들면 업로드가 거부될 수 있다. 같은 HTML에서 body 크기만 바꿔 다시 캡처하면 되므로 비용이 거의 없다.

규격은 바뀔 수 있다. 확신이 없으면 렌더링 전에 공식 문서를 확인한다:
- App Store: developer.apple.com/help/app-store-connect/reference/screenshot-specifications
- Play: support.google.com/googleplay/android-developer/answer/9866151

## 기기 프레임 스펙 (CSS로 그리기)

| 기기 | 화면비(세로) | 코너 radius | 베젤 | 상단 요소 |
|---|---|---|---|---|
| iPhone | 1320:2868 | 프레임 폭의 ~14% | 균일 ~2% | Dynamic Island 알약 (폭 ~26%, 상단에서 ~1.8%) |
| iPad | 3:4 | ~4% | 균일 ~3.5% | 없음 |
| Galaxy S | 9:19.5 | ~7% | 균일 ~1.5% | 중앙 펀치홀 (지름 ~3%) |
| Galaxy Fold 펼침 | 1856:2160 (거의 정사각) | ~5% | ~2% | 우측 상단 펀치홀 + 중앙 세로 힌지 라인(아주 옅게) |
| Galaxy Flip | 9:22 | ~8% | ~1.5% | 중앙 펀치홀 |

- 프레임 = `div` (배경 `#101014`, `border-radius`, `padding` = 베젤, `inset box-shadow`로 금속 테두리 느낌 한 줄).
- 화면 = `img { object-fit: cover; object-position: top; border-radius: 바깥radius − 베젤 }`.
- 원본 비율이 프레임과 다르면 cover로 채우되 **상단 고정**(상태바 쪽이 보이는 게 자연스럽다).
- Galaxy 펀치홀: 알약 대신 `.hole { width:34px; height:34px; border-radius:50%; background:#000; top:40px; left:50%; }`.
- Fold 힌지: 화면 위에 세로 1줄 `linear-gradient` 오버레이(투명→살짝 어둡게→투명).

### 실사 프레임을 원할 때

사용자가 "진짜 기기 사진 프레임"을 원하면 직접 다운로드하지 말고 안내한다:
- Apple 공식 베젤: developer.apple.com/design/resources (Product Bezels, 마케팅 가이드라인 확인 필요)
- Samsung: Samsung Developers / Samsung Newsroom의 공식 목업 검색
사용자가 파일을 받아 경로를 주면, CSS 프레임 대신 그 이미지를 겹쳐 쓴다.

## 슬라이드 HTML 템플릿 (iPhone 6.9" 예시)

다른 기기는 body 크기, `.device`의 `aspect-ratio`·`border-radius`·상단 요소만 위 표대로 바꾼다.

```html
<!doctype html>
<html><head><meta charset="utf-8">
<style>
  :root { --c1:#4F46E5; --c2:#312E81; }        /* 앱 주조색으로 교체 */
  * { margin:0; box-sizing:border-box; }
  body {
    width:1320px; height:2868px; overflow:hidden;
    background:linear-gradient(165deg,var(--c1),var(--c2));
    font-family:-apple-system,"Apple SD Gothic Neo","Malgun Gothic",sans-serif;
    display:flex; flex-direction:column; align-items:center;
  }
  .copy { text-align:center; margin-top:220px; color:#fff; padding:0 90px; }
  .copy h1 { font-size:118px; font-weight:800; letter-spacing:-2px; line-height:1.15; }
  .copy p  { font-size:52px; font-weight:500; opacity:.85; margin-top:36px; }
  .device {
    margin-top:170px; width:1010px; aspect-ratio:1320/2868;
    background:#101014; border-radius:145px; padding:24px;
    box-shadow:0 80px 160px rgba(0,0,0,.45), inset 0 0 0 5px #3c3c40;
    position:relative; transform:rotate(-2deg);
  }
  .device img { width:100%; height:100%; object-fit:cover; object-position:top; border-radius:121px; }
  .island { position:absolute; top:52px; left:50%; transform:translateX(-50%);
            width:270px; height:78px; border-radius:44px; background:#000; }
</style></head>
<body>
  <div class="copy"><h1>헤드라인</h1><p>서브카피</p></div>
  <div class="device"><img src="./01-home.png"><div class="island"></div></div>
</body></html>
```

## 문구 톤 가이드

- 스토어 목록에서 실제로 보이는 건 1~2장 — 첫 장에 앱의 존재 이유를 담는다.
- 혜택 > 기능: "가계부가 자동으로 써진다" > "SMS 파싱 기능".
- 사용자가 "알아서"를 골랐으면: 스크린샷을 보고 차별점이 큰 순서로 4개를 뽑아, 왜 그걸 골랐는지 근거와 함께 제시한다.
- 로케일: 기본 한국어. 영어 스토어에도 올린다면 같은 레이아웃에 영어 문구 세트를 추가로 생성한다.

---
name: store-screenshots
description: 앱스토어·플레이스토어 등록용 마케팅 스크린샷 자동 생성. 원본 앱 스크린샷에 기기 프레임(iPhone, iPad, Galaxy, Fold, Flip)을 씌우고 배경 그라데이션과 홍보 문구를 얹어 스토어 규격 PNG로 출력한다. "스토어 스크린샷 만들어줘", "앱스토어 이미지", "플레이스토어 스크린샷" 같은 요청에 사용.
argument-hint: [원본 스크린샷 폴더 경로]
---

# 스토어 스크린샷 생성기

원본 앱 스크린샷을 받아 → 기기 프레임 + 브랜드 배경 + 홍보 문구를 얹은 스토어 등록용 이미지를 만든다.

## 절대 규칙

1. **문구 컨펌 전에는 렌더링하지 않는다.** 헤드라인·서브카피 초안을 표로 보여주고 사용자 OK를 받은 뒤에만 렌더링 단계로 진행한다.
2. 최종 PNG는 전부 픽셀 크기를 검증(`sips` 또는 ImageMagick `identify`)하고, 규격 표와 다르면 다시 렌더링한다.
3. 기기 프레임은 외부에서 다운로드하지 않고 HTML/CSS로 직접 그린다(라이선스·네트워크 문제 없음). 실사 프레임을 원하면 아래 "실사 프레임" 참고.
4. 중간 작업 파일(html 등)은 스크래치패드에 두고, 출력 폴더에는 최종 PNG만 남긴다.

## 워크플로

### 1. 원본 수집

- 인자(`$ARGUMENTS`)로 폴더를 받았으면 거기서, 아니면 현재 디렉토리에서 png/jpg를 스캔한다:
  `find . -maxdepth 2 \( -iname '*.png' -o -iname '*.jpg' \) -not -path '*/store-screenshots/*'`
- 원본이 하나도 없으면 실행 중인 기기에서 직접 캡처를 제안한다:
  - iOS 시뮬레이터: `xcrun simctl io booted screenshot 01.png`
  - Android 기기/에뮬레이터: `adb exec-out screencap -p > 01.png`
- **각 원본을 Read로 열어 무슨 화면인지 파악한다.** 문구는 실제 화면 내용에 근거해야 한다.
- 앱 이름·아이콘을 찾을 수 있으면 참고한다(Info.plist, strings.xml, ic_launcher, 앱 아이콘 파일 등).

### 2. 타깃·어필 방향 질문

AskUserQuestion 한 번에 두 질문을 묶어서 묻는다. 이미 대화에서 답이 나온 항목은 다시 묻지 않는다.

- **타깃 기기** (multiSelect): App Store iPhone / App Store iPad / Play 폰(Galaxy) / Play 폴더블(Fold·Flip)
- **어필 방향**: "알아서 뽑아줘 (Recommended)" / "핵심 기능 직접 지정" — 직접 지정을 고르면 어떤 기능을 자랑할지 이어서 듣는다(Other 입력 활용).

### 3. 슬라이드 기획 + 문구 초안 ← 컨펌 게이트

- 슬라이드 수: **4~6장 권장** (App Store 최대 10장, Play 폼팩터당 2~8장).
- 권장 구성:
  - 1장: 훅 — 앱의 핵심 가치 한 줄. 목록에서 1~2장만 보이므로 가장 중요하다.
  - 2~5장: 자랑할 기능 하나씩 (어필 방향 반영).
  - 마지막: 신뢰·마무리(리뷰, 다운로드 수, CTA) — 근거가 있을 때만.
- 문구 규칙:
  - 헤드라인: 한글 6~14자, 혜택 중심. "할 일을 잊지 않게" > "알림 기능".
  - 서브카피: 0~1줄, 20자 이내.
  - "1위", "최고" 같은 표현은 근거가 있을 때만.
- 표로 제시하고, 배경 색·기울임 등 디자인 계획도 한 줄로 언급한다:

  | # | 원본 파일 | 헤드라인 | 서브카피 |
  |---|---|---|---|

- 수정 요청이 오면 반영해서 다시 제시한다. **OK를 받으면** 다음 단계로.

### 4. 디자인 결정

- **주조색**: 앱 아이콘 또는 첫 스크린샷에서 추출 → `--c1`(주조색), `--c2`(같은 계열의 어둡거나 밝은 변형). 슬라이드마다 같은 hue에서 명도만 살짝 바꾸면 세트 느낌이 난다.
- **텍스트**: 상단 12~18% 위치, 배경과 대비 확실한 색(보통 흰색). 폰트는 `-apple-system, "Apple SD Gothic Neo", "Malgun Gothic", sans-serif`.
- **기기 배치**: 하단 중앙, 화면 아래로 살짝 잘리게(bottom bleed) 두면 시원해 보인다. `box-shadow`로 띄우고, 홀수 장 -2도 / 짝수 장 +2도 기울이면 리듬감이 생긴다(기울임 여부는 컨펌 때 함께 언급).

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
├── appstore-iphone-69/01.png …
├── appstore-ipad-13/
├── play-phone/
└── play-foldable/
```

## 스토어 규격 (2025년 기준)

| 타깃 | 크기(px, 세로) | 장수 | 비고 |
|---|---|---|---|
| App Store iPhone 6.9" | 1320×2868 | 최대 10 | 필수 1종. 1290×2796도 허용 |
| App Store iPad 13" | 2064×2752 | 최대 10 | iPad 지원 앱이면 필수. 2048×2732 허용 |
| Play 폰 | 1080×1920 | 2~8 | 9:16. 피처링 조건: 최소 1080px, 4장 이상 |
| Play 태블릿 7"/10" | 1080×1920 또는 1920×1080 | 최대 8 | 태블릿 지원 시 |
| Play 폴더블 | 폰 슬롯에 포함 | — | 별도 슬롯 없음. Fold 펼침 화면은 태블릿 슬롯에도 활용 가능 |

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

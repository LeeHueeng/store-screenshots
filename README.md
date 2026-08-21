# store-screenshots

> A Claude Code skill that turns raw app screenshots into store-ready marketing images — device frame, branded background, and marketing copy — for the App Store and Google Play.

앱스토어·플레이스토어에 올릴 마케팅 스크린샷, 매번 피그마 열어서 만들기 귀찮으시죠?
이 스킬은 **원본 앱 스크린샷만 주면** Claude Code가 알아서 만들어줍니다:

- 📱 기기 프레임을 씌우고 — iPhone, iPad, Galaxy, **Fold**, **Flip**
- 🎨 앱 주조색을 뽑아 배경 그라데이션을 그리고
- ✍️ 화면 내용을 읽고 홍보 문구(헤드라인·서브카피)를 제안하고 — **컨펌 받은 뒤에만** 진행
- 📐 스토어 규격에 딱 맞는 PNG로 렌더링 (App Store 1320×2868, Play 1080×1920 등)

## 설치

```bash
git clone https://github.com/<your-id>/store-screenshots.git
cp -r store-screenshots/store-screenshots ~/.claude/skills/
```

- 모든 프로젝트에서 쓰려면: `~/.claude/skills/` (위 명령)
- 특정 프로젝트에서만 쓰려면: 그 프로젝트의 `.claude/skills/`에 복사

## 사용법

Claude Code에서:

```
/store-screenshots ./screenshots
```

또는 그냥 자연어로 — "앱스토어 스크린샷 만들어줘".

### 진행 흐름

1. **원본 수집** — 폴더에서 스크린샷을 찾고, 없으면 Claude가 **직접 앱을 조작해서** 캡처 (Android는 adb tap/screencap, iOS 시뮬레이터는 idb·Maestro·AppleScript). 사용자는 화면을 넘길 필요 없음 — 선택지 고르기와 문구 컨펌만 하면 됨
2. **질문** — 타깃 기기(iPhone / iPad / Galaxy / Fold·Flip)와 어필 방향(직접 지정 vs 알아서)
3. **문구 컨펌** — 슬라이드별 헤드라인·서브카피 초안을 표로 제시, OK 받기 전엔 렌더링 안 함
4. **렌더링** — HTML/CSS로 프레임·배경·문구를 조합해 헤드리스 크롬으로 캡처
5. **검증** — 모든 PNG의 픽셀 크기를 스토어 규격과 대조

### 출력 예시

```
store-screenshots/
├── appstore-iphone-69/01.png …   # 1320×2868
├── appstore-ipad-13/             # 2064×2752
├── play-phone/                   # 1080×1920
└── play-foldable/
```

## 요구사항

- [Claude Code](https://claude.com/claude-code)
- Chrome 또는 Chromium (헤드리스 렌더링용) — 없으면 Playwright로 자동 폴백
- 이미지 크기 검증: macOS는 내장 `sips`, 그 외 OS는 ImageMagick 권장

## 특징

- 기기 프레임을 외부에서 다운로드하지 않고 CSS로 직접 그립니다 — 라이선스·네트워크 걱정 없음. 실사 베젤을 원하면 Apple Design Resources 등 공식 소스를 안내합니다.
- 문구는 스크린샷을 실제로 읽고(어떤 화면인지 파악한 뒤) 혜택 중심으로 작성합니다.
- 한국어 기본, 영어 스토어용 문구 세트도 함께 생성 가능합니다.

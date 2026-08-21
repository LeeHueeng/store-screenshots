<div align="center">

<img src="promo/skill-promo-v2.png" alt="store-screenshots — 앱 스크린샷만 주면 프레임·배경·문구까지 얹어 스토어 등록 이미지를 만들어주는 에이전트 스킬 (Claude Code · Codex)" width="100%" />

# store-screenshots

**원본 앱 스크린샷만 주면, 스토어 등록용 마케팅 이미지가 나옵니다.**<br>
기기 프레임 · 브랜드 배경 · 홍보 문구까지 — App Store & Google Play 규격 그대로.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Agent%20Skill-D97757?logo=claude&logoColor=white)](https://claude.com/claude-code)
[![Codex](https://img.shields.io/badge/Codex-Agent%20Skill-000000?logo=openai&logoColor=white)](https://openai.com/codex)
[![Stores](https://img.shields.io/badge/App%20Store%20%26%20Google%20Play-%EA%B7%9C%EA%B2%A9%20%EC%A7%80%EC%9B%90-4285F4)](#-스토어-규격)

[English](README.md) · **한국어**

</div>

---

앱스토어·플레이스토어에 올릴 마케팅 스크린샷, 매번 피그마 열어서 만들기 귀찮으시죠?
이 스킬은 **원본 앱 스크린샷만 주면** 에이전트가 알아서 만들어줍니다. **Claude Code와 OpenAI Codex 둘 다 지원**해요 — 같은 SKILL.md 형식(Agent Skills)을 그대로 씁니다.

## ✨ 무엇을 해주나요

- 📱 **기기 프레임** — iPhone, iPad, Galaxy, **Fold**, **Flip** 프레임을 CSS로 직접 그려서 씌우고 (목업 다운로드·라이선스 걱정 없음)
- 🎨 **브랜드 배경** — 앱의 실제 디자인(colors.xml, xcassets, 아이콘)에서 색을 뽑아 배경을 그리고 (모던·미니멀 / 팝 / 다크 프리셋도 선택 가능)
- ✍️ **홍보 문구** — 화면 내용을 읽고 혜택 중심 헤드라인·서브카피를 제안하고 (**컨펌 받은 뒤에만** 진행)
- 📐 **스토어 규격 렌더링** — App Store 1320×2868 / 1284×2778, iPad 2064×2752, Play 1080×1920 등 규격에 딱 맞는 PNG로 출력하고 픽셀 검증까지
- 🤖 **자동 캡처** — 스크린샷이 없으면 에이전트가 직접 앱을 조작해서 캡처 (Android adb, iOS 시뮬레이터 딥링크·idb·Maestro), 상태바는 9:41·배터리 100%로 통일

## 🖼️ 결과물 예시

<div align="center">
<img src="promo/sample-output.png" alt="생성 결과 예시 — 앱 디자인에 맞춘 배경과 홍보 문구가 얹힌 App Store용 마케팅 스크린샷" width="300" />

*App Store iPhone 6.5″ 규격(1242×2688), "앱 디자인에 맞추기" 스타일로 생성한 예시*

</div>

## 🚀 설치

```bash
git clone https://github.com/LeeHueeng/store-screenshots.git

# Claude Code
cp -r store-screenshots/store-screenshots ~/.claude/skills/

# Codex
mkdir -p ~/.agents/skills
cp -r store-screenshots/store-screenshots ~/.agents/skills/
```

| 에이전트 | 모든 프로젝트에서 사용 | 특정 프로젝트에서만 사용 |
|----------|------------------------|--------------------------|
| Claude Code | `~/.claude/skills/` | 그 프로젝트의 `.claude/skills/` |
| Codex | `~/.agents/skills/` | 저장소 루트의 `.agents/skills/` |

> Codex 구버전을 쓴다면 스킬 경로가 `~/.codex/skills/`일 수 있어요.

## 💬 사용법

Claude Code에서:

```
/store-screenshots ./screenshots
```

Codex에서:

```
$store-screenshots ./screenshots
```

(또는 `/skills`로 목록에서 선택)

어느 쪽이든 그냥 자연어로도 됩니다 — **"앱스토어 스크린샷 만들어줘"**.

### 진행 흐름

1. **원본 수집** — 폴더에서 스크린샷을 찾고, 없으면 에이전트가 **직접 앱을 조작해서** 캡처합니다.
   - Android: `adb` tap/screencap · iOS 시뮬레이터: 딥링크(`simctl openurl`) 우선 → idb · Maestro · AppleScript
   - 상태바는 9:41 · 배터리 100%로 통일
   - 사용자는 화면을 넘길 필요 없음 — 선택지 고르기와 문구 컨펌만 하면 됩니다
   - 재실행이면 "기존 스크린샷 재활용 / 처음부터 새로"를 먼저 물어봅니다
2. **질문 (전부 선택지)** — 플랫폼(App Store / Google Play) → 기종(iPhone, iPad, Galaxy, Fold·Flip) → 스타일(앱 디자인에 맞추기 / 모던·미니멀 / 팝 / 다크) → 장수 → 어필 방향
3. **시안 컨펌** — 문구 초안 표 + 첫 슬라이드 시안 1장을 실제 규격으로 렌더링해 보여주고, OK 받은 뒤에만 전체 렌더링
4. **렌더링** — HTML/CSS로 프레임·배경·문구를 조합해 헤드리스 크롬으로 캡처
5. **검증** — 모든 PNG의 픽셀 크기를 스토어 규격과 대조

## 📦 출력 예시

```
store-screenshots/
├── appstore-iphone-69/01.png …   # 1320×2868
├── appstore-iphone-65/           # 1242×2688 — iPhone은 항상 두 세트
├── appstore-ipad-13/             # 2064×2752
├── play-phone/                   # 1080×1920
└── play-foldable/
```

## 📏 스토어 규격

| 타깃 | 크기(px, 세로) | 장수 | 비고 |
|------|---------------:|------|------|
| App Store iPhone 6.9″ | 1320×2868 | 최대 10 | 1290×2796도 허용 |
| App Store iPhone 6.5″ | 1284×2778 | 최대 10 | 1242×2688도 허용 |
| App Store iPad 13″ | 2064×2752 | 최대 10 | iPad 지원 앱이면 필수 |
| Play 폰 | 1080×1920 | 2~8 | 9:16 · 피처링 조건: 최소 1080px, 4장 이상 |
| Play 태블릿 | 1080×1920 / 1920×1080 | 최대 8 | 태블릿 지원 시 |
| Play 폴더블 | 폰 슬롯에 포함 | — | Fold 펼침 화면은 태블릿 슬롯에도 활용 가능 |

## ✅ 요구사항

| 필요한 것 | 비고 |
|-----------|------|
| [Claude Code](https://claude.com/claude-code) 또는 [Codex](https://openai.com/codex) | 스킬 실행 환경 — 둘 중 하나면 됩니다 |
| Chrome 또는 Chromium | 헤드리스 렌더링용 — 없으면 Playwright로 자동 폴백 |
| `sips`(macOS 내장) 또는 ImageMagick | 이미지 크기 검증용 |

## 🧩 웹 툴 대신 이걸 쓰는 이유

- **앱을 실제로 읽습니다.** 색은 진짜 디자인 시스템에서 뽑고, 문구는 각 화면에 실제로 뭐가 있는지 보고 씁니다 — 템플릿이 아니라요.
- **캡처까지 대신 합니다.** 대부분의 툴은 이미 있는 이미지에서 시작하지만, 이 스킬은 앱을 직접 실행·탐색해서 원본부터 만들어냅니다.
- **아무것도 밖으로 안 나갑니다.** 업로드·계정·워터마크 없음 — 로컬에서 HTML/CSS로 렌더링합니다.
- **한국어 기본, 영어 지원.** 기본은 한국어 문구, 영어 스토어용 문구 세트도 같은 레이아웃으로 함께 생성할 수 있습니다.

## 📄 라이선스

[MIT](LICENSE)

---

<div align="center">

피그마 한 번 덜 열게 됐다면, ⭐ 하나가 다른 개발자들에게도 이 스킬을 찾게 해줍니다.

</div>

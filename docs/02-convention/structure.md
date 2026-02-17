# DOOGIE - Project Structure (프로젝트 구조 규칙)

> 파일 배치, 코드 영역 분리, 의존성 방향을 정의하는 문서

**버전**: 1.0
**최종 수정일**: 2026-02-18

---

## 1. 프로젝트 디렉토리 구조

```
doogie-diary/
│
├── index.html              # [핵심] 메인 애플리케이션 (CSS+HTML+JS)
├── gif.worker.js           # GIF 생성 Web Worker
├── card_dos.gif            # 크리스마스 카드 에셋
│
├── image/                  # 정적 이미지 에셋
│   ├── doogie.png
│   └── Doogie_diary2.png
│
├── vite.config.js          # Vite 개발 서버 설정
├── vercel.json             # Vercel 배포 설정
├── package.json            # npm 패키지 관리
├── package-lock.json       # npm 의존성 잠금
│
├── CLAUDE.md               # AI/팀원 자동 참조 지침
├── CONVENTIONS.md          # 코딩 컨벤션 (전체)
├── DOOGIE_SPEC.md          # 기획/기능 명세서
├── README.md               # 프로젝트 소개
├── BLOG_POST.md            # 개발기 블로그
│
├── docs/                   # bkit 기반 기획 문서
│   ├── 01-plan/            # Phase 1: 스키마/용어 정의
│   │   ├── glossary.md
│   │   ├── schema.md
│   │   └── domain-model.md
│   └── 02-convention/      # Phase 2: 코딩 컨벤션
│       ├── naming.md
│       └── structure.md
│
├── node_modules/           # npm 패키지 (git ignored)
└── .gitignore              # Git 제외 목록
```

---

## 2. index.html 내부 구조

Single HTML File 내부는 다음 순서로 구성됩니다.

### 2.1 전체 구조

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="...">
    <title>Doogie - Personal Diary</title>

    <!-- ===== SECTION 1: STYLES ===== -->
    <style>
        /* 1-1. 폰트 임포트 */
        /* 1-2. 리셋 & 기본 */
        /* 1-3. 타이틀 화면 */
        /* 1-4. 에디터 화면 */
        /* 1-5. 이미지 처리 */
        /* 1-6. 달력 뷰 */
        /* 1-7. 크레딧 화면 */
        /* 1-8. 인증 화면 */
        /* 1-9. 모달 & 다이얼로그 */
        /* 1-10. 크리스마스 테마 */
        /* 1-11. 애니메이션 (@keyframes) */
        /* 1-12. 미디어 쿼리 (태블릿 → 모바일 → 터치) */
    </style>
</head>

<body>
    <!-- ===== SECTION 2: HTML MARKUP ===== -->
    <!-- 2-1. 타이틀 화면 -->
    <div id="titleScreen" class="screen"> ... </div>

    <!-- 2-2. 인증 화면 -->
    <div id="authScreen" style="display:none;"> ... </div>

    <!-- 2-3. 앱 컨테이너 (에디터/리스트/달력 등) -->
    <div id="appContainer" style="display:none;">
        <div id="contentView" class="mode-view"> ... </div>
        <div id="entryList" class="mode-list"> ... </div>
        <div id="calendarContainer" class="mode-calendar"> ... </div>
        <div id="helpScreen" class="mode-help"> ... </div>
        <div id="settingsScreen" class="mode-settings"> ... </div>
        <div id="creditsScreen" class="mode-credits"> ... </div>
        <div id="xmasCardScreen" class="mode-xmascard"> ... </div>
    </div>

    <!-- 2-4. 이미지 모달 -->
    <div id="imageModal" style="display:none;"> ... </div>

    <!-- ===== SECTION 3: EXTERNAL SCRIPTS ===== -->
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>

    <!-- ===== SECTION 4: APPLICATION SCRIPT ===== -->
    <script>
        /* 4-1. 상수 & 설정값 */
        /* 4-2. class BkendAPI - API 클라이언트 */
        /* 4-3. class DiaryDB - 데이터 관리 */
        /* 4-4. 인증 함수 (showAuthScreen, handleAuthSubmit...) */
        /* 4-5. 일기 CRUD 함수 (saveEntry, loadEntry...) */
        /* 4-6. 이미지 처리 함수 (pixelateImage, processImage...) */
        /* 4-7. 마크다운 함수 (initMermaid, togglePreview...) */
        /* 4-8. 달력 뷰 함수 (renderCalendar, navigateMonth...) */
        /* 4-9. 크레딧 화면 함수 (createNYCSkyline, createCars...) */
        /* 4-10. 크리스마스 함수 (snow, bgm, card...) */
        /* 4-11. 유틸리티 함수 (showLoading, showError...) */
        /* 4-12. 초기화: async function init() */
    </script>
</body>
</html>
```

### 2.2 CSS 영역 순서

| # | 영역 | 설명 |
|---|------|------|
| 1 | 폰트 & 리셋 | `@font-face`, `*`, `body` |
| 2 | 레이아웃 | `.screen`, `.menu-bar`, `.editor-container` |
| 3 | 컴포넌트 | 버튼, 입력, 드롭다운, 모달 |
| 4 | 화면별 스타일 | 타이틀, 에디터, 달력, 크레딧, 인증 |
| 5 | 테마 | 크리스마스 모드 |
| 6 | 애니메이션 | `@keyframes` 모음 |
| 7 | 반응형 | `@media` 쿼리 (큰→작은 순) |

### 2.3 JavaScript 영역 순서

| # | 영역 | 의존 방향 |
|---|------|----------|
| 1 | 상수/설정 (`USE_SERVER_MODE`) | 없음 (독립) |
| 2 | `class BkendAPI` (`projectId`, `environment` 내부 정의) | localStorage 참조 |
| 3 | `class DiaryDB` | BkendAPI 참조 |
| 4 | 기능 함수들 (재시도 로직 등 포함) | DiaryDB, BkendAPI 참조 |
| 5 | `init()` | 모든 것을 조합 |

---

## 3. 의존성 방향 (레이어)

```
┌─────────────────────────────────────┐
│         init() - 앱 초기화           │  ← 모든 것을 조합
├─────────────────────────────────────┤
│     기능 함수들 (Feature Functions)   │  ← 화면별 로직
│  showAuthScreen, saveEntry,         │
│  renderCalendar, createStars ...    │
├─────────────────────────────────────┤
│     class DiaryDB                    │  ← 데이터 관리 추상화
│  (BkendAPI를 내부적으로 호출)         │
├─────────────────────────────────────┤
│     class BkendAPI                   │  ← HTTP 요청 래퍼
│  (fetch, Authorization 헤더 관리)    │
├─────────────────────────────────────┤
│     상수 & 설정값                     │  ← 의존성 없음
│  USE_SERVER_MODE                    │
└─────────────────────────────────────┘
```

**핵심 규칙**: 위 레이어는 아래 레이어만 참조. 아래→위 참조 금지.

---

## 4. 화면(Screen) 관리 규칙

### 4.1 화면 전환

```javascript
// 모든 화면은 display: none/flex로 전환
// 한 번에 하나의 화면만 표시
document.querySelectorAll('.screen').forEach(s => s.style.display = 'none');
document.getElementById('target-screen').style.display = 'flex';
```

### 4.2 화면 ID 규칙

> HTML ID는 JavaScript DOM 요소 참조와 일관성을 위해 **camelCase**를 사용합니다.

| 화면 | ID | CSS 클래스 | display 값 |
|------|-----|-----------|-----------|
| 타이틀 | `titleScreen` | `.screen` | flex (기본) |
| 인증 | `authScreen` | - | none → flex |
| 앱 컨테이너 | `appContainer` | - | none → flex |
| 에디터 | `contentView` | `.mode-view` | none → flex |
| 일기 목록 | `entryList` | `.mode-list` | none → flex |
| 달력 | `calendarContainer` | `.mode-calendar` | none → flex |
| 크레딧 | `creditsScreen` | `.mode-credits` | none → flex |
| 도움말 | `helpScreen` | `.mode-help` | none → block |
| 설정 | `settingsScreen` | `.mode-settings` | none → flex |
| 크리스마스 카드 | `xmasCardScreen` | `.mode-xmascard` | none → flex |

### 4.3 모달/오버레이

```
z-index 계층:
  0~1     기본 콘텐츠, 별, 메뉴
  10      날짜 헤더
  100     메뉴바 (sticky)
  1000    드롭다운 메뉴
  1001    드롭다운 오버레이
  2000    크롭 모달
  3000    크롭 오버레이
  9998    Picture 드롭다운 (fixed)
  9999    List 드롭다운 (fixed)
  10000   확인 다이얼로그
  10001   다이얼로그 배경
```

---

## 5. 새 기능 추가 가이드

### 5.1 새 화면 추가 시

```
1. HTML: <div id="new-screen" class="screen" style="display:none;"> 추가
2. CSS: /* ===== New Screen ===== */ 섹션 추가
3. JS: show/hide 함수 추가, init()에서 이벤트 바인딩
4. 반응형: @media 쿼리에 모바일 대응 추가
```

### 5.2 새 API 연동 시

```
1. BkendAPI 클래스에 메서드 추가
2. DiaryDB에 래퍼 메서드 추가 (필요 시)
3. 기능 함수에서 try-catch로 호출
4. 로딩/에러 상태 처리
```

### 5.3 새 애니메이션 추가 시

```
1. @keyframes 섹션에 추가 (camelCase 네이밍)
2. DOS 녹색 팔레트 내에서 색상 선택
3. 성능: transform/opacity 위주 (layout 변경 최소화)
```

---

## 6. 문서 폴더 구조 규칙

### bkit 파이프라인 매핑

| 폴더 | Phase | 내용 |
|------|-------|------|
| `docs/01-plan/` | Phase 1 | 용어집, 스키마, 도메인 모델 |
| `docs/02-convention/` | Phase 2 | 네이밍, 구조 규칙 |
| `docs/03-mockup/` | Phase 3 | UI 목업 (향후) |
| `docs/04-api/` | Phase 4 | API 설계서 (향후) |
| `docs/05-design-system/` | Phase 5 | 디자인 시스템 (향후) |
| `docs/06-ui-integration/` | Phase 6 | UI-API 통합 (향후) |
| `docs/07-seo-security/` | Phase 7 | SEO/보안 점검 (향후) |
| `docs/08-review/` | Phase 8 | 코드 리뷰 (향후) |
| `docs/09-deployment/` | Phase 9 | 배포 설정 (향후) |

### 문서 파일 규칙

| 규칙 | 설명 |
|------|------|
| 파일명 | kebab-case.md |
| 헤더 | `# DOOGIE - 제목 (한글 제목)` |
| 메타정보 | 버전, 수정일, 프로젝트명 |
| 형식 | Markdown (GFM) |
| 다이어그램 | Mermaid 사용 |

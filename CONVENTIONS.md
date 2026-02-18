# DOOGIE - Coding Conventions (코딩 컨벤션)

> 기획자와 개발자(AI 포함)가 일관된 코드 스타일을 유지하기 위한 규칙

**버전**: 1.0
**최종 수정일**: 2026-02-18
**프로젝트**: DOOGIE Diary
**레벨**: Starter (Single HTML File 아키텍처)

---

## 1. 프로젝트 아키텍처

### 핵심 원칙: Single HTML File

DOOGIE는 **index.html 단일 파일**에 CSS + HTML + JavaScript가 모두 포함된 구조입니다.

```
index.html
├── <style>      ... CSS 전체
├── <body>       ... HTML 마크업 전체
└── <script>     ... JavaScript 전체
    ├── class BkendAPI { }    // API 클라이언트
    ├── class DiaryDB { }     // 데이터 관리 레이어
    ├── function init() { }   // 앱 초기화
    └── 기능별 함수들           // 화면별 로직
```

### 왜 Single File인가?

| 장점 | 설명 |
|------|------|
| 배포 간편 | 하나의 HTML 파일만 서빙하면 됨 |
| 의존성 최소 | 빌드 도구 없이도 동작 가능 |
| DOS 컨셉 부합 | 1990년대 단일 실행파일 느낌 |
| 빠른 프로토타이핑 | 파일 간 이동 없이 즉시 수정 |

### 예외 파일

| 파일 | 이유 |
|------|------|
| `gif.worker.js` | Web Worker는 별도 파일 필수 |
| `vite.config.js` | Vite 빌드 설정 |
| `vercel.json` | Vercel 배포 설정 (API 프록시) |
| `card_dos.gif` | 크리스마스 카드 에셋 |
| `image/` | 정적 이미지 에셋 |

---

## 2. 네이밍 규칙

### 2.1 JavaScript

| 대상 | 규칙 | 예시 |
|------|------|------|
| 변수 | camelCase | `dateKey`, `entryNumber`, `isLoggedIn` |
| 함수 | camelCase | `saveEntry()`, `loadEntries()`, `showAuthScreen()` |
| 클래스 | PascalCase | `BkendAPI`, `DiaryDB` |
| 상수 | UPPER_SNAKE_CASE | `USE_SERVER_MODE`, `MAX_ENTRIES_PER_DAY` |
| 이벤트 핸들러 | handle + 동작 | `handleAuthSubmit()`, `handleCheckboxToggle()` |
| 비동기 함수 | async + 동사 | `async loadLanguage()`, `async saveCharacterData()` |
| Boolean 변수 | is/has/can 접두사 | `isLoggedIn`, `hasImages`, `canWrite` |

### 2.2 CSS

| 대상 | 규칙 | 예시 |
|------|------|------|
| 클래스 | kebab-case | `.diary-editor`, `.image-modal`, `.auth-screen` |
| ID | kebab-case | `#title-screen`, `#diary-text` |
| 애니메이션 | camelCase | `@keyframes cursorBlink`, `@keyframes glowPulse` |
| CSS 변수 | --kebab-case | `--dos-green`, `--bg-color` |

### 2.3 파일/폴더

| 대상 | 규칙 | 예시 |
|------|------|------|
| HTML 파일 | kebab-case | `index.html`, `todo-list.html` |
| JS 파일 | kebab-case | `gif.worker.js`, `vite.config.js` |
| 이미지 | kebab-case 또는 PascalCase | `doogie.png`, `Doogie_diary2.png` |
| 문서 | kebab-case | `glossary.md`, `schema.md` |
| 문서 폴더 | 번호-이름 | `docs/01-plan/`, `docs/02-convention/` |

### 2.4 API/데이터

| 대상 | 규칙 | 예시 |
|------|------|------|
| API 엔드포인트 | 복수형 명사 | `/data/entries`, `/auth/signup/password` |
| DB 필드 | camelCase | `dateKey`, `entryNumber`, `createdBy` |
| 쿼리 파라미터 | camelCase | `andFilters`, `sortBy`, `sortDirection` |
| 헤더 | X-Capitalized | `X-Project-Id`, `X-Environment` |

---

## 3. 코드 스타일

### 3.1 JavaScript 스타일

```javascript
// 들여쓰기: 4 spaces (index.html 내부)
// 문자열: 작은따옴표 우선, 템플릿 리터럴 적극 사용
// 세미콜론: 사용

// [O] 좋은 예
const dateKey = '2025-12-18';
const message = `오늘은 ${count}/100개 작성했습니다`;
const entries = await api.getEntriesByDate(dateKey);

// [X] 나쁜 예
var dateKey = "2025-12-18"
const message = "오늘은 " + count + "/100개 작성했습니다"
```

### 3.2 함수 작성 패턴

```javascript
// 1. 일반 함수 선언 (hoisting 필요 시)
function showAuthScreen() {
    // ...
}

// 2. async 함수
async function loadEntries() {
    try {
        const data = await api.getAllEntries();
        // ...
    } catch (error) {
        console.error('Failed to load entries:', error);
        showError('서버에 연결할 수 없습니다');
    }
}

// 3. 이벤트 리스너 (인라인 화살표 함수)
button.addEventListener('click', () => {
    handleSave();
});

// 4. 콜백 (화살표 함수)
entries.filter(e => e.type === 'markdown');
entries.map(e => e.dateKey);
```

### 3.3 클래스 패턴

```javascript
// API 클라이언트 패턴
class BkendAPI {
    constructor() {
        this.baseUrl = '/api';
        this.token = localStorage.getItem('doogie_token');
    }

    async request(endpoint, options = {}) {
        // 공통 요청 로직
    }

    // 도메인별 메서드 그룹핑
    async signup(nickname, pin) { /* ... */ }
    async login(nickname, pin) { /* ... */ }
    async getAllEntries() { /* ... */ }
    async saveEntry(data) { /* ... */ }
}
```

### 3.4 에러 처리 패턴

```javascript
// API 에러: try-catch + 사용자 메시지
async function saveEntry() {
    try {
        showLoading('저장 중...');
        await api.saveEntry(data);
        showStatus('저장되었습니다!');
    } catch (error) {
        console.error('Save failed:', error);
        if (error.message.includes('401')) {
            handleLogout();  // 토큰 만료
        } else {
            showError('저장에 실패했습니다');
        }
    } finally {
        hideLoading();
    }
}
```

---

## 4. CSS 스타일

### 4.1 DOS 디자인 시스템 색상 (CSS 변수)

```css
/* :root에 정의된 CSS Custom Properties — 257곳에서 사용 중 */
--color-bg: #000000;              /* 배경 */
--color-bg-accent: #001100;       /* 어두운 배경 강조 */
--color-bg-field: #002200;        /* 입력 필드 배경 */
--color-bg-highlight: #003300;    /* 호버/활성 배경 */
--color-primary: #00ff00;         /* 기본 텍스트, 테두리, 글로우 */
--color-primary-bright: #55ff55;  /* 마크다운 모드 강조 */
--color-primary-dim: #00cc00;     /* 보조 밝은 텍스트 */
--color-primary-dark: #004400;    /* 어두운 강조, 구분선 */
--color-text: #00ff00;            /* 텍스트 (primary와 동일) */
--color-text-hover: #ffffff;      /* 호버 시 텍스트 */
--color-text-secondary: #00aa00;  /* 보조 텍스트, 보조 테두리 */
--color-text-shadow: #008800;     /* text-shadow 깊이, 흐린 텍스트 */
--color-text-dim: #006600;        /* 매우 흐린 텍스트 */
--color-text-muted: #005500;      /* 플레이스홀더 텍스트 */
--color-border: #00ff00;          /* 테두리 (primary와 동일) */

/* 비 녹색 팔레트 (하드코딩 유지) */
/* #ff4444 — 에러 상태 */
/* #ffaa00 — 경고/크리스마스 */
/* #ff8800 — 코드 블록 (마크다운 미리보기) */
```

> **규칙**: CSS에서 색상을 쓸 때는 반드시 `var(--color-*)` 변수를 사용합니다.
> `@keyframes`, `rgba()`, `gradient()` 내부는 예외입니다.

### 4.2 미디어 쿼리 브레이크포인트

```css
/* 데스크톱 (기본) */
/* 4:3 비율, 좌우 분할 레이아웃 */

/* 태블릿: 768px 이하 */
@media (max-width: 768px) {
    /* 메뉴 축소, 모드 토글 숨김 */
}

/* 모바일: 480px 이하 */
@media (max-width: 480px) {
    /* 세로 레이아웃, 로고 축소, 스와이프 갤러리 */
}

/* 터치 기기 */
@media (hover: none) and (pointer: coarse) {
    /* 최소 터치 영역 44px */
}
```

### 4.3 애니메이션 네이밍

```css
/* @keyframes: 기능 설명 camelCase */
@keyframes cursorBlink { }    /* 커서 깜빡임 */
@keyframes glowPulse { }     /* 글로우 펄스 */
@keyframes scanReveal { }    /* 스캔 효과 나타남 */
@keyframes snowfall { }      /* 눈 내리기 */
@keyframes driveRight { }    /* 자동차 오른쪽 이동 */
@keyframes twinkle { }       /* 별 깜빡임 */
@keyframes loadingBlink { }  /* 로딩 깜빡임 */
```

---

## 5. HTML 구조 컨벤션

### 5.1 화면(Screen) 구조

```html
<!-- 각 화면은 screen 클래스 + 고유 ID -->
<div id="title-screen" class="screen">
    <!-- 화면 내용 -->
</div>

<div id="diary-screen" class="screen" style="display:none;">
    <!-- 메뉴바 -->
    <div class="menu-bar"> ... </div>
    <!-- 에디터 영역 -->
    <div class="editor-container"> ... </div>
</div>

<div id="calendar-screen" class="screen" style="display:none;">
    <!-- 달력 뷰 -->
</div>
```

### 5.2 화면 전환 패턴

```javascript
// display: none/flex로 전환
function showScreen(screenId) {
    document.querySelectorAll('.screen').forEach(s => s.style.display = 'none');
    document.getElementById(screenId).style.display = 'flex';
}
```

### 5.3 모달 패턴

```html
<!-- 모달: position fixed, z-index 높은 값 -->
<div id="image-modal" style="display:none;">
    <div class="modal-content">
        <!-- 내용 -->
    </div>
</div>
```

---

## 6. 환경 변수 / 설정 컨벤션

### 6.1 빌드 설정

| 파일 | 용도 |
|------|------|
| `vite.config.js` | 개발 서버 (port 3000, API 프록시) |
| `vercel.json` | 프로덕션 API 프록시 (rewrites) + 보안 헤더 |

### 6.2 API 프록시

```
개발 환경:  localhost:3000/api/* → api-enduser.bkend.ai/*
프로덕션:   doogie.app/api/*    → api-enduser.bkend.ai/*
```

### 6.3 앱 내부 설정값

```javascript
// 코드 내 상수로 관리 (환경 변수 불필요)
const USE_SERVER_MODE = true;            // 서버 모드 (false=IndexedDB 전용)
const AUTO_SAVE_DELAY = 2000;            // 자동 저장 debounce (ms)
const NICKNAME_CHECK_DELAY = 500;        // 닉네임 중복 체크 debounce (ms)
const MAX_ENTRIES_PER_DAY = 100;         // 하루 최대 일기 수
const IMAGE_THUMBNAIL_SIZE = 600;        // 썸네일 최대 px
const IMAGE_FULL_SIZE = 1200;            // 원본 최대 px
const PIXEL_BLOCK_SIZE = 3;              // 픽셀화 블록 크기
// projectId, environment는 BkendAPI 클래스 내부에 정의
```

### 6.4 로컬 저장소 키

```javascript
// localStorage 키: doogie_ 접두사 (언더스코어 구분)
'doogie_access_token'    // JWT Access Token
'doogie_christmas_mode'  // 크리스마스 모드 (true|false)
'doogie_xmas_cards'      // 크리스마스 카드 JSON 배열

// 레거시 키 (하이픈 구분 — 호환성 유지)
'doogie-diary'           // IndexedDB 로컬 데이터
'doogie-characters'      // 캐릭터 데이터
'doogie-language'        // 언어 설정 (ko|en)
```

---

## 7. 커밋 메시지 컨벤션

### 형식

```
<type>: <subject>

<body (optional)>
```

### 타입

| 타입 | 설명 | 예시 |
|------|------|------|
| `Add` | 새 기능 추가 | `Add: debug logs for multiple file selection` |
| `Fix` | 버그 수정 | `Fix: allow adding multiple images at once` |
| `Update` | 기존 기능 개선 | `Update: mobile layout for image gallery` |
| `Refactor` | 코드 리팩토링 | `Refactor: extract image processing logic` |
| `Style` | CSS/UI 변경 | `Style: center single image on mobile` |
| `Docs` | 문서 수정 | `Docs: update DOOGIE_SPEC.md` |

---

## 8. 주석 컨벤션

```javascript
// 단일 행 주석: 설명이 필요한 로직만
// 닉네임을 이메일 형식으로 변환 (Bkend는 이메일 인증만 지원)
const email = `${nickname}@doogie.app`;

// 섹션 구분: ===== 로 그룹핑
// ===== Authentication =====
// ===== Diary CRUD =====
// ===== Image Processing =====

// TODO: 향후 구현 예정
// TODO: 오프라인 모드 지원

// 함수 설명: 복잡한 로직에만 블록 주석
/*
 * 이미지를 DOS 스타일 픽셀 아트로 변환
 * 1. Canvas에 원본 이미지 렌더링
 * 2. 3px 단위로 다운스케일
 * 3. 녹색 5단계 팔레트 매핑
 */
function pixelateImage(img, maxSize) { }
```

---

## 9. 외부 라이브러리 사용 규칙

| 규칙 | 설명 |
|------|------|
| CDN 우선 | npm 패키지보다 CDN `<script>` 태그 선호 |
| 최소 의존성 | Vanilla JS로 가능하면 라이브러리 추가 금지 |
| devDependencies만 npm | Vite만 npm 관리, 런타임 의존성은 CDN |
| 폰트 = Google Fonts | `@import url()` 또는 `@font-face` 사용 |

### 승인된 외부 라이브러리

| 라이브러리 | 로드 방식 | 용도 |
|-----------|----------|------|
| Marked.js | CDN | 마크다운 파싱 |
| Mermaid.js | CDN | 다이어그램 렌더링 |
| gif.js | CDN | GIF 생성 |
| html2canvas | CDN | PNG 캡처 |
| Vite | npm (dev) | 개발 서버/빌드 |

---

## 10. 컨벤션 체크리스트

### 코드 작성 전

- [ ] 기존 함수/패턴 확인 (중복 방지)
- [ ] 네이밍 규칙 준수 (camelCase, kebab-case 등)
- [ ] DOS 색상 팔레트 내에서 선택

### 코드 작성 후

- [ ] 에러 처리 포함 (try-catch + 사용자 메시지)
- [ ] 모바일 반응형 확인 (768px, 480px)
- [ ] 한국어/English 다국어 대응
- [ ] 자동 저장 로직과 충돌 없음
- [ ] console.error로 디버그 로그 유지

# DOOGIE - Naming Rules (네이밍 규칙)

> 코드 내 모든 이름의 작성 규칙을 정의하는 문서

**버전**: 1.0
**최종 수정일**: 2026-02-18

---

## 1. JavaScript 네이밍

### 1.1 변수

| 규칙 | 패턴 | 예시 (실제 코드) |
|------|------|-----------------|
| **일반 변수** | camelCase | `dateKey`, `entryNumber`, `currentEntry` |
| **Boolean** | is/has/can + 형용사 | `isLoggedIn`, `hasImages`, `canWrite` |
| **DOM 요소** | 요소명 + 설명 | `titleScreen`, `diaryText`, `imageModal` |
| **상수** | UPPER_SNAKE_CASE | `USE_SERVER_MODE`, `MAX_ENTRIES_PER_DAY` |
| **배열** | 복수형 | `entries`, `images`, `characters` |
| **객체** | 단수형 | `entry`, `image`, `config` |

### 1.2 함수

| 패턴 | 접두사 | 용도 | 예시 |
|------|--------|------|------|
| **동작** | 동사 + 명사 | 일반 동작 | `saveEntry()`, `loadEntries()`, `showScreen()` |
| **UI 표시** | show/hide | 화면 전환 | `showAuthScreen()`, `hideLoading()` |
| **UI 생성** | create | DOM 생성 | `createStars()`, `createNYCSkyline()`, `createCars()` |
| **UI 설정** | setup | 이벤트 바인딩 | `setupCharacterUpload()`, `setupEditableText()` |
| **데이터 로드** | load | 데이터 가져오기 | `loadLanguage()`, `loadCharacterData()` |
| **데이터 저장** | save | 데이터 저장 | `saveCharacterData()`, `saveEntry()` |
| **이벤트 처리** | handle | 이벤트 핸들러 | `handleAuthSubmit()`, `handleTabKey()` |
| **변환** | 동사 + To + 결과 | 데이터 변환 | `nicknameToEmail()`, `pinToPassword()` |
| **검증** | check/is | 유효성 검사 | `checkNickname()`, `isGifFile()` |
| **적용** | apply | 설정 적용 | `applyLanguage()`, `applyCroppedImage()` |
| **업데이트** | update | 상태 갱신 | `updateAuthUI()`, `updateTodayCount()` |
| **초기화** | init | 앱/모듈 시작 | `init()`, `initMermaid()` |

### 1.3 클래스

| 패턴 | 예시 | 설명 |
|------|------|------|
| PascalCase | `BkendAPI` | Bkend 서버 API 클라이언트 |
| PascalCase | `DiaryDB` | 일기 데이터 관리 레이어 |

### 1.4 이벤트 리스너

```javascript
// 패턴: element.addEventListener('이벤트', 핸들러)
button.addEventListener('click', () => handleSave());
document.addEventListener('keydown', (e) => handleKeyPress(e));
input.addEventListener('input', () => checkNickname(input.value));
```

---

## 2. CSS 네이밍

### 2.1 클래스/ID

| 대상 | 규칙 | 예시 |
|------|------|------|
| HTML ID | camelCase | `titleScreen`, `authScreen`, `contentView`, `calendarContainer` |
| CSS 클래스 | kebab-case | `.menu-bar`, `.editor-container`, `.image-section` |
| 영역 클래스 | `{기능}-{역할}` | `.menu-bar`, `.editor-container`, `.image-section` |
| 모드 클래스 | `mode-{이름}` | `.mode-view`, `.mode-list`, `.mode-calendar` |
| 컴포넌트 | `{컴포넌트}-{변형}` | `.btn-save`, `.modal-content`, `.dropdown-menu` |
| 상태 | `is-{상태}` 또는 `{컴포넌트}-active` | `.is-hidden`, `.tab-active` |
| 유틸리티 | 기능 설명 | `.text-center`, `.hidden` |

### 2.2 애니메이션 (@keyframes)

| 규칙 | 패턴 | 예시 |
|------|------|------|
| 동작 설명 | camelCase | `cursorBlink`, `glowPulse`, `scanReveal` |
| 방향 포함 | 동사 + 방향 | `driveRight`, `driveLeft`, `snowfall` |
| 상태 변화 | 명사 + 동작 | `loadingBlink`, `treeSparkle`, `twinkleChristmas` |

### 2.3 미디어 쿼리 분류

```css
/* 순서: 데스크톱 → 태블릿 → 모바일 → 터치 */
/* (1) 기본 스타일: 데스크톱 (768px 이상) */
.editor-container { display: grid; grid-template-columns: 1fr 1fr; }

/* (2) 태블릿 */
@media (max-width: 768px) { }

/* (3) 모바일 */
@media (max-width: 480px) { }

/* (4) 터치 기기 */
@media (hover: none) and (pointer: coarse) { }
```

---

## 3. API/데이터 네이밍

### 3.1 API 엔드포인트

| 규칙 | 예시 |
|------|------|
| 복수형 명사 | `/data/entries` (O), `/data/entry` (X) |
| 소문자 | `/auth/signup/password` |
| 하이픈 없음 | `/auth/signin` (O), `/auth/sign-in` (X) |

### 3.2 DB 필드

| 규칙 | 예시 |
|------|------|
| camelCase | `dateKey`, `entryNumber`, `createdBy` |
| 시간 필드: ~At | `createdAt`, `updatedAt` |
| Boolean 필드: is~ | (현재 해당 없음) |
| 외래키: ~By/~Id | `createdBy` |

### 3.3 쿼리 파라미터

```
andFilters={"dateKey":"2025-12-18"}    // 필터
sortBy=dateKey                          // 정렬 기준
sortDirection=desc                      // 정렬 방향
limit=1000                              // 페이지 크기
```

---

## 4. 파일/폴더 네이밍

### 4.1 프로젝트 루트

```
doogie-diary/              # 프로젝트: kebab-case
├── index.html             # 메인 앱: kebab-case
├── gif.worker.js          # 워커: kebab-case + .worker 접미사
├── vite.config.js         # 설정: kebab-case + .config 접미사
├── vercel.json            # 배포: kebab-case
├── package.json           # npm 설정
├── CONVENTIONS.md         # 컨벤션: UPPER (프로젝트 메타 문서)
├── CLAUDE.md              # AI 지침: UPPER
├── DOOGIE_SPEC.md         # 기획서: UPPER
├── README.md              # 설명서: UPPER
└── BLOG_POST.md           # 블로그: UPPER
```

### 4.2 문서 폴더

```
docs/
├── 01-plan/               # 번호-이름 (정렬 보장)
│   ├── glossary.md        # kebab-case
│   ├── schema.md
│   └── domain-model.md
└── 02-convention/
    ├── naming.md
    └── structure.md
```

### 4.3 이미지/에셋

```
image/
├── doogie.png             # kebab-case (권장)
└── Doogie_diary2.png      # 기존 파일 (변경 불필요)
```

---

## 5. 다국어 키 네이밍

```javascript
// 한국어/영문 번역 객체의 키는 영문 기능 설명
const translations = {
    diary: { ko: '일기쓰기', en: 'Diary' },
    memory: { ko: '나의 일기장', en: 'Memory' },
    diskette: { ko: '디스켓', en: 'Diskette' },
    save: { ko: '저장', en: 'Save' },
    newEntry: { ko: '새 일기', en: 'New' },
    // ...
};
```

---

## 6. 네이밍 안티패턴 (하지 말 것)

| 안티패턴 | 예시 | 올바른 예시 |
|----------|------|------------|
| 약어 남용 | `cnt`, `btn`, `msg` | `count`, `button`, `message` |
| 숫자 접미사 | `entry1`, `entry2` | `firstEntry`, `currentEntry` |
| 의미 없는 이름 | `data`, `temp`, `val` | `entryData`, `tempCanvas`, `brightness` |
| 한글 변수명 | `const 일기 = ...` | `const entry = ...` |
| 불일치 동사 | `fetch`와 `get` 혼용 | `get` 통일 (Bkend API 패턴) |

**예외**: 반복문 변수 (`i`, `j`), 이벤트 객체 (`e`), 에러 (`error`)는 짧은 이름 허용

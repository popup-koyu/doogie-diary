# DOOGIE - UI 구현 + API 연동 명세 (Phase 6)

> 화면 구현과 백엔드 API 연동 패턴 정의

**버전**: 1.0
**최종 수정일**: 2026-02-18
**프로젝트**: DOOGIE Diary

---

## 1. 아키텍처 개요

### 1.1 Single HTML File 통합 구조

DOOGIE는 프레임워크 없이 Vanilla JS로 구현된 Single HTML File 앱입니다.
React/Next.js의 컴포넌트 분리 대신 **레이어 기반 통합 구조**를 사용합니다.

```
┌─────────────────────────────────────────────────────────┐
│                    UI Layer (DOM)                        │
│     이벤트 리스너, 화면 전환, DOM 조작                      │
├─────────────────────────────────────────────────────────┤
│                 Feature Functions                        │
│   handleAuthSubmit, saveCurrentEntry, renderCalendar    │
├─────────────────────────────────────────────────────────┤
│                   class DiaryDB                         │
│          IndexedDB 래퍼 (로컬 저장소)                      │
├─────────────────────────────────────────────────────────┤
│                  class BkendAPI                         │
│        HTTP 클라이언트 (서버 통신, 인증 관리)                │
├─────────────────────────────────────────────────────────┤
│                 Constants & Config                       │
│       USE_SERVER_MODE, PROJECT_ID, BASE_URL             │
└─────────────────────────────────────────────────────────┘
```

### 1.2 이중 저장 모드 (Hybrid Storage)

```
┌──────────────────────────────────────────────────┐
│                Feature Functions                  │
│           saveEntrySingle(), loadEntries()        │
├───────────────┬──────────────────────────────────┤
│  Server Mode  │        Local Mode                │
│ (USE_SERVER_  │   (USE_SERVER_MODE = false)       │
│  MODE = true) │                                  │
├───────────────┼──────────────────────────────────┤
│  BkendAPI     │     DiaryDB (IndexedDB)          │
│  (REST API)   │     (브라우저 내장 DB)             │
├───────────────┼──────────────────────────────────┤
│  MongoDB      │     IndexedDB                    │
│  (클라우드)    │     (로컬)                        │
└───────────────┴──────────────────────────────────┘
```

---

## 2. API 클라이언트 (BkendAPI)

### 2.1 클래스 구조

```javascript
class BkendAPI {
    // ===== 초기화 =====
    constructor()
    //  - baseUrl: '/api'
    //  - projectId: 'xmw50no31alhxozn08ex'
    //  - environment: 'dev'
    //  - accessToken: localStorage에서 복원

    // ===== 핵심 요청 메서드 =====
    async request(endpoint, options = {})
    //  - 오프라인 체크 (navigator.onLine)
    //  - 공통 헤더 자동 주입
    //  - 401 응답 시 자동 로그아웃
    //  - 에러 파싱 및 throw

    // ===== 인증 메서드 =====
    signup(nickname, pin)        // POST /auth/signup
    signin(nickname, pin)        // POST /auth/signin/password
    fetchCurrentUser()           // GET /auth/me
    checkNicknameAvailable(name) // 더미 로그인 시도로 판별
    logout()                     // 클라이언트 토큰 삭제
    isLoggedIn()                 // accessToken + currentUser 확인

    // ===== 일기 CRUD =====
    getAllEntries()              // GET /data/entries (limit=1000)
    getEntriesByDate(dateKey)    // GET /data/entries (andFilters)
    getEntriesByMonth(y, m)     // GET /data/entries ($regex)
    getTodayCount()             // GET /data/entries (count)
    createEntry(...)            // POST /data/entries
    updateEntry(id, updates)    // PUT /data/entries/{id}
    deleteEntry(id)             // DELETE /data/entries/{id}
    getNextEntryNumber(dateKey) // 서버 조회 → 다음 번호
    canCreateEntryToday()       // 100개 제한 체크

    // ===== 유틸리티 =====
    nicknameToEmail(nickname)   // → {nickname}@doogie.app
    pinToPassword(pin)          // → Doo@gie{pin}#Pwd
}
```

### 2.2 공통 헤더

```
모든 요청에 자동 주입:

Content-Type:   application/json
X-Project-Id:   xmw50no31alhxozn08ex
X-Environment:  dev
Authorization:  Bearer {accessToken}    ← 로그인 시에만
```

### 2.3 요청/응답 흐름

```
Feature Function
    │
    ▼
bkendAPI.request(endpoint, options)
    │
    ├── navigator.onLine 체크 → false → throw OFFLINE
    │
    ├── headers 자동 구성
    │   ├── Content-Type: application/json
    │   ├── X-Project-Id
    │   ├── X-Environment
    │   └── Authorization: Bearer {token}
    │
    ├── fetch(url, options)
    │
    ├── response.ok?
    │   ├── true  → return response.json()
    │   └── false → 에러 파싱
    │       ├── 401 → 자동 logout() + throw SESSION_EXPIRED
    │       └── 4xx/5xx → throw 에러 메시지
    │
    └── catch → throw NETWORK_ERROR
```

---

## 3. 로컬 저장소 (DiaryDB)

### 3.1 IndexedDB 구조

```
Database: 'doogie-diary'

Object Stores:
┌──────────────┬──────────────┬──────────────────────┐
│ Store Name   │ keyPath      │ 용도                  │
├──────────────┼──────────────┼──────────────────────┤
│ entries      │ { key }      │ 일기 데이터 캐시       │
│ settings     │ { key }      │ 앱 설정 (언어 등)      │
│ characters   │ { key }      │ 캐릭터 초상화          │
└──────────────┴──────────────┴──────────────────────┘
```

### 3.2 주요 메서드

| 메서드 | 파라미터 | 용도 |
|--------|----------|------|
| `saveEntry(key, data)` | key: 엔트리 키, data: 엔트리 객체 | 단건 저장 |
| `saveAllEntries(entries)` | entries: 전체 엔트리 배열 | 전체 교체 |
| `getAllEntries()` | - | 전체 조회 |
| `getEntriesByDate(date)` | date: YYYY-MM-DD | 날짜별 조회 |
| `saveSetting(key, value)` | key: 설정 키, value: 값 | 설정 저장 |
| `getSetting(key)` | key: 설정 키 | 설정 조회 |
| `saveCharacters(chars)` | chars: 캐릭터 배열 | 캐릭터 저장 |
| `migrateFromLocalStorage()` | - | localStorage → IndexedDB 마이그레이션 |

---

## 4. 상태 관리

### 4.1 전역 상태 변수

```javascript
// ===== 엔트리 상태 =====
let entries = {};              // { key: { text, images, _id, ... } }
let currentDate;               // 현재 보고 있는 날짜 (ISO)
let currentEntryNumber;        // 현재 엔트리 번호 (같은 날짜 내)
let currentEntryId;            // Bkend MongoDB _id (업데이트용)
let lastSavedContent = '';     // 중복 저장 방지용 비교값

// ===== 인증 상태 =====
let isAuthMode = 'login';     // 'login' | 'signup'
let nicknameCheckTimeout;      // 닉네임 중복체크 디바운스

// ===== 에디터 상태 =====
let autoSaveTimeout;           // 2초 자동저장 디바운스
let editorMode = 'diary';     // 'diary' | 'markdown'
let markdownPreviewVisible;    // 마크다운 프리뷰 토글

// ===== UI 상태 =====
let currentCalendarMonth;      // 달력 현재 월
let currentCalendarYear;       // 달력 현재 연도
```

### 4.2 상태 영속성 매핑

```
┌─────────────────┬──────────────────────┬────────────────┐
│ 상태             │ 저장 위치             │ 수명            │
├─────────────────┼──────────────────────┼────────────────┤
│ 일기 데이터       │ MongoDB (서버)        │ 영구           │
│                 │ IndexedDB (로컬 캐시)  │ 영구           │
│ 인증 토큰        │ localStorage          │ 세션 기반       │
│ 앱 설정          │ IndexedDB (settings)  │ 영구           │
│ 캐릭터           │ IndexedDB (chars)     │ 영구           │
│ 에디터 상태       │ 메모리 (변수)          │ 세션 내         │
│ UI 상태          │ 메모리 (변수)          │ 세션 내         │
└─────────────────┴──────────────────────┴────────────────┘
```

---

## 5. 핵심 통합 흐름

### 5.1 인증 흐름

```
[타이틀 화면]
    │
    ├── "Login/Signup" 클릭
    │
    ▼
[인증 화면 표시] ← showAuthScreen()
    │
    ├── 탭 전환: Login ↔ Signup
    │
    ├── (Signup 모드) 닉네임 입력
    │   └── 500ms 디바운스 → checkNicknameAvailable()
    │       ├── "✓ 사용 가능" (녹색)
    │       └── "✗ 이미 사용 중" (빨간색)
    │
    ├── 폼 제출 → handleAuthSubmit()
    │   │
    │   ├── 유효성 검사
    │   │   ├── 닉네임: 2-12자
    │   │   └── PIN: 4자리 숫자
    │   │
    │   ├── showLoading('로그인 중...')
    │   │
    │   ├── bkendAPI.signin() 또는 signup()
    │   │   ├── 닉네임 → 이메일 변환
    │   │   └── PIN → 비밀번호 변환
    │   │
    │   ├── 성공 시:
    │   │   ├── hideAuthScreen()
    │   │   ├── updateAuthUI()     → 메뉴바에 닉네임 표시
    │   │   ├── loadEntries()      → 서버에서 일기 로드
    │   │   └── updateTodayCount() → 오늘 작성 수 표시
    │   │
    │   └── 실패 시:
    │       └── 에러 메시지 표시 (한국어)
    │           ├── "닉네임 또는 PIN이 올바르지 않습니다"
    │           ├── "이미 존재하는 닉네임입니다"
    │           └── "존재하지 않는 사용자입니다"
    │
    └── hideLoading()
```

### 5.2 세션 복원 흐름

```
[앱 시작 - init()]
    │
    ├── localStorage에서 토큰 확인
    │
    ├── 토큰 있음 → tryRestoreSession()
    │   │
    │   ├── bkendAPI.fetchCurrentUser()
    │   │   ├── 성공 → updateAuthUI() + loadEntries()
    │   │   └── 실패 → logout() (토큰 만료)
    │   │
    │   └── 화면 전환 없이 자동 로그인
    │
    └── 토큰 없음 → 타이틀 화면 유지
```

### 5.3 일기 작성/저장 흐름

```
[에디터 화면]
    │
    ├── 사용자가 텍스트 입력
    │   │
    │   ├── input 이벤트 발생
    │   │   ├── updateCharCount()  → 글자수 표시
    │   │   ├── clearTimeout(autoSaveTimeout)
    │   │   └── setTimeout(2초) → saveCurrentEntry()
    │   │
    │   └── 2초 동안 추가 입력 없으면 자동 저장 시작
    │
    ├── saveCurrentEntry()
    │   │
    │   ├── 중복 저장 방지
    │   │   └── content === lastSavedContent → "이미 저장됨" → 종료
    │   │
    │   ├── 빈 내용 + 이미지 없음 → 저장 건너뜀
    │   │
    │   └── saveEntrySingle(key, data)
    │       │
    │       ├── showStatus('Saving...', 0)
    │       │
    │       ├── [Server Mode]
    │       │   ├── data._id 존재 → updateEntry(id, updates)
    │       │   │   └── PUT /data/entries/{id}
    │       │   │
    │       │   └── data._id 없음 → createEntry(...)
    │       │       ├── POST /data/entries
    │       │       ├── 성공 → _id 저장 (이후 update 용)
    │       │       └── E11000 중복 → 재시도 (최대 3회)
    │       │           └── 서버에서 최신 번호 조회 후 재생성
    │       │
    │       ├── [Local Mode]
    │       │   └── diaryDB.saveEntry(key, data)
    │       │
    │       ├── lastSavedContent = content
    │       └── showStatus('Saved!', 2000)
    │
    └── updateTodayCount()
```

### 5.4 일기 로드 흐름

```
[앱 시작 또는 로그인 후]
    │
    ▼
loadEntries()
    │
    ├── [Server Mode]
    │   ├── bkendAPI.isLoggedIn() 확인
    │   ├── bkendAPI.getAllEntries()
    │   │   └── GET /data/entries?limit=1000&sortBy=dateKey
    │   │       → { data: { items: [...] } }
    │   │
    │   └── 서버 → 로컬 포맷 변환:
    │       서버: { dateKey, entryNumber, type, text, images, _id }
    │       로컬: entries[key] = { text, images, _id, dateKey, ... }
    │       키 생성: generateLocalKey(dateKey, entryNumber, type)
    │         └── '#2024-01-15_(2)' (마크다운 2번째)
    │         └── '2024-01-15'      (일기 1번째)
    │
    └── [Local Mode]
        └── diaryDB.getAllEntries()
            → 파싱 후 entries 객체에 매핑

    ▼
[달력에서 날짜 클릭]
    │
    ├── loadEntryByDate(dateStr)
    │   └── 해당 날짜 첫 번째 엔트리 로드
    │
    └── loadCurrentEntry()
        ├── entries[key]에서 데이터 가져오기
        ├── editor.value = entry.text
        ├── lastSavedContent = entry.text
        └── displayImages()  → 이미지 그리드 렌더링
```

### 5.5 이미지 업로드 흐름

```
[에디터 화면 - Picture 메뉴]
    │
    ├── "Add Pictures" 클릭
    │   └── openImagePicker() → fileInput.click()
    │
    ├── 파일 선택 (복수 가능)
    │   └── handleFiles(files)
    │       │
    │       ├── 파일별 처리 루프
    │       │   ├── GIF → extractGifFrame() → 첫 프레임 추출
    │       │   └── 기타 → FileReader.readAsDataURL()
    │       │
    │       └── pixelateImage(imageSrc)
    │           │
    │           ├── Canvas에 이미지 로드
    │           │
    │           ├── 3:1 축소 (픽셀화)
    │           │
    │           ├── 녹색 팔레트 매핑
    │           │   brightness = (R + G + B) / 3
    │           │   level = floor(brightness / 64) * 64
    │           │   → R=0, G=level, B=0
    │           │
    │           ├── Thumbnail 생성 (max 600px)
    │           ├── Full 이미지 생성 (max 1500px)
    │           │
    │           └── addImageToEntry(thumbnail, full)
    │               ├── entry.images.push({ thumbnail, full })
    │               ├── saveEntrySingle() → 서버 저장
    │               ├── displayImages()   → UI 갱신
    │               └── showStatus('Image added!')
    │
    └── [이미지 클릭 시]
        └── openImageModal() → full 이미지 표시
```

### 5.6 일기 삭제 흐름

```
[엔트리 목록 또는 에디터]
    │
    ├── 삭제 버튼 클릭
    │
    └── deleteEntrySingle(key)
        │
        ├── showStatus('Deleting...', 0)
        │
        ├── [Server Mode]
        │   ├── entry._id 확인
        │   ├── bkendAPI.deleteEntry(id)
        │   │   └── DELETE /data/entries/{id}
        │   └── updateTodayCount()
        │
        ├── [Local Mode]
        │   └── diaryDB.deleteEntry(key)
        │
        ├── delete entries[key]
        ├── 에디터 초기화
        └── showStatus('Deleted!', 2000)
```

### 5.7 달력 표시 흐름

```
[달력 화면 - switchMode('calendar')]
    │
    ▼
renderCalendar()
    │
    ├── 현재 월/연도 표시
    │
    ├── entries 객체에서 날짜별 엔트리 수 집계
    │   └── entryCounts = { '2024-01-15': 3, ... }
    │
    ├── 달력 그리드 생성 (7x5~6)
    │   │
    │   └── 각 날짜 셀:
    │       ├── 오늘 → 'today' 클래스
    │       ├── 엔트리 있음 → 'has-entry' 클래스 + 녹색 점
    │       └── click → loadEntryByDate(dateStr)
    │
    └── renderCalendarEntryList()
        │
        ├── 해당 월 엔트리 필터링
        ├── 날짜 역순 정렬
        │
        └── 각 엔트리:
            ├── [MD] 표시 (마크다운인 경우)
            ├── YYYY.MM.DD (No.N) 형식
            ├── 미리보기 텍스트 (100자)
            ├── [사진 N장] 이미지 수
            └── click → loadEntry(key) → switchMode('view')
```

---

## 6. 이벤트 바인딩 체계

### 6.1 초기화 시 바인딩 (init 함수)

| 카테고리 | 함수 | 바인딩 대상 |
|----------|------|------------|
| 인증 | `setupAuthListeners()` | 폼 제출, 탭 전환, 닉네임 입력, PIN 입력, 로그아웃 |
| 메뉴 | `setupMenuListeners()` | 메뉴 항목 클릭, 마우스오버 |
| 상단 메뉴 | `setupTopMenuListeners()` | 상단 메뉴 항목 클릭 |
| 달력 | `setupCalendar()` | 이전/다음 월 버튼 |
| 이미지 | `setupImageHandlers()` | 파일 입력 변경 |
| 모달 | `setupImageModal()` | 모달 닫기, 외부 클릭 |
| 확인 | `setupConfirmDialog()` | 예/아니오 버튼 |
| 에디터 | `setupCursor()` | 커서 위치 추적 |
| 마크다운 | `setupMarkdownMode()` | 툴바, 프리뷰 토글, 내보내기 |
| 자동서식 | `setupAutoFormatting()` | 체크박스, 리스트, 탭 키 |

### 6.2 동적 바인딩

| 시점 | 대상 | 이벤트 |
|------|------|--------|
| 달력 렌더링 | 날짜 셀 | click → loadEntryByDate |
| 엔트리 목록 | 엔트리 항목 | click → loadEntry |
| 이미지 표시 | 이미지 요소 | click → openImageModal |
| 이미지 표시 | 삭제 버튼 | click → removeImage |

### 6.3 키보드 단축키

```
전역 단축키 (document.addEventListener('keydown', ...)):

┌──────────┬─────────────────────────┐
│ 키        │ 동작                    │
├──────────┼─────────────────────────┤
│ F1       │ 도움말 토글              │
│ F2       │ 현재 엔트리 저장          │
│ F3       │ 달력 보기                │
│ F4       │ 새 엔트리               │
│ ←/→     │ 날짜 이동 (에디터)        │
│ Ctrl+M   │ 에디터 모드 전환         │
│ Ctrl+P   │ 마크다운 프리뷰 토글      │
│ ESC      │ 모달/메뉴 닫기           │
└──────────┴─────────────────────────┘
```

---

## 7. 화면 전환 체계

### 7.1 화면 ID와 전환 방식

```
모든 화면은 display: none/flex 로 전환
한 번에 하나의 화면만 표시

[title-screen] ←─ returnToMenu()
    │
    ├── handleMenuAction('auth')  → [auth-screen] (모달)
    ├── handleMenuAction('write') → startApp('write')
    ├── handleMenuAction('list')  → startApp('calendar')
    └── handleMenuAction('credits') → startApp('credits')
```

### 7.2 startApp(mode) → switchMode(mode)

```
startApp(mode):
  1. titleScreen.display = 'none'
  2. appContainer.display = 'flex'
  3. updateDateHeader()
  4. loadCurrentEntry()
  5. switchMode(mode)

switchMode(mode):
  1. 모든 모드 컨테이너 숨김
  2. 선택된 모드만 표시:
     ┌──────────┬──────────────────────────────┐
     │ mode     │ 동작                          │
     ├──────────┼──────────────────────────────┤
     │ 'view'   │ 에디터 표시, editor.focus()    │
     │ 'list'   │ renderEntryList()            │
     │ 'calendar'│ renderCalendar()            │
     │ 'help'   │ 도움말 화면                    │
     │ 'settings'│ 설정 화면                    │
     │ 'credits' │ 크레딧 화면                   │
     │ 'xmascard'│ 크리스마스 카드               │
     └──────────┴──────────────────────────────┘
  3. 상태바 메시지 업데이트
```

### 7.3 인증 화면 (모달 방식)

```
showAuthScreen():
  - 오버레이 표시
  - 폼 초기화
  - 랜덤 별 생성 (배경)
  - 포커스: 닉네임 입력

hideAuthScreen():
  - 오버레이 숨김
  - 타이틀 화면으로 복귀
```

---

## 8. 에러 처리 패턴

### 8.1 API 에러 체계

```javascript
// BkendAPI.request() 내부 에러 처리

에러 유형:
┌──────────────────┬──────────────────────┬──────────────────┐
│ 에러 코드         │ 한국어 메시지          │ 후속 동작          │
├──────────────────┼──────────────────────┼──────────────────┤
│ OFFLINE          │ 인터넷 연결을 확인해    │ 상태바 5초 표시    │
│                  │ 주세요                │                  │
│ NETWORK_ERROR    │ 서버에 연결할 수       │ 상태바 5초 표시    │
│                  │ 없습니다              │                  │
│ SESSION_EXPIRED  │ 세션이 만료되었습니다   │ 1초 후 인증 화면   │
│ 401              │ (자동 로그아웃)        │ logout() 호출     │
│ invalid-         │ 닉네임 또는 PIN이      │ 인증 폼 에러 표시  │
│   credentials    │ 올바르지 않습니다      │                  │
│ duplicate        │ 이미 존재하는          │ 인증 폼 에러 표시  │
│                  │ 닉네임입니다           │                  │
│ E11000           │ (내부 처리)            │ 재시도 (3회)      │
│ (중복 키)         │                      │                  │
└──────────────────┴──────────────────────┴──────────────────┘
```

### 8.2 에러 표시 함수

```javascript
// handleApiError(error) - 전역 에러 핸들러
//   → showStatus(message, duration) - 상태바 표시

// 인증 에러: authError 요소에 직접 표시
// 저장 에러: showStatus('Error: ...', 3000)
// 네트워크 에러: showStatus('...', 5000)
// 세션 만료: showStatus() + showAuthScreen()
```

### 8.3 에러 복구 패턴

| 에러 | 복구 방법 |
|------|----------|
| 네트워크 끊김 | 상태 메시지 표시, 재시도 안내 |
| 세션 만료 | 자동 로그아웃 → 인증 화면 표시 |
| 중복 키 (E11000) | 서버에서 최신 번호 조회 → 자동 재시도 (3회) |
| 저장 실패 | 에러 메시지 표시, 수동 재시도 가능 |
| 이미지 처리 실패 | 개별 이미지 건너뜀, 나머지 계속 처리 |

---

## 9. 디바운싱 & 최적화

### 9.1 디바운스 패턴

```
┌──────────────────┬──────────┬──────────────────────┐
│ 기능              │ 대기 시간 │ 트리거                │
├──────────────────┼──────────┼──────────────────────┤
│ 자동 저장 (일기)   │ 2,000ms  │ editor.input 이벤트   │
│ 자동 저장 (마크다운)│ 2,000ms  │ markdownEditor.input │
│ 닉네임 중복 체크   │ 500ms   │ authNickname.input   │
└──────────────────┴──────────┴──────────────────────┘
```

### 9.2 중복 방지 패턴

```javascript
// 중복 저장 방지
if (content === lastSavedContent && content !== '') {
    showStatus('이미 저장되었습니다!');
    return;  // 저장 건너뜀
}

// 저장 후 비교값 업데이트
lastSavedContent = content;
```

### 9.3 동시 쓰기 충돌 해결

```
createEntry() 호출
    │
    ├── POST /data/entries
    │   ├── 성공 → _id 저장, 완료
    │   └── E11000 (중복 키) 에러
    │       │
    │       ├── 시도 횟수 < 3?
    │       │   ├── 예 → getNextEntryNumber() → 재시도
    │       │   └── 아니오 → 에러 throw
    │       │
    │       └── 서버에서 최신 entryNumber 조회
    │           └── entryNumber + 1 로 재생성
```

---

## 10. 로딩/상태 표시

### 10.1 로딩 오버레이

```
showLoading(text):
  - <div id="loadingOverlay"> 에 'active' 클래스 추가
  - 텍스트 설정: '로그인 중...', '불러오는 중...' 등
  - 전체 화면 오버레이 (z-index 높음)

hideLoading():
  - 'active' 클래스 제거
```

### 10.2 상태바 메시지

```
showStatus(message, duration):
  - 하단 좌측 상태바에 메시지 표시
  - duration > 0 → 자동으로 'Ready' 복원
  - duration = 0 → 수동으로 변경할 때까지 유지

상태 전환 예시:
  'Ready' → 'Saving...' → 'Saved!' (2초) → 'Ready'
  'Ready' → 'Deleting...' → 'Deleted!' (2초) → 'Ready'
  'Ready' → 'Image added!' (2초) → 'Ready'
```

---

## 11. 데이터 포맷 변환

### 11.1 서버 ↔ 로컬 키 매핑

```
서버 데이터:
  { dateKey: '2024-01-15', entryNumber: 1, type: 'diary' }
  { dateKey: '2024-01-15', entryNumber: 2, type: 'markdown' }
  { dateKey: '2024-01-15', entryNumber: 3, type: 'diary' }

     ↓ generateLocalKey()

로컬 키:
  '2024-01-15'        ← diary, entryNumber 1
  '#2024-01-15_(2)'   ← markdown, entryNumber 2
  '2024-01-15_(3)'    ← diary, entryNumber 3

규칙:
  - '#' 접두사 = markdown 타입
  - '_(N)' 접미사 = entryNumber > 1
  - entryNumber 1 + diary = 접미사 없음
```

### 11.2 엔트리 객체 구조

```javascript
// 로컬 entries 객체
entries = {
    '2024-01-15': {
        text: '오늘의 일기...',
        images: [
            { thumbnail: 'data:image/png;base64,...', full: 'data:image/png;base64,...' }
        ],
        _id: 'mongo_id_123',          // 서버 ID (업데이트용)
        dateKey: '2024-01-15',         // 서버 필드
        entryNumber: 1,                // 서버 필드
        type: 'diary'                  // 서버 필드
    },
    '#2024-01-15_(2)': {
        text: '# 마크다운 일기\n...',
        images: [],
        _id: 'mongo_id_456',
        dateKey: '2024-01-15',
        entryNumber: 2,
        type: 'markdown'
    }
}
```

---

## 12. 보안 고려사항

### 12.1 인증 정보 처리

| 항목 | 처리 방식 |
|------|----------|
| 닉네임 | 클라이언트에서 이메일로 변환 (`@doogie.app`) |
| PIN | 클라이언트에서 비밀번호로 변환 (`Doo@gie{pin}#Pwd`) |
| 토큰 | localStorage에 저장 (`doogie_access_token`) |
| 401 응답 | 자동 로그아웃 (토큰 삭제 + 인증 화면) |

### 12.2 데이터 보호

| 항목 | 처리 방식 |
|------|----------|
| 이미지 | Base64로 인코딩하여 엔트리에 포함 저장 |
| 텍스트 | 서버 전송 시 JSON body (HTTPS) |
| XSS 방지 | 마크다운 렌더링 시 sanitize 처리 |
| 페이지 이탈 | `beforeunload` 이벤트로 미저장 데이터 경고 |

---

## 13. Zero Script QA 체크리스트

### 13.1 인증 통합 테스트

```
[QA-UI-AUTH-01] 회원가입 → 에디터 진입
  1. 타이틀 > Login/Signup 클릭
  2. Signup 탭 선택
  3. 닉네임 입력 → "✓ 사용 가능" 확인
  4. PIN 4자리 입력
  5. 제출
  [기대] 인증 화면 닫힘, 메뉴바에 닉네임, 빈 에디터

[QA-UI-AUTH-02] 로그인 → 기존 데이터 로드
  1. 기존 계정으로 로그인
  [기대] 서버에서 일기 로드, 달력에 기존 엔트리 표시

[QA-UI-AUTH-03] 세션 복원
  1. 로그인 후 페이지 새로고침
  [기대] 자동 로그인, 데이터 유지

[QA-UI-AUTH-04] 세션 만료 처리
  1. 토큰 만료 상태에서 저장 시도
  [기대] "세션이 만료되었습니다" → 인증 화면 표시
```

### 13.2 일기 CRUD 통합 테스트

```
[QA-UI-CRUD-01] 새 일기 작성 + 자동 저장
  1. 에디터에 텍스트 입력
  2. 2초 대기
  [기대] 상태바: "Saving..." → "Saved!"

[QA-UI-CRUD-02] 기존 일기 수정
  1. 달력에서 기존 엔트리 클릭
  2. 텍스트 수정
  3. 2초 대기
  [기대] PUT 요청, "Saved!" 표시

[QA-UI-CRUD-03] 일기 삭제
  1. 엔트리 삭제 버튼 클릭
  [기대] DELETE 요청, 목록에서 제거

[QA-UI-CRUD-04] 하루 100개 제한
  1. 100개 엔트리가 있는 날짜에서 새 엔트리 시도
  [기대] 생성 불가 안내 메시지

[QA-UI-CRUD-05] 중복 키 충돌 복구
  1. 동시에 같은 날짜로 엔트리 생성 (탭 2개)
  [기대] 자동 재시도로 다른 entryNumber 할당
```

### 13.3 이미지 통합 테스트

```
[QA-UI-IMG-01] 이미지 추가
  1. Picture > Add Pictures
  2. 이미지 파일 선택
  [기대] 녹색 픽셀아트로 변환, 에디터에 표시

[QA-UI-IMG-02] 복수 이미지 추가
  1. 여러 이미지 동시 선택
  [기대] 각각 변환 후 이미지 그리드에 표시

[QA-UI-IMG-03] GIF 처리
  1. GIF 파일 추가
  [기대] 첫 프레임 추출 → 픽셀화 → 정지 이미지로 저장

[QA-UI-IMG-04] 이미지 모달
  1. 이미지 클릭
  [기대] Full 사이즈 이미지 모달 표시
```

### 13.4 화면 전환 테스트

```
[QA-UI-NAV-01] 타이틀 → 에디터
  1. 로그인 후 "Write" 클릭
  [기대] 타이틀 숨김, 에디터 표시, 날짜 헤더 업데이트

[QA-UI-NAV-02] 에디터 → 달력 → 에디터
  1. F3으로 달력 진입
  2. 날짜 클릭
  [기대] 해당 날짜 엔트리 로드, 에디터 표시

[QA-UI-NAV-03] 미저장 확인 다이얼로그
  1. 텍스트 입력 (저장 전)
  2. "New" 클릭
  [기대] "Save changes?" 확인 다이얼로그 표시

[QA-UI-NAV-04] 키보드 내비게이션
  1. F1~F4, Ctrl+M, Ctrl+P, ESC 테스트
  [기대] 각 단축키별 올바른 화면/기능 전환
```

### 13.5 오프라인/에러 테스트

```
[QA-UI-ERR-01] 오프라인 상태 저장
  1. 네트워크 끊고 저장 시도
  [기대] "인터넷 연결을 확인해주세요" 메시지

[QA-UI-ERR-02] 서버 다운 시
  1. 서버 응답 없는 상태에서 API 호출
  [기대] "서버에 연결할 수 없습니다" 메시지

[QA-UI-ERR-03] 잘못된 인증 정보
  1. 존재하지 않는 닉네임/PIN으로 로그인
  [기대] 한국어 에러 메시지, 폼 유지
```

---

## 14. 구현 패턴 요약

### 14.1 DOOGIE에서 지켜야 할 패턴

| # | 패턴 | 설명 |
|---|------|------|
| 1 | **레이어 순서** | Constants → BkendAPI → DiaryDB → Features → init() |
| 2 | **API 호출 위치** | Feature 함수에서만 (DOM 이벤트 핸들러에서 직접 호출 X) |
| 3 | **에러 처리** | try-catch + handleApiError() + 한국어 메시지 |
| 4 | **로딩 표시** | 긴 작업 → showLoading(), 짧은 작업 → showStatus() |
| 5 | **자동 저장** | 2초 디바운스, 중복 저장 방지 (lastSavedContent) |
| 6 | **화면 전환** | switchMode() 중앙 집중, display none/flex |
| 7 | **이중 모드** | USE_SERVER_MODE 분기, 로컬/서버 동일 인터페이스 |
| 8 | **동시성** | E11000 재시도 (3회), getNextEntryNumber() |

### 14.2 새 기능 추가 시 체크리스트

```
[ ] BkendAPI에 API 메서드 추가 (필요 시)
[ ] Feature 함수 작성 (try-catch, 에러 처리)
[ ] DOM 이벤트 리스너 setup 함수에 바인딩
[ ] 로딩/상태 표시 적용
[ ] Server Mode / Local Mode 분기 처리
[ ] 키보드 단축키 추가 (필요 시)
[ ] 반응형 대응 (모바일)
[ ] QA 체크리스트 항목 추가
```

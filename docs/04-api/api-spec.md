# DOOGIE - API Specification (API 설계서)

> Bkend BaaS 기반 API 엔드포인트, 요청/응답 형식, 에러 처리를 정의하는 문서

**버전**: 1.0
**최종 수정일**: 2026-02-18
**프로젝트**: DOOGIE Diary
**백엔드**: Bkend (BaaS) - `api-enduser.bkend.ai`

---

## 1. API 개요

### 1.1 Base URL

| 환경 | Base URL | 설명 |
|------|----------|------|
| 개발 | `http://localhost:3000/api` | Vite 프록시 → Bkend |
| 프로덕션 | `https://{도메인}/api` | Vercel 리라이트 → Bkend |
| Bkend 직접 | `https://api-enduser.bkend.ai` | 원본 서버 |

### 1.2 프록시 설정

**개발 (vite.config.js)**:
```javascript
proxy: {
  '/api': {
    target: 'https://api-enduser.bkend.ai',
    changeOrigin: true,
    rewrite: (path) => path.replace(/^\/api/, ''),
  },
}
```

**프로덕션 (vercel.json)**:
```json
{
  "rewrites": [
    { "source": "/api/:path*", "destination": "https://api-enduser.bkend.ai/:path*" }
  ]
}
```

### 1.3 필수 헤더

| 헤더 | 값 | 필수 |
|------|-----|------|
| `Content-Type` | `application/json` | 모든 요청 |
| `X-Project-Id` | `xmw50no31alhxozn08ex` | 모든 요청 |
| `X-Environment` | `dev` | 모든 요청 |
| `Authorization` | `Bearer {access_token}` | 인증 필요 요청 |

### 1.4 프론트엔드 클라이언트

```javascript
class BkendAPI {
    constructor() {
        this.baseUrl = '/api';
        this.projectId = 'xmw50no31alhxozn08ex';
        this.environment = 'dev';
        this.accessToken = localStorage.getItem('doogie_access_token') || null;
    }

    async request(endpoint, options = {}) {
        const headers = {
            'Content-Type': 'application/json',
            'X-Project-Id': this.projectId,
            'X-Environment': this.environment,
            ...options.headers
        };
        if (this.accessToken) {
            headers['Authorization'] = `Bearer ${this.accessToken}`;
        }
        const response = await fetch(`${this.baseUrl}${endpoint}`, { ...options, headers });
        // ... 에러 처리
        return await response.json();
    }
}
```

---

## 2. 인증 API (Auth)

### 2.1 회원가입

**닉네임 + PIN → 이메일 + 패스워드 변환 후 전송**

```
POST /api/auth/signup/password
```

**변환 규칙**:
```
닉네임 "홍길동" → email: "홍길동@doogie.app"
PIN "1234"     → password: "Doo@gie1234#Pwd"
```

**Request Body**:
```json
{
  "email": "홍길동@doogie.app",
  "password": "Doo@gie1234#Pwd",
  "name": "홍길동",
  "callbackUrl": "https://doogie.app"
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "_id": "65a1b2c3d4e5f6789...",
      "email": "홍길동@doogie.app",
      "name": "홍길동"
    }
  }
}
```

**Error (409 Conflict)** - 닉네임 중복:
```json
{
  "success": false,
  "error": {
    "code": "auth/email-already-exists",
    "message": "This email is already registered."
  }
}
```

**프론트엔드 처리**:
```javascript
async signup(nickname, pin) {
    const email = this.nicknameToEmail(nickname);      // "닉네임@doogie.app"
    const password = this.pinToPassword(pin);           // "Doo@gie{pin}#Pwd"
    const data = await this.request('/auth/signup/password', {
        method: 'POST',
        body: JSON.stringify({ email, password, name: nickname, callbackUrl: window.location.origin })
    });
    if (data.data?.access_token) {
        this.accessToken = data.data.access_token;
        localStorage.setItem('doogie_access_token', this.accessToken);
        await this.fetchCurrentUser();
    }
    return data;
}
```

---

### 2.2 로그인

```
POST /api/auth/signin/password
```

**Request Body**:
```json
{
  "email": "홍길동@doogie.app",
  "password": "Doo@gie1234#Pwd",
  "callbackUrl": "https://doogie.app"
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "_id": "65a1b2c3d4e5f6789...",
      "email": "홍길동@doogie.app"
    }
  }
}
```

**Error (401 Unauthorized)** - 인증 실패:
```json
{
  "success": false,
  "error": {
    "code": "auth/invalid-credentials",
    "message": "Invalid email or password."
  }
}
```

---

### 2.3 내 정보 조회

```
GET /api/auth/me
Authorization: Bearer {access_token}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "_id": "65a1b2c3d4e5f6789...",
    "email": "홍길동@doogie.app",
    "name": "홍길동",
    "createdAt": "2025-12-18T10:00:00Z"
  }
}
```

**Error (401 Unauthorized)** - 토큰 만료:
- 프론트엔드에서 자동 로그아웃 처리

---

### 2.4 닉네임 중복 체크

> Bkend에 별도 API가 없으므로 더미 로그인 시도로 확인

```
POST /api/auth/signin/password
```

**Request Body** (더미 패스워드):
```json
{
  "email": "체크할닉네임@doogie.app",
  "password": "DummyCheck!123#Pwd"
}
```

**판정 로직**:

| 응답 | 판정 | 닉네임 상태 |
|------|------|------------|
| `error.code === "auth/invalid-credentials"` | 계정 존재 | 사용 불가 (✗) |
| 기타 에러 | 계정 없음 | 사용 가능 (✓) |

---

### 2.5 로그아웃

> 서버 API 없음. 클라이언트 측 처리만.

```javascript
logout() {
    this.accessToken = null;
    this.currentUser = null;
    localStorage.removeItem('doogie_access_token');
}
```

---

## 3. 일기 CRUD API (Entries)

### 3.1 일기 목록 조회 (전체)

```
GET /api/data/entries?limit=1000&sortBy=dateKey&sortDirection=desc
Authorization: Bearer {access_token}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "_id": "65b2c3d4e5f67890...",
        "createdBy": "65a1b2c3d4e5f6789...",
        "dateKey": "2026-02-18",
        "entryNumber": 1,
        "type": "diary",
        "text": "오늘의 일기 내용...",
        "images": [
          { "thumbnail": "data:image/png;base64,...", "full": "data:image/png;base64,..." }
        ],
        "createdAt": "2026-02-18T09:30:00Z",
        "updatedAt": "2026-02-18T09:35:00Z"
      }
    ],
    "total": 42
  }
}
```

---

### 3.2 특정 날짜 일기 조회

```
GET /api/data/entries?andFilters={"dateKey":"2026-02-18"}&sortBy=entryNumber&sortDirection=asc
Authorization: Bearer {access_token}
```

**쿼리 파라미터**:

| 파라미터 | 값 | 설명 |
|---------|-----|------|
| `andFilters` | `{"dateKey":"YYYY-MM-DD"}` | URL 인코딩 필요 |
| `sortBy` | `entryNumber` | 항목 번호순 |
| `sortDirection` | `asc` | 오름차순 |

---

### 3.3 특정 월 일기 조회 (달력 뷰)

```
GET /api/data/entries?andFilters={"dateKey":{"$regex":"^2026-02"}}&limit=1000&sortBy=dateKey&sortDirection=desc
Authorization: Bearer {access_token}
```

**$regex 패턴**: `^YYYY-MM` (해당 월의 모든 날짜 매칭)

---

### 3.4 일기 생성

```
POST /api/data/entries
Authorization: Bearer {access_token}
```

**Request Body**:
```json
{
  "dateKey": "2026-02-18",
  "entryNumber": 1,
  "type": "diary",
  "text": "오늘의 일기 내용...",
  "images": [
    {
      "thumbnail": "data:image/png;base64,...",
      "full": "data:image/png;base64,..."
    }
  ]
}
```

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "_id": "65b2c3d4e5f67890...",
    "createdBy": "65a1b2c3d4e5f6789...",
    "dateKey": "2026-02-18",
    "entryNumber": 1,
    "type": "diary",
    "text": "오늘의 일기 내용...",
    "images": [...],
    "createdAt": "2026-02-18T09:30:00Z",
    "updatedAt": "2026-02-18T09:30:00Z"
  }
}
```

**Error (409 Conflict)** - 중복 (Unique Index 위반):
```json
{
  "success": false,
  "error": {
    "message": "Duplicate key error"
  }
}
```

**중복 저장 재시도 로직**:
```javascript
// 최대 3회 재시도
for (let retry = 0; retry < 3; retry++) {
    try {
        return await api.createEntry(dateKey, entryNumber, type, text, images);
    } catch (error) {
        if (error.message.includes('Duplicate') && retry < 2) {
            entryNumber = await api.getNextEntryNumber(dateKey);
            continue;
        }
        throw error;
    }
}
```

---

### 3.5 일기 수정

```
PUT /api/data/entries/{id}
Authorization: Bearer {access_token}
```

**Request Body** (수정 가능 필드만):
```json
{
  "text": "수정된 일기 내용...",
  "images": [
    { "thumbnail": "data:image/png;base64,...", "full": "data:image/png;base64,..." }
  ]
}
```

**수정 불가 필드**: `dateKey`, `entryNumber`, `type`, `createdBy`

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "_id": "65b2c3d4e5f67890...",
    "text": "수정된 일기 내용...",
    "updatedAt": "2026-02-18T10:00:00Z"
  }
}
```

---

### 3.6 일기 삭제

```
DELETE /api/data/entries/{id}
Authorization: Bearer {access_token}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": null
}
```

---

## 4. 쿼리 패턴 레퍼런스

### 4.1 필터링 (andFilters)

```javascript
// 단일 필드 매칭
andFilters={"dateKey":"2026-02-18"}

// 정규식 매칭 (월별)
andFilters={"dateKey":{"$regex":"^2026-02"}}

// 타입 필터
andFilters={"type":"markdown"}

// 복합 필터
andFilters={"dateKey":"2026-02-18","type":"diary"}
```

### 4.2 정렬 (sortBy + sortDirection)

```javascript
// 날짜 내림차순 (최신순)
sortBy=dateKey&sortDirection=desc

// 항목 번호 오름차순
sortBy=entryNumber&sortDirection=asc
```

### 4.3 페이지네이션 (limit)

```javascript
// 최대 1000개
limit=1000

// 기본값 (Bkend 기본 20개)
// limit 생략 시
```

---

## 5. 에러 처리

### 5.1 에러 응답 형식

Bkend 에러 응답은 여러 형식이 가능:

```javascript
// 형식 1: error 객체
{ "success": false, "error": { "code": "...", "message": "..." } }

// 형식 2: error 문자열
{ "success": false, "error": "에러 메시지" }

// 형식 3: message 문자열
{ "message": "에러 메시지" }
```

### 5.2 프론트엔드 에러 분류

| HTTP 상태 | 에러 유형 | 프론트엔드 처리 |
|----------|---------|----------------|
| 401 | 인증 만료 | 자동 로그아웃 → 로그인 화면 |
| 409 | 중복 저장 | entryNumber 재조회 → 재시도 (최대 3회) |
| 네트워크 에러 | 연결 실패 | "서버에 연결할 수 없습니다" 메시지 |
| 오프라인 | navigator.onLine false | "오프라인 상태입니다" 메시지 |
| 기타 | 서버 에러 | 에러 메시지 표시 |

### 5.3 에러 메시지 (한/영)

| 에러 | 한국어 | English |
|------|--------|---------|
| 네트워크 | 서버에 연결할 수 없습니다 | Cannot connect to server |
| 인증 만료 | 세션이 만료되었습니다 | Session expired |
| 100개 초과 | 오늘은 더 이상 일기를 쓸 수 없습니다 (100/100) | Cannot write more today (100/100) |
| 저장 실패 | 저장에 실패했습니다 | Save failed |
| 닉네임 중복 | 이미 사용 중인 닉네임입니다 | Nickname already taken |
| 인증 실패 | 닉네임 또는 PIN이 올바르지 않습니다 | Invalid nickname or PIN |

---

## 6. 비즈니스 로직 플로우

### 6.1 자동 저장 플로우

```
[텍스트 입력] → [debounce 2초 타이머 리셋]
                        ↓ 2초 경과
                [내용 변경 확인]
                    ↓ 변경됨          ↓ 미변경
            [API 호출 (POST/PUT)]    [스킵]
                ↓ 성공     ↓ 실패
          ["저장됨!"]   [에러 처리]
```

### 6.2 새 일기 생성 플로우

```
[New 버튼 클릭]
      ↓
[오늘 일기 수 조회 (getTodayCount)]
      ↓
[100개 미만?]
  ↓ Yes                    ↓ No
[getNextEntryNumber]    ["100/100 초과" 메시지]
      ↓
[에디터 초기화]
      ↓
[텍스트 입력 → 자동 저장]
      ↓
[createEntry API 호출]
  ↓ 성공          ↓ 중복 에러
[저장 완료]    [entryNumber 재조회 → 재시도 (최대 3회)]
```

### 6.3 인증 플로우

```
[닉네임 + PIN 입력]
        ↓
[닉네임→이메일, PIN→패스워드 변환]
        ↓
[로그인 or 회원가입 API 호출]
        ↓ 성공
[access_token 수령]
        ↓
[localStorage에 토큰 저장]
        ↓
[fetchCurrentUser() 호출]
        ↓
[UI 갱신 (닉네임 표시, 메뉴 활성화)]
```

---

## 7. 로컬 저장소 (IndexedDB + localStorage)

> 서버 API가 아닌 브라우저 로컬 저장 영역

### 7.1 localStorage

| 키 | 타입 | 용도 | API 관련 |
|----|------|------|---------|
| `doogie_access_token` | String | JWT 토큰 | Authorization 헤더에 사용 |
| `doogie_language` | String | 언어 설정 | API 무관 (로컬) |
| `doogie_christmas` | String | 크리스마스 모드 | API 무관 (로컬) |

### 7.2 IndexedDB (doogie-diary-db)

| Store | Key | 용도 | API 관련 |
|-------|-----|------|---------|
| `characters` | slotIndex | 크레딧 캐릭터 데이터 | API 무관 (로컬) |
| `settings` | key | 설정값 | API 무관 (로컬) |
| `entries` | key | (레거시, 현재 미사용) | Bkend로 이전됨 |

---

## 8. Zero Script QA 체크리스트

### 인증 QA

| # | 테스트 | 기대 결과 | 확인 |
|---|--------|---------|------|
| 1 | 새 닉네임으로 회원가입 | 200, access_token 발급 | [ ] |
| 2 | 중복 닉네임으로 회원가입 | 409, email-already-exists | [ ] |
| 3 | 올바른 PIN으로 로그인 | 200, access_token 발급 | [ ] |
| 4 | 틀린 PIN으로 로그인 | 401, invalid-credentials | [ ] |
| 5 | 만료 토큰으로 API 호출 | 401, 자동 로그아웃 | [ ] |
| 6 | 닉네임 중복 체크 (존재) | invalid-credentials → 사용 불가 | [ ] |
| 7 | 닉네임 중복 체크 (미존재) | 기타 에러 → 사용 가능 | [ ] |

### 일기 CRUD QA

| # | 테스트 | 기대 결과 | 확인 |
|---|--------|---------|------|
| 1 | 일기 생성 (diary) | 201, _id 반환 | [ ] |
| 2 | 일기 생성 (markdown) | 201, type: "markdown" | [ ] |
| 3 | 같은 날 2번째 일기 생성 | entryNumber: 2 | [ ] |
| 4 | 일기 목록 조회 | items 배열, 최신순 | [ ] |
| 5 | 날짜별 조회 | 해당 날짜만 반환 | [ ] |
| 6 | 월별 조회 (달력) | 해당 월만 반환 | [ ] |
| 7 | 일기 수정 (text) | updatedAt 갱신 | [ ] |
| 8 | 일기 수정 (images 추가) | images 배열 업데이트 | [ ] |
| 9 | 일기 삭제 | 200, 리스트에서 제거 | [ ] |
| 10 | 중복 저장 시 재시도 | 409 → 재조회 → 재시도 성공 | [ ] |
| 11 | 100개 초과 시 생성 | 생성 차단, 메시지 표시 | [ ] |

### 네트워크 QA

| # | 테스트 | 기대 결과 | 확인 |
|---|--------|---------|------|
| 1 | 오프라인 상태에서 저장 | "오프라인" 에러 메시지 | [ ] |
| 2 | 서버 다운 시 조회 | "서버 연결 불가" 메시지 | [ ] |
| 3 | 느린 네트워크에서 저장 | 로딩 인디케이터 표시 | [ ] |

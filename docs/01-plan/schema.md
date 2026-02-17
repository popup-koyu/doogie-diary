# DOOGIE - Data Schema (데이터 스키마)

> 모든 엔티티의 데이터 구조와 관계를 정의하는 문서

**버전**: 1.0
**최종 수정일**: 2026-02-18
**프로젝트**: DOOGIE Diary

---

## 1. 엔티티 목록

| # | 엔티티 | English | 저장소 | 설명 |
|---|--------|---------|--------|------|
| 1 | 사용자 | User | Bkend (자동) | 닉네임 + PIN 기반 계정 |
| 2 | 일기 | Entry | Bkend `entries` 테이블 | 텍스트 + 이미지 일기 항목 |
| 3 | 이미지 | Image | Entry 내 embedded | 픽셀화된 썸네일/원본 이미지 |
| 4 | 캐릭터 | Character | IndexedDB (로컬) | 크레딧 화면 프로필 |
| 5 | 설정 | Settings | localStorage (로컬) | 언어, 크리스마스 모드 |

---

## 2. 엔티티 상세 스키마

### 2.1 User (사용자)

> Bkend 내장 인증 시스템 사용. 별도 테이블 불필요.

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| _id | ObjectId | auto | Bkend 자동 생성 |
| email | String | Y | `{nickname}@doogie.app` (닉네임 변환) |
| password | String | Y | `Doo@gie{pin}#Pwd` (PIN 변환) |
| createdAt | Date | auto | 가입일시 |

**비즈니스 규칙**:
- 닉네임: 2~12자, 한글/영문/숫자 허용
- PIN: 숫자 4자리 (0000~9999)
- 닉네임 중복 불가 (=이메일 중복 불가)
- 비밀번호 찾기 미지원

**닉네임→이메일 변환 예시**:
```
닉네임: "홍길동" → email: "홍길동@doogie.app"
PIN: "1234" → password: "Doo@gie1234#Pwd"
```

---

### 2.2 Entry (일기)

> Bkend `entries` 테이블에 저장

```javascript
{
  _id: ObjectId,              // Bkend 자동 생성
  createdBy: ObjectId,        // 작성자 ID (Bkend 자동 주입)
  dateKey: String,            // "YYYY-MM-DD" 형식
  entryNumber: Number,        // 같은 날 항목 번호 (1~100)
  type: String,               // "diary" | "markdown"
  text: String,               // 본문 내용
  images: Array<Image>,       // 이미지 배열 (0~N개)
  createdAt: Date,            // Bkend 자동 생성
  updatedAt: Date             // Bkend 자동 갱신
}
```

| 필드 | 타입 | 필수 | 기본값 | 제약 조건 |
|------|------|------|--------|----------|
| _id | ObjectId | auto | - | PK |
| createdBy | ObjectId | auto | - | FK → User._id |
| dateKey | String | Y | 오늘 날짜 | `/^\d{4}-\d{2}-\d{2}$/` |
| entryNumber | Number | Y | 자동 계산 | 1~100, 정수 |
| type | String | Y | "diary" | `"diary"` \| `"markdown"` |
| text | String | N | "" | 제한 없음 |
| images | Array | N | [] | Image 객체 배열 |
| createdAt | Date | auto | now | - |
| updatedAt | Date | auto | now | 수정 시 자동 갱신 |

**인덱스**:

| 인덱스 이름 | 필드 | 타입 | 용도 |
|------------|------|------|------|
| dateKey_entryNumber_unique | { createdBy, dateKey, entryNumber } | Unique | 중복 저장 방지 |
| dateKey_idx | { dateKey } | Normal | 날짜별 조회 |
| createdBy_idx | { createdBy } | Normal | 사용자별 조회 |

**비즈니스 규칙**:
- 하루 최대 100개 (diary + markdown 합산)
- type 변경 불가 (diary ↔ markdown 전환 불가)
- dateKey, entryNumber 수정 불가
- 본인 일기만 조회/수정/삭제 가능 (self role)

---

### 2.3 Image (이미지)

> Entry.images[] 내부에 embedded 저장

```javascript
{
  thumbnail: String,   // Base64 Data URL (600px 제한)
  full: String         // Base64 Data URL (1200px 제한)
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| thumbnail | String | Y | 600px 이하 픽셀화 이미지 (Base64) |
| full | String | Y | 1200px 이하 픽셀화 이미지 (Base64) |

**이미지 처리 파이프라인**:
```
원본 이미지 → Canvas 로드 → 3px 단위 다운스케일 → 녹색 5단계 팔레트 적용
                                ↓                         ↓
                         Thumbnail (600px)          Full (1200px)
                                ↓                         ↓
                          Base64 인코딩            Base64 인코딩
```

**녹색 팔레트**:
| 단계 | 밝기 범위 | 색상값 |
|------|----------|--------|
| 0 | 0~42 | `rgb(0, 0, 0)` |
| 1 | 43~84 | `rgb(0, 51, 0)` |
| 2 | 85~126 | `rgb(0, 102, 0)` |
| 3 | 127~169 | `rgb(0, 153, 0)` |
| 4 | 170~212 | `rgb(0, 204, 0)` |
| 5 | 213~255 | `rgb(0, 255, 0)` |

---

### 2.4 Character (캐릭터)

> IndexedDB `characters` store에 로컬 저장

```javascript
{
  slotIndex: Number,     // 0~5 (6개 슬롯)
  name: String,          // 캐릭터 이름
  role: String,          // 캐릭터 역할
  imageData: String      // 픽셀화된 이미지 (Base64)
}
```

| 필드 | 타입 | 필수 | 기본값 | 설명 |
|------|------|------|--------|------|
| slotIndex | Number | Y | - | 슬롯 위치 (0~5) |
| name | String | N | "NAME" | 캐릭터 이름 |
| role | String | N | "ROLE" | 캐릭터 역할 |
| imageData | String | N | null | 크롭+픽셀화된 이미지 (Base64) |

---

### 2.5 Settings (설정)

> localStorage에 키-값 형태로 저장

| 키 | 타입 | 기본값 | 설명 |
|----|------|--------|------|
| `doogie_language` | String | "en" | 언어 설정 ("ko" \| "en") |
| `doogie_christmas` | Boolean | auto | 크리스마스 모드 (12월 자동 활성화) |
| `doogie_token` | String | null | JWT Access Token (24시간 유효) |

---

## 3. 권한 모델 (RBAC)

### entries 테이블 권한

| Role | Create | Read | Update | Delete | 설명 |
|------|--------|------|--------|--------|------|
| admin | O | O | O | O | 관리자 (전체 접근) |
| user | O | O | O | O | 로그인 사용자 |
| self | - | O | O | O | 본인 데이터만 |
| guest | - | X | - | - | 비로그인 (접근 불가) |

**핵심**: `self` 역할에 의해 본인이 작성한(createdBy) 일기만 조회/수정/삭제 가능

---

## 4. API 엔드포인트 매핑

### 인증 API

| 동작 | Method | Endpoint | Request Body |
|------|--------|----------|--------------|
| 회원가입 | POST | `/auth/signup/password` | `{ email, password }` |
| 로그인 | POST | `/auth/signin/password` | `{ email, password }` |
| 내 정보 | GET | `/auth/me` | - |

### 일기 CRUD API

| 동작 | Method | Endpoint | 설명 |
|------|--------|----------|------|
| 목록 조회 | GET | `/data/entries` | 필터/정렬/페이지네이션 |
| 상세 조회 | GET | `/data/entries/{id}` | 단건 조회 |
| 생성 | POST | `/data/entries` | 새 일기 저장 |
| 수정 | PUT | `/data/entries/{id}` | 텍스트/이미지 수정 |
| 삭제 | DELETE | `/data/entries/{id}` | 즉시 삭제 (복구 불가) |

### 필수 헤더

```
Authorization: Bearer {access_token}
X-Project-Id: {project_id}
X-Environment: dev
```

### 주요 쿼리 패턴

```javascript
// 특정 날짜 일기 조회
GET /data/entries?andFilters={"dateKey":"2025-12-18"}

// 특정 월 일기 조회 (달력 뷰)
GET /data/entries?andFilters={"dateKey":{"$regex":"^2025-12"}}

// 마크다운 일기만 조회
GET /data/entries?andFilters={"type":"markdown"}

// 최신순 정렬
GET /data/entries?sort={"dateKey":-1,"entryNumber":-1}
```

---

## 5. 데이터 흐름도

### 일기 작성 플로우
```
[사용자 입력] → [자동저장 debounce 2초] → [Bkend API POST/PUT]
                                              ↓
                                     [중복 체크 (Unique Index)]
                                         ↓ 실패 시
                                     [entryNumber 재조회 → 재시도 (최대 3회)]
```

### 인증 플로우
```
[닉네임 + PIN 입력] → [닉네임→이메일, PIN→패스워드 변환]
                          ↓
                    [Bkend Auth API]
                          ↓
                    [JWT Token 발급]
                          ↓
                    [localStorage 저장]
                          ↓
                    [API 요청 시 Authorization 헤더 첨부]
```

### 이미지 처리 플로우
```
[파일 선택/드래그] → [FileReader API 로드] → [Canvas 렌더링]
                                                  ↓
                                          [3px 다운스케일]
                                                  ↓
                                          [녹색 팔레트 매핑]
                                             ↓         ↓
                                      [Thumbnail]  [Full]
                                        (600px)    (1200px)
                                             ↓         ↓
                                          [Base64 인코딩]
                                                  ↓
                                          [Entry.images[]에 저장]
```

# DOOGIE - Domain Model (도메인 모델)

> 엔티티 간 관계와 시스템 경계를 정의하는 문서

**버전**: 1.0
**최종 수정일**: 2026-02-18
**프로젝트**: DOOGIE Diary

---

## 1. 엔티티 관계도 (ERD)

```mermaid
erDiagram
    USER ||--o{ ENTRY : "작성 (createdBy)"
    ENTRY ||--o{ IMAGE : "포함 (embedded)"

    USER {
        ObjectId _id PK
        String email "nickname@doogie.app"
        String password "Doo@gie{pin}#Pwd"
        Date createdAt
    }

    ENTRY {
        ObjectId _id PK
        ObjectId createdBy FK
        String dateKey "YYYY-MM-DD"
        Number entryNumber "1~100"
        String type "diary|markdown"
        String text
        Array images
        Date createdAt
        Date updatedAt
    }

    IMAGE {
        String thumbnail "Base64 600px"
        String full "Base64 1200px"
    }

    CHARACTER {
        Number slotIndex PK "0~5"
        String name
        String role
        String imageData "Base64"
    }

    SETTINGS {
        String language "ko|en"
        Boolean christmas
        String token "JWT"
    }
```

---

## 2. 시스템 경계 (Bounded Context)

### 2.1 컨텍스트 맵

```
┌─────────────────────────────────────────────────────────┐
│                    DOOGIE Application                     │
│                                                           │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  인증 컨텍스트  │  │  일기 컨텍스트  │  │ 프레젠테이션    │  │
│  │  (Auth)       │  │  (Diary)      │  │ (Presentation) │  │
│  │               │  │               │  │                │  │
│  │ - 회원가입     │  │ - CRUD        │  │ - 크레딧 화면   │  │
│  │ - 로그인      │  │ - 이미지 처리   │  │ - 캐릭터 슬롯   │  │
│  │ - 세션 관리    │  │ - 마크다운 모드  │  │ - NYC 스카이라인 │  │
│  │ - 닉네임 변환  │  │ - 달력 뷰      │  │ - 자동차 애니    │  │
│  │               │  │ - 자동 저장     │  │                │  │
│  └───────┬───────┘  └───────┬───────┘  └───────┬────────┘  │
│          │                  │                   │           │
│          ▼                  ▼                   ▼           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                   공통 인프라 (Shared)                   │  │
│  │  - 설정 관리 (언어, 크리스마스 모드)                       │  │
│  │  - UI 테마 (DOS 녹색 디자인 시스템)                       │  │
│  │  - 반응형 레이아웃 (4:3 비율)                             │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                              │
                              ▼
                 ┌──────────────────────┐
                 │   External Services   │
                 │                      │
                 │  - Bkend API Server  │
                 │  - CDN (폰트/라이브러리)│
                 └──────────────────────┘
```

### 2.2 컨텍스트별 책임

| 컨텍스트 | 책임 | 저장소 | 외부 의존성 |
|----------|------|--------|------------|
| **Auth** | 사용자 인증, 세션 유지 | Bkend Auth + localStorage(token) | Bkend Auth API |
| **Diary** | 일기 CRUD, 이미지 처리, 마크다운 | Bkend entries 테이블 | Bkend Data API, Marked.js, Mermaid.js |
| **Presentation** | 크레딧 화면, 캐릭터 관리 | IndexedDB (로컬) | gif.js, html2canvas |
| **Shared** | 설정, 테마, 레이아웃 | localStorage | Google Fonts |

---

## 3. 도메인 규칙 (Domain Rules)

### 3.1 일기 작성 규칙

| # | 규칙 | 검증 위치 |
|---|------|----------|
| R1 | 로그인한 사용자만 일기 작성 가능 | Frontend + Backend |
| R2 | 하루 최대 100개 (diary + markdown 합산) | Frontend + Backend |
| R3 | entryNumber는 자동 계산 (기존 최대값 + 1) | Frontend |
| R4 | 같은 (createdBy, dateKey, entryNumber) 조합은 유일 | Backend (Unique Index) |
| R5 | type 변경 불가 (diary ↔ markdown) | Frontend |
| R6 | 텍스트 미변경 시 저장 스킵 | Frontend |
| R7 | 입력 후 2초 뒤 자동 저장 (debounce) | Frontend |

### 3.2 인증 규칙

| # | 규칙 | 검증 위치 |
|---|------|----------|
| A1 | 닉네임 2~12자 (한글/영문/숫자) | Frontend |
| A2 | PIN 숫자 4자리 (0000~9999) | Frontend |
| A3 | 닉네임 중복 불가 | Backend |
| A4 | JWT 토큰 24시간 유효 | Backend |
| A5 | 토큰 만료 시 자동 로그아웃 | Frontend |

### 3.3 이미지 규칙

| # | 규칙 | 검증 위치 |
|---|------|----------|
| I1 | 모든 이미지는 픽셀화 처리 (3px 단위) | Frontend |
| I2 | 녹색 5단계 팔레트만 사용 | Frontend |
| I3 | Thumbnail: 600px 이하, Full: 1200px 이하 | Frontend |
| I4 | Base64 Data URL 형식으로 저장 | Frontend |

### 3.4 접근 권한 규칙

| # | 규칙 | 설명 |
|---|------|------|
| P1 | 비로그인 사용자는 일기 접근 불가 | guest: read=false |
| P2 | 본인 일기만 조회/수정/삭제 가능 | self role |
| P3 | 관리자는 모든 일기 접근 가능 | admin role |

---

## 4. 화면 흐름도 (Screen Flow)

```mermaid
flowchart TD
    A[타이틀 화면] --> B{로그인 상태?}

    B -->|비로그인| C[인증 화면]
    C -->|회원가입| C
    C -->|로그인 성공| A

    B -->|로그인| D[일기쓰기 Diary]
    B -->|로그인| E[나의 일기장 Memory]
    B -->|로그인/비로그인| F[디스켓 Diskette]

    D --> G[일기 작성 화면]
    G -->|Save| G
    G -->|New| G
    G -->|Picture| H[이미지 관리]
    G -->|List| I[일기 목록 드롭다운]
    G -->|Help| J[도움말]
    G -->|Fullscreen| K[전체화면]
    G -->|DIARY/MARKDOWN| L[모드 전환]
    G -->|Home| A

    H -->|Add Picture| H
    H -->|View Images| M[이미지 모달]

    I -->|항목 선택| G

    E --> N[달력 뷰]
    N -->|날짜 클릭| G
    N -->|리스트 항목 클릭| G
    N -->|Settings| O[설정 화면]
    N -->|◀ ▶| N

    O -->|언어 변경| O
    O -->|크리스마스 모드| O

    F --> P[크레딧 화면]
    P -->|캐릭터 업로드| Q[크롭 에디터]
    Q -->|Crop & Apply| P

    A -->|12월| R[🎄 크리스마스 카드]
    R -->|GIF 다운로드| R
    R -->|PNG 다운로드| R
```

---

## 5. 상태 다이어그램

### 5.1 일기 항목 상태

```mermaid
stateDiagram-v2
    [*] --> 새_일기: New 버튼 클릭

    새_일기 --> 작성중: 텍스트 입력
    작성중 --> 저장대기: 2초 경과 (debounce)
    저장대기 --> 저장완료: API 성공
    저장대기 --> 저장실패: API 오류
    저장실패 --> 작성중: 재시도

    저장완료 --> 작성중: 텍스트 수정
    저장완료 --> [*]: Home / 다른 일기 선택

    작성중 --> 저장확인: Home / New 클릭 (미저장 시)
    저장확인 --> 저장대기: Yes
    저장확인 --> [*]: No
```

### 5.2 인증 상태

```mermaid
stateDiagram-v2
    [*] --> 비로그인: 앱 시작

    비로그인 --> 로그인중: 로그인 시도
    로그인중 --> 로그인됨: JWT 토큰 발급
    로그인중 --> 비로그인: 인증 실패

    로그인됨 --> 비로그인: 로그아웃
    로그인됨 --> 비로그인: 토큰 만료 (24시간)

    비로그인 --> 회원가입중: 회원가입 시도
    회원가입중 --> 로그인됨: 가입 성공 → 자동 로그인
    회원가입중 --> 비로그인: 가입 실패 (닉네임 중복 등)
```

---

## 6. 배포 아키텍처

```
┌─────────────────────────────────────┐
│           Client (Browser)           │
│                                     │
│  ┌───────────────────────────────┐  │
│  │   index.html (Single File)    │  │
│  │   - HTML + CSS + JavaScript   │  │
│  │   - Canvas API (이미지 처리)    │  │
│  │   - Web Audio API (BGM)       │  │
│  └──────────────┬────────────────┘  │
│                 │                    │
│  ┌──────────────┴────────────────┐  │
│  │      Local Storage            │  │
│  │  - localStorage (설정, 토큰)   │  │
│  │  - IndexedDB (캐릭터)          │  │
│  └───────────────────────────────┘  │
└─────────────────┬───────────────────┘
                  │ HTTPS
                  ▼
┌─────────────────────────────────────┐
│         Vercel (Hosting)             │
│  - 정적 파일 서빙                      │
│  - API 프록시 (vercel.json)           │
└─────────────────┬───────────────────┘
                  │ HTTPS
                  ▼
┌─────────────────────────────────────┐
│      Bkend API Server                │
│  (api-enduser.bkend.ai)             │
│                                     │
│  ┌─────────────┐ ┌──────────────┐   │
│  │  Auth API    │ │  Data API    │   │
│  │  - signup    │ │  - entries   │   │
│  │  - signin    │ │    CRUD      │   │
│  │  - me        │ │              │   │
│  └─────────────┘ └──────────────┘   │
│                                     │
│  ┌───────────────────────────────┐  │
│  │       MongoDB (Storage)        │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

---

## 7. 외부 라이브러리 의존성

| 라이브러리 | 버전 | 용도 | 로드 방식 |
|-----------|------|------|----------|
| Marked.js | latest | 마크다운 파싱 (GFM) | CDN |
| Mermaid.js | latest | 다이어그램 렌더링 | CDN |
| gif.js | latest | GIF 생성 (크리스마스 카드) | CDN |
| html2canvas | latest | PNG 캡처 (크리스마스 카드) | CDN |
| NeoDunggeunmo | - | 한글 폰트 | Google Fonts |
| VT323 | - | 영문 폰트 | Google Fonts |
| Press Start 2P | - | 타이틀 폰트 | Google Fonts |
| Vite | 6.x | 개발 서버/빌드 | npm |

# DOOGIE - Glossary (용어집)

> 기획자와 개발자 간 동일한 용어로 소통하기 위한 문서

**버전**: 1.0
**최종 수정일**: 2026-02-18
**프로젝트**: DOOGIE Diary

---

## 1. 비즈니스 용어 (Business Terms)

| 용어 | English | 정의 | 글로벌 표준 매핑 |
|------|---------|------|-----------------|
| 두기 | DOOGIE | 1990년대 DOS 감성의 웹 기반 일기장 애플리케이션 | Web Application |
| 일기 | Entry | 사용자가 작성한 하나의 일기 항목 (텍스트 + 이미지) | Document, Record |
| 일기장 | Memory | 저장된 일기 목록을 보는 화면 (달력 뷰 포함) | Archive, Journal |
| 디스켓 | Diskette | 크레딧 화면 (제작진 정보, NYC 스카이라인 등) | Credits Screen |
| 항목 번호 | Entry Number | 같은 날짜에 여러 일기를 구분하는 번호 (1~100) | Sequence Number |
| 날짜 키 | Date Key | 일기를 식별하는 날짜 문자열 (`YYYY-MM-DD`) | Date Identifier |
| 픽셀화 | Pixelation | 이미지를 3px 단위 녹색 팔레트로 변환하는 처리 | Image Processing |
| 스캔 효과 | Scan Effect | CRT 모니터를 재현한 위→아래 스캔라인 애니메이션 | CRT Animation |
| 닉네임 | Nickname | 사용자 식별명 (2-12자, 한글/영문/숫자) | Username |
| PIN | PIN | 4자리 숫자 비밀번호 (0000-9999) | Password |
| 마크다운 모드 | Markdown Mode | GFM 마크다운으로 일기를 작성하는 모드 | Editor Mode |
| 일반 모드 | Diary Mode | 기본 텍스트로 일기를 작성하는 모드 | Editor Mode |
| 크리스마스 모드 | Christmas Mode | 12월 한정 시각 효과 (눈, BGM, 카드 생성) | Seasonal Feature |
| 캐릭터 슬롯 | Character Slot | 크레딧 화면의 팀원 프로필 영역 (6개, 3x2 그리드) | Profile Card |

## 2. 화면/뷰 용어 (Screen Terms)

| 용어 | English | 정의 | 진입 경로 |
|------|---------|------|----------|
| 타이틀 화면 | Title Screen | 앱 최초 진입 화면 (메뉴 선택) | 앱 시작 |
| 일기 작성 화면 | Editor Screen | 일기를 작성/편집하는 메인 화면 | 일기쓰기 메뉴 |
| 달력 뷰 | Calendar View | 월간 달력 + 일기 리스트 (좌우 분할) | 나의 일기장 메뉴 |
| 인증 화면 | Auth Screen | 로그인/회원가입 화면 | 타이틀 화면 버튼 |
| 도움말 화면 | Help Screen | 단축키 및 사용법 안내 | F1 또는 Help 메뉴 |
| 설정 화면 | Settings Screen | 언어, 크리스마스 모드 설정 | 달력 뷰 하단 버튼 |
| 이미지 모달 | Image Modal | 이미지 전체 화면 보기 (스캔 효과) | View Images 메뉴 |
| 크리스마스 카드 | Christmas Card | 커스텀 카드 생성 화면 (12월 한정) | 타이틀 화면 🎄 메뉴 |

## 3. 기술 용어 (Technical Terms)

### 3.1 글로벌 표준

| 용어 | 정의 | 참조 |
|------|------|------|
| JWT | JSON Web Token, 인증 토큰 | RFC 7519 |
| REST API | RESTful API 아키텍처 | - |
| Base64 | 바이너리→텍스트 인코딩 (이미지 저장용) | RFC 4648 |
| GFM | GitHub Flavored Markdown | GitHub Spec |
| Canvas API | HTML5 그래픽 렌더링 API (픽셀 아트 변환) | W3C |
| Web Audio API | 브라우저 오디오 합성 API (BGM 생성) | W3C |
| Fullscreen API | 브라우저 전체화면 API | W3C |
| IndexedDB | 브라우저 로컬 저장소 (설정 저장용) | W3C |
| Mermaid | 마크다운 기반 다이어그램 라이브러리 | mermaid.js |

### 3.2 프로젝트 기술 용어

| 용어 | English | 정의 |
|------|---------|------|
| Bkend | Bkend | BaaS 플랫폼 (백엔드 서비스) |
| entries 테이블 | entries table | 일기 데이터를 저장하는 Bkend 테이블 |
| 썸네일 | Thumbnail | 600px 제한 축소 이미지 (일기 보기용) |
| Full 이미지 | Full Image | 1200px 제한 원본 이미지 (모달용) |
| 녹색 팔레트 | Green Palette | 픽셀화에 사용되는 6단계 녹색 색상 세트 |
| 자동 저장 | Auto Save | 입력 후 2초 지연 저장 (debounce) |
| 닉네임→이메일 변환 | Nickname-to-Email | `{nickname}@doogie.app` 형식 변환 |
| PIN→패스워드 변환 | PIN-to-Password | `Doo@gie{pin}#Pwd` 형식 변환 |

## 4. 용어 사용 규칙

### 코드에서의 사용

| 구분 | 규칙 | 예시 |
|------|------|------|
| 변수/함수명 | 영문 camelCase | `dateKey`, `entryNumber`, `saveEntry()` |
| API 필드 | 영문 camelCase | `{ dateKey, entryNumber, type }` |
| CSS 클래스 | kebab-case | `.diary-editor`, `.image-modal` |
| 상수 | UPPER_SNAKE_CASE | `MAX_ENTRIES_PER_DAY`, `PIXEL_SIZE` |

### UI/문서에서의 사용

| 구분 | 한국어 | English |
|------|--------|---------|
| 메뉴 | 일기쓰기, 나의 일기장, 디스켓 | Diary, Memory, Diskette |
| 버튼 | 저장, 새 일기, 목록 | Save, New, List |
| 메시지 | "저장되었습니다!" | "Saved!" |
| 에러 | "서버에 연결할 수 없습니다" | "Cannot connect to server" |

### 혼동 주의 용어

| 혼동 가능 | 올바른 용어 | 설명 |
|-----------|------------|------|
| 일기 vs 항목 | **Entry** (일기) | 하나의 완성된 일기 단위 |
| 비밀번호 vs PIN | **PIN** | 4자리 숫자만 허용 |
| 회원 vs 사용자 | **사용자 (User)** | Bkend의 User 모델과 매핑 |
| 저장소 vs 데이터베이스 | **Bkend 서버** | 클라우드 저장소 (MongoDB 기반) |
| 로컬 저장 vs 서버 저장 | **서버 저장** (일기), **로컬 저장** (설정) | 일기=서버, 설정=localStorage |

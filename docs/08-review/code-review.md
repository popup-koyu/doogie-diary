# DOOGIE - 코드 리뷰 및 갭 분석 (Phase 8)

> 아키텍처 일관성, 컨벤션 준수, 코드 품질 검증

**버전**: 1.0
**최종 수정일**: 2026-02-18
**프로젝트**: DOOGIE Diary
**대상 파일**: index.html (~7,200줄)

---

## 1. 리뷰 요약

### 1.1 종합 등급

```
┌──────────────────────┬────────┬───────────────────────────┐
│ 카테고리              │ 등급    │ 요약                       │
├──────────────────────┼────────┼───────────────────────────┤
│ 컨벤션 준수           │ B+     │ 대부분 준수, 일부 불일치     │
│ 아키텍처 일관성        │ B      │ 레이어 순서 OK, 캡슐화 미흡  │
│ 코드 품질             │ C+     │ 중복 코드, 긴 함수 존재      │
│ 기능 완성도           │ A      │ 모든 기획 기능 구현 완료      │
│ API 연동              │ A      │ 모든 명세 엔드포인트 구현     │
│ 보안                  │ B      │ 기본 안전, 개선 여지 있음     │
│ 잠재 버그             │ C+     │ 레이스 컨디션, 메모리 누수    │
└──────────────────────┴────────┴───────────────────────────┘

종합: B (양호 - 안정적으로 동작하나 리팩토링 권장)
```

---

## 2. 컨벤션 준수 분석

### 2.1 네이밍 규칙 (docs/02-convention/naming.md 기준)

| 규칙 | 준수 | 위반 사항 |
|------|------|----------|
| 변수/함수: camelCase | ✅ | `currentDate`, `saveEntry`, `handleAuthSubmit` |
| CSS 클래스: kebab-case | ✅ | `.diary-editor`, `.menu-bar`, `.image-modal` |
| 상수: UPPER_SNAKE_CASE | ✅ | `USE_SERVER_MODE`, `PROJECT_ID` |
| 접두사: show/hide | ✅ | `showAuthScreen()`, `hideLoading()` |
| 접두사: handle | ✅ | `handleAuthSubmit()`, `handleMenuAction()` |
| 접두사: setup | ✅ | `setupAuthListeners()`, `setupCalendar()` |

### 2.2 발견된 불일치

```
1. 상태 변수 접두사 불일치
   ✅ currentDate, currentEntryNumber, currentEntryId
   ❌ entries (→ currentEntries 또는 entryMap이 명확)
   ❌ autoSaveTimeout (→ autoSaveTimeoutId가 명확)

2. 유사 함수 네이밍 혼란
   saveEntries() vs saveEntrySingle()
   parseEntryKey() vs parseLocalKey()  ← 같은 역할, 두 함수 존재

3. CSS 색상 매직 넘버
   JS 코드 내 #00xx00 계열 색상이 상수 없이 직접 사용
   → DOS_COLORS 상수 객체로 추출 권장
```

---

## 3. 아키텍처 분석

### 3.1 레이어 순서 검증

```
규칙 (structure.md):
  상수 → BkendAPI → DiaryDB → 기능 함수 → init()

실제 코드:
  Line 3175    상수 (USE_SERVER_MODE 등)     ✅
  Line 3180    class BkendAPI                ✅
  Line 3454    class DiaryDB                 ✅
  Line 3663    전역 상태 변수                  ⚠️ (별도 영역)
  Line 3761+   기능 함수들                    ✅
  Line 4888    init()                        ✅

결론: 레이어 순서 준수 ✅
```

### 3.2 레이어 위반 사항

```
위반 유형 1: 기능 함수에서 전역 entries 직접 조작
──────────────────────────────────────────────
  entries[key] = { text, images };     ← 여러 함수에서 직접 접근
  delete entries[key];                  ← 삭제도 직접 수행

  위치: addImageToEntry(), saveCurrentEntry(), loadEntries(),
       deleteEntrySingle(), proceedWithNewEntry() 등 10+ 곳

  문제: 데이터 레이어(DiaryDB) 우회, 캡슐화 위반
  권장: 데이터 접근 함수 통합 (getEntry, setEntry, deleteEntry)

위반 유형 2: BkendAPI 직접 호출
──────────────────────────────────────────────
  await bkendAPI.signin()              ← handleAuthSubmit()에서 직접
  await bkendAPI.getAllEntries()        ← loadEntries()에서 직접
  await bkendAPI.updateEntry()          ← saveEntrySingle()에서 직접

  문제: 중간 추상화 레이어 없음 (DiaryDB가 서버 연동을 담당해야 함)
  영향: DiaryDB는 IndexedDB 전용이 되어, 서버/로컬 분기가 기능 함수에 노출됨
```

### 3.3 아키텍처 개선 방향

```
현재:
  기능 함수 → BkendAPI (서버)
  기능 함수 → DiaryDB (로컬)
  기능 함수 → entries 전역 객체 (직접)

권장:
  기능 함수 → DataManager(통합 레이어)
              ├── Server Mode → BkendAPI
              └── Local Mode  → DiaryDB

  장점: 기능 함수에서 모드 분기 코드 제거
        데이터 접근 일관성 확보
```

---

## 4. 코드 품질 분석

### 4.1 긴 함수 목록 (>50줄)

| 함수 | 줄 수 | 위치 | 문제점 |
|------|-------|------|--------|
| `createCars()` | ~111 | Line 4180 | 중첩 함수 + setInterval |
| `handleListContinuation()` | ~106 | Line 4779 | **동일 로직 3회 반복** |
| `createNYCSkyline()` | ~106 | Line 3784 | 하드코딩된 건물 설정 |
| `setupCropDrag()` | ~101 | Line 4399 | 6단계 중첩 |
| `loadCharacterPortraits()` | ~75 | Line 4545 | Canvas 중첩 루프 |
| `handleAuthSubmit()` | ~70 | Line 5173 | 중첩 조건문 |
| `handleAutoFormat()` | ~69 | Line 4707 | 유사 패턴 반복 |
| `setupMarkdownMode()` | ~65 | Line 5321 | 이벤트 + 복잡 로직 |
| `init()` | ~56 | Line 4888 | 초기화 함수 (허용 범위) |

### 4.2 중복 코드

```
심각도: 높음
──────────────────────────────────────────

1. handleListContinuation() - 동일 로직 3회 반복
   ├── 불릿 리스트 처리  (Lines 4793-4821)
   ├── 체크박스 리스트 처리 (Lines 4824-4852)
   └── 숫자 리스트 처리   (Lines 4855-4883)
   → 리스트 타입을 파라미터화하면 ~60줄 절약

2. IndexedDB 트랜잭션 패턴 - 8회 반복
   const tx = db.transaction([store], mode);
   const objStore = tx.objectStore(store);
   → 헬퍼 함수 추출 가능

심각도: 중간
──────────────────────────────────────────

3. Canvas 색상 생성 (createCars 내)
   Lines 4191-4196 vs 4223-4228 (동일 코드)
   → 유틸리티 함수 추출

4. 엔트리 키 파싱 함수 중복
   parseEntryKey() (Line 5923) vs parseLocalKey() (Line 6623)
   → 하나로 통합 필요
```

### 4.3 깊은 중첩 (>4단계)

```
위치: setupCropDrag() (Line 4399)
중첩: 최대 6단계

  addEventListener → if → else if → if → if → 처리
       L1              L2    L2      L3   L4    L5-6

권장: Early Return 패턴, 핸들 타입을 객체 맵으로 변환
```

### 4.4 미사용/죽은 코드

| 항목 | 위치 | 설명 |
|------|------|------|
| `selectedMenuItem` | Line 3671 | 선언 후 미사용 |
| `canCreateEntryToday()` | Line 3443 | BkendAPI 메서드이나 직접 호출 거의 없음 |
| `starsContainer` 중복 참조 | Line 3715 | 직접 `getElementById`도 사용 |

---

## 5. 갭 분석 (설계 vs 구현)

### 5.1 스키마 갭 (docs/01-plan/schema.md)

```
Entry 스키마 필드     │ 코드 구현              │ 상태
─────────────────────┼───────────────────────┼──────
_id                  │ entry._id             │ ✅
createdBy            │ 서버 자동 설정          │ ✅
dateKey              │ entry.dateKey          │ ✅
entryNumber          │ entry.entryNumber      │ ✅
type                 │ entry.type             │ ✅
text                 │ entry.text             │ ✅
images               │ entry.images[]         │ ✅
  ├ thumbnail        │ images[].thumbnail     │ ✅
  └ full             │ images[].full          │ ✅
createdAt            │ 서버 자동 설정          │ ✅
updatedAt            │ 서버 자동 설정          │ ✅

결론: 스키마 일치율 100% ✅
```

### 5.2 화면 갭 (docs/03-mockup/mockup-spec.md)

```
명세 화면              │ HTML 구현               │ 상태
─────────────────────┼────────────────────────┼──────
타이틀 화면            │ #titleScreen            │ ✅
인증 화면              │ #auth-screen            │ ✅
일기 에디터            │ #diary-screen           │ ✅
마크다운 에디터         │ #markdown-editor-area   │ ✅
달력 뷰               │ #calendar-screen        │ ✅
크레딧 화면            │ #credits-screen         │ ✅
도움말 화면            │ #help-screen            │ ✅
설정 화면              │ #settings-screen        │ ✅
크리스마스 카드         │ #christmas-card         │ ✅
이미지 모달            │ #imageModal             │ ✅
확인 다이얼로그         │ #confirmDialog          │ ✅

결론: 화면 일치율 100% ✅
```

### 5.3 API 갭 (docs/04-api/api-spec.md)

```
명세 API               │ BkendAPI 메서드          │ 상태
─────────────────────┼────────────────────────┼──────
POST /auth/signup     │ signup()               │ ✅
POST /auth/signin     │ signin()               │ ✅
GET /auth/me          │ fetchCurrentUser()     │ ✅
GET /data/entries     │ getAllEntries()         │ ✅
GET entries (날짜별)   │ getEntriesByDate()     │ ✅
GET entries (월별)     │ getEntriesByMonth()    │ ✅
POST /data/entries    │ createEntry()          │ ✅
PUT /data/entries/:id │ updateEntry()          │ ✅
DELETE /data/entries   │ deleteEntry()          │ ✅

미문서화 메서드 (코드에만 존재):
  ⚠️ checkNicknameAvailable()  → API 명세에 추가 권장
  ⚠️ getTodayCount()           → API 명세에 추가 권장
  ⚠️ getNextEntryNumber()      → API 명세에 추가 권장
  ⚠️ canCreateEntryToday()     → API 명세에 추가 권장

결론: 핵심 API 일치율 100% ✅
      미문서화 메서드 4개 → 명세 업데이트 권장
```

### 5.4 갭 요약

```
┌──────────────────┬──────────┬──────────┬──────────┐
│ 비교 대상         │ 일치      │ 미구현    │ 미문서화  │
├──────────────────┼──────────┼──────────┼──────────┤
│ 스키마 필드       │ 11/11    │ 0        │ 0        │
│ 화면             │ 11/11    │ 0        │ 0        │
│ API 엔드포인트    │ 9/9      │ 0        │ 4        │
├──────────────────┼──────────┼──────────┼──────────┤
│ 총합             │ 31/31    │ 0        │ 4        │
│ 일치율            │ 100%     │          │          │
└──────────────────┴──────────┴──────────┴──────────┘
```

---

## 6. 잠재 버그 분석

### 6.1 에러 핸들링 미흡

```
위험도: 중간

1. init() 함수 - try-catch 없음
   Line 4888: async function init() {
     await setupAuthListeners();    // 실패 시 앱 전체 중단
     await tryRestoreSession();     // 네트워크 오류 시 처리 없음
     await loadLanguage();          // 언어 파일 실패 시 처리 없음
   }

   영향: 초기화 중 에러 발생 시 앱이 빈 화면으로 멈춤
   권장: try-catch + 사용자 안내 메시지

2. loadEntries() - 오프라인 시 빈 entries
   Line 6554: 서버 조회 실패 시 entries = {} 상태 유지
   영향: 오프라인에서 앱 접근 시 빈 달력
   권장: IndexedDB 캐시 폴백
```

### 6.2 레이스 컨디션

```
위험도: 중간

1. 자동 저장 타이밍
   상황: 사용자가 빠르게 엔트리를 전환
   문제: 이전 엔트리의 autoSaveTimeout이 새 엔트리 저장 시점에 실행
   영향: 잘못된 엔트리에 다른 내용이 저장될 수 있음

   현재 완화:
     clearTimeout(autoSaveTimeout)   ← 전환 시 취소

   남은 위험:
     이미 fetch 중인 저장 요청은 취소 불가
     → AbortController 도입 권장

2. 복수 이미지 동시 업로드
   상황: 여러 이미지 선택 후 순차 처리
   문제: pixelateImage()가 공유 UI 상태를 변경
   영향: 이미지 순서 꼬임 가능성 낮으나 존재
```

### 6.3 메모리 누수 위험

```
위험도: 중간

1. setInterval 미정리 (크레딧 화면)
   Line 3879: setInterval(() => { /* 창문 애니메이션 */ }, 250)
   Line 4259: setInterval(() => { /* 자동차 생성 */ }, ...)
   문제: 크레딧 화면을 떠나도 인터벌이 계속 실행
   권장: switchMode()에서 clearInterval() 호출

2. 이벤트 리스너 누적 (크롭 모달)
   Line 4399: setupCropDrag() - document에 mousemove/mouseup 추가
   문제: 크롭 모달 열 때마다 새 리스너 추가, 기존 리스너 미제거
   권장: 모달 닫을 때 removeEventListener() 호출

3. Base64 이미지 메모리
   문제: 대용량 이미지 Base64 변환 시 메모리 급증
   완화: 픽셀화로 크기 축소 (3px 해상도)
   권장: 변환 전 파일 크기 5MB 제한 추가
```

### 6.4 타입 불일치

```
위험도: 낮음

1. entries[key] 타입 혼재
   Line 6167: typeof entries[key] === 'string' 체크
   문제: 레거시 호환을 위해 string/object 혼재
   영향: 새 코드에서 타입 가정이 깨질 수 있음
   권장: 마이그레이션 완료 후 string 지원 제거

2. 에러 메시지 문자열 비교
   Line 5221: if (message === 'OFFLINE')
   문제: 상수 대신 문자열 리터럴 비교 (오타 위험)
   권장: ERROR_CODES 상수 객체 사용
```

---

## 7. 리팩토링 권장 목록

### 7.1 우선순위: 긴급 (배포 전)

| # | 항목 | 영향 | 난이도 |
|---|------|------|--------|
| 1 | `init()` try-catch 추가 | 앱 안정성 | ⭐ |
| 2 | 크레딧 화면 setInterval 정리 | 메모리 누수 | ⭐ |
| 3 | 크롭 모달 이벤트 리스너 정리 | 메모리 누수 | ⭐⭐ |

### 7.2 우선순위: 높음 (다음 스프린트)

| # | 항목 | 영향 | 난이도 |
|---|------|------|--------|
| 4 | `handleListContinuation()` 중복 제거 | 유지보수성 | ⭐⭐ |
| 5 | `parseEntryKey()` / `parseLocalKey()` 통합 | 일관성 | ⭐ |
| 6 | IndexedDB 트랜잭션 헬퍼 추출 | 중복 제거 | ⭐⭐ |
| 7 | 에러 코드 상수화 (문자열 비교 제거) | 안정성 | ⭐ |

### 7.3 우선순위: 중간 (향후)

| # | 항목 | 영향 | 난이도 |
|---|------|------|--------|
| 8 | entries 직접 접근 → 데이터 레이어 추상화 | 아키텍처 | ⭐⭐⭐ |
| 9 | `setupCropDrag()` 리팩토링 (중첩 축소) | 가독성 | ⭐⭐ |
| 10 | 이미지 파일 크기 제한 추가 | 안정성 | ⭐ |
| 11 | AutoSave에 AbortController 도입 | 레이스 컨디션 | ⭐⭐ |

### 7.4 우선순위: 낮음 (백로그)

| # | 항목 | 영향 | 난이도 |
|---|------|------|--------|
| 12 | `createNYCSkyline()` 설정 데이터 분리 | 가독성 | ⭐ |
| 13 | `createCars()` 유틸리티 함수 추출 | 중복 제거 | ⭐ |
| 14 | 미사용 변수 정리 | 코드 청결 | ⭐ |
| 15 | string 타입 entries 레거시 지원 제거 | 타입 안전 | ⭐ |

---

## 8. API 명세 업데이트 권장

```
현재 docs/04-api/api-spec.md에 누락된 메서드:

1. checkNicknameAvailable(nickname)
   - 방식: 더미 비밀번호로 signin 시도 → 결과 판별
   - 용도: 회원가입 시 닉네임 중복 체크
   - 반환: boolean

2. getTodayCount()
   - 엔드포인트: GET /data/entries (오늘 날짜 필터)
   - 용도: 오늘 작성한 일기 수 확인
   - 반환: number

3. getNextEntryNumber(dateKey)
   - 엔드포인트: GET /data/entries (날짜 필터, 정렬)
   - 용도: 새 엔트리 번호 산출
   - 반환: number

4. canCreateEntryToday()
   - 내부: getTodayCount() → 100 미만 체크
   - 용도: 하루 최대 엔트리 수 제한
   - 반환: boolean
```

---

## 9. 크로스 Phase 일관성 검증

### 9.1 Phase별 검증 결과

```
Phase 1 (스키마/용어) → Phase 8 검증
  ✅ glossary.md 용어가 코드에서 일관 사용
  ✅ schema.md 엔티티가 코드와 일치
  ✅ domain-model.md 화면 흐름이 실제와 일치

Phase 2 (컨벤션) → Phase 8 검증
  ✅ 네이밍 규칙 대부분 준수
  ⚠️ 일부 상태 변수 접두사 불일치
  ⚠️ CSS 색상 매직 넘버 존재

Phase 3 (목업) → Phase 8 검증
  ✅ 모든 화면 구현 완료 (11/11)
  ✅ 반응형 대응 구현

Phase 4 (API) → Phase 8 검증
  ✅ 모든 핵심 엔드포인트 구현 (9/9)
  ⚠️ 4개 메서드 문서 누락

Phase 5 (디자인 시스템) → Phase 8 검증
  ✅ DOS 녹색 테마 일관 적용
  ✅ 모노스페이스 폰트 적용
  ⚠️ 일부 CSS 색상값이 토큰 대신 직접 사용

Phase 6 (UI-API 연동) → Phase 8 검증
  ✅ 인증 흐름 정상 동작
  ✅ CRUD 흐름 정상 동작
  ⚠️ 데이터 접근 캡슐화 미흡

Phase 7 (SEO/보안) → Phase 8 검증
  ✅ textContent 기반 XSS 방어
  ✅ HTTPS + API 프록시
  ⚠️ 보안 헤더 미설정 (vercel.json)
  ⚠️ DOMPurify 미적용
```

### 9.2 일관성 점수

```
┌──────────────────┬──────────┬──────────┐
│ Phase            │ 일치 항목 │ 불일치    │
├──────────────────┼──────────┼──────────┤
│ Phase 1 스키마    │ 3/3      │ 0        │
│ Phase 2 컨벤션    │ 4/6      │ 2        │
│ Phase 3 목업      │ 11/11    │ 0        │
│ Phase 4 API      │ 9/9      │ 0 (+4)   │
│ Phase 5 디자인    │ 2/3      │ 1        │
│ Phase 6 연동      │ 2/3      │ 1        │
│ Phase 7 보안      │ 2/4      │ 2        │
├──────────────────┼──────────┼──────────┤
│ 총합             │ 33/39    │ 6        │
│ 일관성 점수       │ 85%      │          │
└──────────────────┴──────────┴──────────┘
```

---

## 10. 결론 및 권장사항

### 10.1 강점

```
✅ 기능 완성도 높음: 기획된 모든 화면과 API 100% 구현
✅ 레이어 순서 준수: Constants → BkendAPI → DiaryDB → Features → init()
✅ 네이밍 컨벤션 대부분 준수: camelCase, kebab-case, UPPER_SNAKE_CASE
✅ 에러 처리 패턴 확립: handleApiError() + showStatus()
✅ 하이브리드 저장: 서버/로컬 이중 모드 잘 구현
✅ 자동 저장: 2초 디바운스 + 중복 방지 잘 동작
```

### 10.2 개선 필요

```
⚠️ 긴 함수 7개 (50줄 초과) → 분리/추출 필요
⚠️ 중복 코드 4곳 → 유틸리티 함수화 필요
⚠️ 메모리 누수 위험 3곳 → 정리 함수 추가 필요
⚠️ 데이터 접근 캡슐화 → entries 직접 접근 추상화 필요
⚠️ API 명세 4개 메서드 누락 → 문서 업데이트 필요
⚠️ 보안 헤더 미설정 → vercel.json 업데이트 필요
```

### 10.3 배포 전 필수 조치

```
1. init() 함수에 try-catch 추가        (안정성)
2. setInterval 정리 로직 추가           (메모리)
3. 크롭 모달 이벤트 리스너 정리          (메모리)

위 3개 항목은 사용자 경험에 직접 영향을 미치므로
배포 전 수정 강력 권장
```

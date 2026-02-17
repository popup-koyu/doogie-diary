# DOOGIE - SEO / 보안 점검 명세 (Phase 7)

> 검색 최적화 및 보안 강화 점검

**버전**: 1.0
**최종 수정일**: 2026-02-18
**프로젝트**: DOOGIE Diary

---

## 1. 프로젝트 보안 특성

DOOGIE Diary는 **개인 일기장 앱**으로, 일반 웹 서비스와 보안 요구사항이 다릅니다.

```
┌──────────────────────────────────────────────────┐
│              DOOGIE 보안 컨텍스트                   │
├──────────────────────────────────────────────────┤
│ • 개인 사용자 대상 (불특정 다수 X)                   │
│ • 민감한 데이터: 일기 텍스트, 이미지                   │
│ • 인증: 닉네임 + 4자리 PIN (간편 인증)               │
│ • 외부 입력: 자기 자신의 일기 내용만                   │
│ • 배포: Vercel (HTTPS 자동 적용)                   │
│ • BaaS: Bkend (서버 보안은 BaaS가 관리)             │
└──────────────────────────────────────────────────┘
```

---

## 2. SEO 현황 분석

### 2.1 현재 상태

| 항목 | 상태 | 설명 |
|------|------|------|
| `<title>` | ✅ 있음 | `Doogie - Personal Diary` |
| `<meta charset>` | ✅ 있음 | `UTF-8` |
| `<meta viewport>` | ✅ 있음 | `width=device-width, initial-scale=1.0` |
| `<html lang>` | ✅ 있음 | `ko` |
| `<meta description>` | ❌ 없음 | 검색 결과 설명 미제공 |
| Open Graph 태그 | ❌ 없음 | SNS 공유 미리보기 없음 |
| Twitter Card | ❌ 없음 | 트위터 공유 미리보기 없음 |
| Canonical URL | ❌ 없음 | 중복 URL 처리 없음 |
| Favicon | ❌ 없음 | 브라우저 탭 아이콘 없음 |
| robots.txt | ❌ 없음 | 크롤러 가이드 없음 |
| sitemap.xml | ❌ 없음 | 사이트맵 없음 |
| 시맨틱 HTML | ❌ 미흡 | div 위주, `<main>`, `<nav>`, `<article>` 미사용 |
| 구조화된 데이터 | ❌ 없음 | JSON-LD / Schema.org 없음 |

### 2.2 권장 Meta 태그

```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Doogie - 1990s DOS Style Personal Diary</title>
    <meta name="description" content="두기 다이어리 - 1990년대 DOS 감성의 레트로 개인 일기장. 픽셀아트 이미지, 마크다운 지원.">

    <!-- Open Graph -->
    <meta property="og:type" content="website">
    <meta property="og:title" content="Doogie - 1990s DOS Style Personal Diary">
    <meta property="og:description" content="두기 다이어리 - 1990년대 DOS 감성의 레트로 개인 일기장">
    <meta property="og:image" content="/image/doogie.png">
    <meta property="og:url" content="https://doogie-diary.vercel.app">

    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:title" content="Doogie - 1990s DOS Style Personal Diary">
    <meta name="twitter:description" content="두기 다이어리 - 1990년대 DOS 감성의 레트로 개인 일기장">
    <meta name="twitter:image" content="/image/doogie.png">

    <!-- Favicon -->
    <link rel="icon" type="image/png" href="/image/doogie.png">

    <!-- Canonical -->
    <link rel="canonical" href="https://doogie-diary.vercel.app">
</head>
```

### 2.3 SEO 우선순위 (개인 일기장 기준)

| 우선순위 | 항목 | 이유 |
|----------|------|------|
| **높음** | Meta description | 검색 결과에 설명 표시 |
| **높음** | OG 태그 | SNS 공유 시 미리보기 |
| **높음** | Favicon | 브라우저 탭 식별 |
| **낮음** | robots.txt | 개인 앱이므로 크롤링 불필요 |
| **낮음** | sitemap.xml | Single Page App, 페이지 없음 |
| **낮음** | 시맨틱 HTML | 리팩토링 비용 대비 효과 미미 |
| **낮음** | 구조화된 데이터 | 개인 일기장에 불필요 |

---

## 3. 보안 아키텍처

### 3.1 보안 레이어 구조

```
┌─────────────────────────────────────────────────────────┐
│                  1. 네트워크 보안                          │
│   HTTPS (Vercel 자동), Vercel Proxy (/api → Bkend)      │
├─────────────────────────────────────────────────────────┤
│                  2. 인증 보안                             │
│   닉네임→이메일 변환, PIN→비밀번호 변환, Bearer Token       │
├─────────────────────────────────────────────────────────┤
│                  3. 클라이언트 보안                        │
│   입력 검증, XSS 방지, 토큰 관리                          │
├─────────────────────────────────────────────────────────┤
│                  4. 데이터 보안                            │
│   이미지 Base64, 서버사이드 검증 (Bkend)                   │
├─────────────────────────────────────────────────────────┤
│                  5. 서버 보안 (Bkend 관리)                 │
│   MongoDB 보안, API 인증, 데이터 암호화                     │
└─────────────────────────────────────────────────────────┘
```

### 3.2 보안 책임 분담

| 영역 | 책임자 | 주요 보안 사항 |
|------|--------|--------------|
| 네트워크 | Vercel | HTTPS 자동 적용, 프록시 |
| 서버/DB | Bkend (BaaS) | MongoDB 접근 제어, API 인증 |
| 클라이언트 | DOOGIE 코드 | 입력 검증, XSS 방지, 토큰 관리 |
| 인증 | DOOGIE + Bkend | 닉네임/PIN 검증, 세션 관리 |

---

## 4. 인증 보안 점검

### 4.1 현재 인증 방식

```
사용자 입력          변환 로직                  서버 전송
─────────          ─────────                  ─────────
닉네임: "koyu"  →  nicknameToEmail()      →  "koyu@doogie.app"
PIN: "1234"     →  pinToPassword()        →  "Doo@gie1234#Pwd"
```

### 4.2 인증 보안 항목

| 항목 | 현재 상태 | 위험도 | 설명 |
|------|----------|--------|------|
| PIN 복잡도 | ⚠️ 4자리 숫자만 | **낮음** | 개인 일기장 특성상 간편 인증 적절 |
| 비밀번호 변환 | ✅ 패턴 적용 | 낮음 | `Doo@gie{PIN}#Pwd` (Bkend 정책 충족) |
| 토큰 저장 | ⚠️ localStorage | **중간** | XSS 취약 시 토큰 탈취 가능 |
| 세션 만료 | ✅ 자동 처리 | 낮음 | 401 응답 시 자동 로그아웃 |
| 브루트포스 | ⚠️ 미보호 | **중간** | 클라이언트에서 Rate Limiting 없음 |
| 닉네임 노출 | ⚠️ 이메일 변환 | 낮음 | `@doogie.app` 도메인으로 제한 |

### 4.3 토큰 관리 흐름

```
로그인 성공
    │
    ├── accessToken → localStorage('doogie_access_token')
    │
    ├── 모든 API 요청 시:
    │   └── Authorization: Bearer {accessToken}
    │
    ├── 401 응답 수신 시:
    │   ├── localStorage에서 토큰 삭제
    │   ├── currentUser = null
    │   └── 인증 화면 표시
    │
    └── 로그아웃 시:
        ├── localStorage에서 토큰 삭제
        ├── accessToken = null
        ├── currentUser = null
        └── UI 초기화
```

### 4.4 토큰 보안 권장사항

```
현재:  localStorage 저장 (XSS 취약)
권장:  httpOnly Cookie (서버 설정 필요)

현재 구현이 적절한 이유:
  • 개인 일기장 (공격 동기 낮음)
  • Bkend BaaS 제약 (httpOnly 쿠키 미지원)
  • PIN 4자리로 인해 토큰이 탈취되어도 피해 제한적

향후 개선 시:
  • Bkend에서 httpOnly 쿠키 지원 시 전환
  • Refresh Token 도입 검토
```

---

## 5. XSS (Cross-Site Scripting) 보안

### 5.1 XSS 취약점 분석

| 위치 | 방식 | 위험도 | 상태 |
|------|------|--------|------|
| 닉네임 표시 | `textContent` | 없음 | ✅ 안전 |
| 상태 메시지 | `textContent` | 없음 | ✅ 안전 |
| 에러 메시지 | `textContent` | 없음 | ✅ 안전 |
| 일기 에디터 | `<textarea>` value | 없음 | ✅ 안전 (textarea는 HTML 파싱 안 함) |
| 마크다운 렌더링 | `innerHTML` (marked.js) | **중간** | ⚠️ 주의 필요 |
| Mermaid 렌더링 | `innerHTML` (SVG) | **중간** | ⚠️ 주의 필요 |
| 달력 엔트리 목록 | `textContent` | 없음 | ✅ 안전 |
| 이미지 표시 | Base64 `src` | 없음 | ✅ 안전 |

### 5.2 마크다운 XSS 위험

```
위험 시나리오:
  사용자가 마크다운에 악성 코드 입력 가능

  예시: <img src=x onerror="alert('XSS')">
  예시: <script>document.cookie</script>
  예시: [link](javascript:alert('XSS'))

현재 방어:
  • marked.js 기본 sanitize는 제한적
  • 자기 자신의 일기만 렌더링 (외부 입력 없음)

위험 완화 요인:
  • 일기 내용은 작성자 본인만 볼 수 있음
  • 외부에서 일기 내용을 주입할 경로가 없음
  • Self-XSS는 공격 벡터로 인정하지 않는 것이 일반적

권장 개선 (향후):
  DOMPurify 라이브러리 추가
  <script src="https://cdn.jsdelivr.net/npm/dompurify/dist/purify.min.js"></script>

  markdownPreview.innerHTML = DOMPurify.sanitize(marked.parse(content));
```

### 5.3 안전한 DOM 조작 패턴

```javascript
// ✅ 안전: textContent 사용 (DOOGIE 표준)
element.textContent = userInput;

// ✅ 안전: textarea value (에디터)
editor.value = entryText;

// ✅ 안전: src에 Base64 데이터
img.src = 'data:image/png;base64,...';

// ⚠️ 주의: innerHTML (마크다운 전용)
// marked.js 출력에만 사용, 직접 사용자 입력 X
markdownPreview.innerHTML = marked.parse(content);

// ❌ 금지: 사용자 입력을 직접 innerHTML에
element.innerHTML = userInput;  // 절대 금지
```

---

## 6. 입력 검증 현황

### 6.1 클라이언트 검증

| 입력 | 검증 규칙 | 위치 |
|------|----------|------|
| 닉네임 | 2-12자, 문자열 | `handleAuthSubmit()` |
| PIN | 정확히 4자리, 숫자만 (`/^\d{4}$/`) | `handleAuthSubmit()` |
| 일기 텍스트 | 빈 값 체크 (`.trim()`) | `saveCurrentEntry()` |
| 이미지 파일 | `image/*` MIME 타입 | `<input accept="image/*">` |
| 엔트리 수 | 하루 100개 제한 | `canCreateEntryToday()` |

### 6.2 서버 검증 (Bkend)

| 검증 | 서버 동작 |
|------|----------|
| 이메일 형식 | Bkend 자체 검증 |
| 비밀번호 정책 | 8자 이상, 대소문자+숫자+특수문자 |
| 중복 키 | E11000 에러 (unique index) |
| 인증 토큰 | 만료 시 401 응답 |
| 권한 (RBAC) | `self` 정책: 본인 데이터만 접근 |

### 6.3 검증 흐름

```
사용자 입력
    │
    ├── [클라이언트 검증] (UX 향상)
    │   ├── 형식 검사 (길이, 패턴)
    │   ├── 빈 값 방지
    │   └── 실시간 피드백 (닉네임 중복 체크)
    │
    ├── [서버 전송]
    │   └── HTTPS + Authorization 헤더
    │
    └── [서버 검증] (보안 핵심)
        ├── Bkend 자체 검증 규칙
        ├── unique index (중복 방지)
        └── RBAC 권한 확인
```

---

## 7. 네트워크 보안

### 7.1 HTTPS

```
상태: ✅ 자동 적용 (Vercel)

모든 HTTP 요청 → HTTPS로 리다이렉트 (Vercel 기본)
API 프록시: /api/* → https://api-enduser.bkend.ai/*
```

### 7.2 API 프록시 보안

```
브라우저                    Vercel                    Bkend
  │                         │                         │
  ├── /api/data/entries ──→ │                         │
  │   (동일 출처)            ├── api-enduser.bkend.ai ──→
  │                         │   (프록시 전달)           │
  │                         │                         │
  │ ← JSON 응답 ───────────┤ ← JSON 응답 ────────────┤

장점:
  • CORS 문제 없음 (동일 출처)
  • Bkend API URL 노출 방지 (브라우저에서 숨김)
  • Vercel Edge에서 요청 처리 (빠른 응답)
```

### 7.3 보안 헤더 현황

| 헤더 | 현재 상태 | 권장 |
|------|----------|------|
| `Strict-Transport-Security` | ❌ 미설정 | `max-age=63072000` |
| `X-Frame-Options` | ❌ 미설정 | `DENY` |
| `X-Content-Type-Options` | ❌ 미설정 | `nosniff` |
| `Referrer-Policy` | ❌ 미설정 | `strict-origin-when-cross-origin` |
| `Content-Security-Policy` | ❌ 미설정 | 아래 참조 |

### 7.4 권장 vercel.json 보안 헤더

```json
{
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://api-enduser.bkend.ai/:path*"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        },
        {
          "key": "Strict-Transport-Security",
          "value": "max-age=63072000; includeSubDomains"
        }
      ]
    }
  ]
}
```

### 7.5 권장 Content Security Policy

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'unsafe-inline' cdn.jsdelivr.net;
  style-src 'self' 'unsafe-inline' fonts.googleapis.com cdn.jsdelivr.net;
  img-src 'self' data: blob:;
  font-src 'self' fonts.googleapis.com fonts.gstatic.com cdn.jsdelivr.net;
  connect-src 'self';
  frame-ancestors 'none';

설명:
  • 'unsafe-inline': Single HTML File이므로 인라인 스크립트/스타일 필수
  • cdn.jsdelivr.net: marked.js, mermaid 등 CDN 라이브러리
  • data: blob:: Base64 이미지 및 Worker blob
  • frame-ancestors 'none': 클릭재킹 방지
```

---

## 8. 데이터 보안

### 8.1 민감 데이터 분류

| 데이터 | 민감도 | 저장 위치 | 보호 방법 |
|--------|--------|----------|----------|
| 닉네임 | 낮음 | 서버 (이메일 형태) | 메모리 내 표시, 로그아웃 시 삭제 |
| PIN | **높음** | 전송 시에만 | 비밀번호 변환 후 전송, 저장 안 함 |
| 인증 토큰 | **높음** | localStorage | 로그아웃/만료 시 삭제 |
| 일기 텍스트 | **높음** | MongoDB | HTTPS 전송, Bkend 접근 제어 |
| 이미지 | 중간 | MongoDB (Base64) | HTTPS 전송, Bkend 접근 제어 |
| Project ID | 낮음 | 소스코드 | BaaS 표준 (공개 가능) |

### 8.2 클라이언트 데이터 노출

```
localStorage에 저장되는 데이터:
  ✅ doogie_access_token    (인증 토큰 - 필요)

IndexedDB에 저장되는 데이터:
  ✅ entries    (일기 캐시)
  ✅ settings   (앱 설정)
  ✅ characters (캐릭터 데이터)

소스코드에 포함된 데이터:
  ⚠️ projectId: 'xmw50no31alhxozn08ex'  (BaaS 표준, 위험 낮음)
  ⚠️ environment: 'dev'                  (환경 식별자)

저장하지 않는 데이터:
  ✅ PIN (메모리 내에서만 변환 후 즉시 전송)
  ✅ 비밀번호 (변환 후 전송, 저장 안 함)
```

### 8.3 이미지 보안

```
이미지 처리 파이프라인:

파일 선택 → MIME 검사 → Canvas 로드 → 픽셀화 → Base64 변환 → 저장
            (image/*)                   (3px 축소)  (data URL)

보안 특성:
  ✅ MIME 타입 검사 (accept="image/*")
  ✅ Canvas 재생성 (원본 메타데이터 제거)
  ✅ Base64 저장 (외부 URL 없음, XSS 벡터 없음)
  ✅ 픽셀화 처리 (원본 이미지 복원 불가)

미구현:
  ⚠️ 파일 크기 제한 없음 (대용량 이미지 → 메모리 문제)
  ⚠️ Canvas taint 검사 없음 (CORS 이미지 시)

권장:
  최대 파일 크기 제한: 5MB (Base64 변환 전)
  최대 이미지 수 제한: 엔트리당 10장
```

---

## 9. 성능 보안

### 9.1 외부 스크립트 로딩

```html
현재 (동기 로딩 - 렌더링 차단):
  <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/gif.js@0.2.0/dist/gif.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/omggif@1.0.10/omggif.js"></script>

권장 (비동기 로딩):
  <script src="..." defer></script>

참고: <body> 끝에 위치하므로 실질적 차단 영향은 제한적
      하지만 defer 추가 시 병렬 다운로드로 성능 개선
```

### 9.2 CDN 보안 (Subresource Integrity)

```html
권장: SRI 해시 추가로 CDN 변조 방지

<script
  src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"
  integrity="sha384-{해시값}"
  crossorigin="anonymous"
  defer>
</script>

장점: CDN이 해킹되어도 변조된 스크립트 실행 방지
주의: 라이브러리 업데이트 시 해시도 함께 업데이트 필요
```

### 9.3 Base64 이미지 성능

```
문제:
  원본 이미지 100KB → Base64 인코딩 → ~133KB (+33%)
  10장 이미지 엔트리 → ~1.3MB의 JSON 데이터

영향:
  • API 요청/응답 크기 증가
  • IndexedDB 저장 공간 빠르게 소진
  • 엔트리 로드 시 메모리 사용량 증가

현재 완화:
  ✅ 픽셀화로 이미지 크기 대폭 축소 (3px 해상도)
  ✅ Thumbnail (600px) + Full (1500px) 분리 저장
  ✅ 달력 목록에서는 이미지 로드 안 함

추가 권장:
  • 파일 크기 상한선 (Base64 변환 전 5MB)
  • 이미지 수 제한 (엔트리당 10장)
  • 이미지 지연 로딩 (IntersectionObserver)
```

---

## 10. 위험 패턴 점검

### 10.1 안전 확인된 패턴

```
✅ eval() 사용 없음
✅ new Function() 사용 없음
✅ document.write() 사용 없음
✅ 동적 스크립트 생성 없음
✅ 비밀번호 하드코딩 없음
✅ SQL 인젝션 해당 없음 (NoSQL + BaaS)
✅ 직접적인 DOM injection 없음
```

### 10.2 주의가 필요한 패턴

```
⚠️ innerHTML 사용 (마크다운 렌더링 전용)
   → 완화: 자기 자신의 일기만 렌더링, Self-XSS
   → 권장: DOMPurify 추가

⚠️ localStorage 토큰 저장
   → 완화: 개인 일기장, 토큰 만료 자동 처리
   → 권장: httpOnly 쿠키 (Bkend 지원 시)

⚠️ Project ID 소스코드 노출
   → 완화: BaaS 표준, 인증 없이 데이터 접근 불가
   → 권장: 환경 변수로 관리 (빌드 시 주입)
```

---

## 11. 보안 개선 로드맵

### 11.1 우선순위별 개선 항목

```
즉시 적용 가능 (코드 변경 최소):
┌────┬──────────────────────────────┬─────────┐
│ #  │ 항목                          │ 난이도   │
├────┼──────────────────────────────┼─────────┤
│ 1  │ vercel.json 보안 헤더 추가     │ ⭐       │
│ 2  │ Meta description 추가         │ ⭐       │
│ 3  │ OG 태그 추가                  │ ⭐       │
│ 4  │ Favicon 추가                  │ ⭐       │
│ 5  │ 외부 스크립트에 defer 추가     │ ⭐       │
└────┴──────────────────────────────┴─────────┘

단기 개선 (코드 수정 필요):
┌────┬──────────────────────────────┬─────────┐
│ #  │ 항목                          │ 난이도   │
├────┼──────────────────────────────┼─────────┤
│ 6  │ DOMPurify 추가 (마크다운 XSS) │ ⭐⭐     │
│ 7  │ 이미지 파일 크기 제한           │ ⭐⭐     │
│ 8  │ SRI 해시 추가 (CDN 스크립트)   │ ⭐⭐     │
│ 9  │ 이미지 지연 로딩               │ ⭐⭐     │
└────┴──────────────────────────────┴─────────┘

장기 개선 (인프라 변경 필요):
┌────┬──────────────────────────────┬─────────┐
│ #  │ 항목                          │ 난이도   │
├────┼──────────────────────────────┼─────────┤
│ 10 │ httpOnly 쿠키 (Bkend 지원 시) │ ⭐⭐⭐   │
│ 11 │ Rate Limiting                │ ⭐⭐⭐   │
│ 12 │ 이미지 별도 스토리지 (CDN)     │ ⭐⭐⭐   │
└────┴──────────────────────────────┴─────────┘
```

---

## 12. 보안 체크리스트

### 12.1 인증 보안

- [x] 비밀번호 평문 저장 안 함 (변환 후 전송)
- [x] 세션 만료 시 자동 로그아웃
- [x] 로그아웃 시 토큰 완전 삭제
- [x] 닉네임/PIN 클라이언트 검증
- [ ] httpOnly 쿠키 (Bkend 미지원)
- [ ] Rate Limiting (클라이언트 미구현)
- [ ] Refresh Token (미구현)

### 12.2 XSS 방어

- [x] 사용자 입력 textContent로 표시
- [x] textarea value로 에디터 내용 관리
- [x] Base64 이미지 (외부 URL 없음)
- [x] eval(), document.write() 미사용
- [ ] DOMPurify (마크다운 렌더링)
- [ ] CSP 헤더

### 12.3 네트워크 보안

- [x] HTTPS (Vercel 자동)
- [x] API 프록시 (Bkend URL 숨김)
- [x] Authorization 헤더 자동 주입
- [ ] 보안 헤더 (X-Frame-Options 등)
- [ ] SRI 해시 (CDN 스크립트)

### 12.4 데이터 보안

- [x] PIN 미저장 (메모리 내 변환만)
- [x] 이미지 MIME 타입 검사
- [x] 이미지 Canvas 재생성 (메타데이터 제거)
- [x] RBAC self 정책 (본인 데이터만)
- [ ] 이미지 파일 크기 제한
- [ ] 이미지 수 제한

### 12.5 SEO

- [x] Title 태그
- [x] Charset, Viewport
- [x] 한국어 lang 속성
- [ ] Meta description
- [ ] Open Graph 태그
- [ ] Favicon
- [ ] robots.txt (선택적)

---

## 13. 결론

### DOOGIE의 보안 등급: **양호 (B)**

```
강점:
  ✅ 개인 일기장이므로 공격 표면 제한적
  ✅ Vercel + Bkend 인프라 보안 활용
  ✅ 안전한 DOM 조작 패턴 (textContent)
  ✅ 이미지 처리 파이프라인 안전
  ✅ 입력 검증 적절

개선 필요:
  ⚠️ 보안 헤더 미설정 (vercel.json)
  ⚠️ 마크다운 XSS 방어 미흡 (DOMPurify)
  ⚠️ 이미지 크기/수 제한 없음
  ⚠️ CDN SRI 미적용
  ⚠️ SEO 메타 태그 부족
```

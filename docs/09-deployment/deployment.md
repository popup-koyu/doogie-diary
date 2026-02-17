# DOOGIE - 배포 설정 명세 (Phase 9)

> 배포 환경 구성, 빌드, 배포 실행 및 검증

**버전**: 1.0
**최종 수정일**: 2026-02-18
**프로젝트**: DOOGIE Diary

---

## 1. 배포 아키텍처

### 1.1 전체 구조

```
┌──────────────────────────────────────────────────────────┐
│                        사용자 (브라우저)                    │
│                    https://doogie-diary.vercel.app         │
└────────────────────────┬─────────────────────────────────┘
                         │ HTTPS
                         ▼
┌──────────────────────────────────────────────────────────┐
│                     Vercel Edge Network                    │
│                                                           │
│   ┌─────────────────┐    ┌──────────────────────────┐    │
│   │  정적 파일 서빙   │    │  API Rewrites (Proxy)     │    │
│   │  index.html      │    │  /api/* → Bkend API       │    │
│   │  image/*         │    │                           │    │
│   │  gif.worker.js   │    │                           │    │
│   └─────────────────┘    └────────────┬─────────────┘    │
│                                        │                  │
└────────────────────────────────────────┼──────────────────┘
                                         │ HTTPS
                                         ▼
                              ┌─────────────────────┐
                              │   Bkend BaaS         │
                              │   api-enduser.       │
                              │   bkend.ai           │
                              │                     │
                              │   ┌───────────────┐ │
                              │   │   MongoDB      │ │
                              │   │   (클라우드)    │ │
                              │   └───────────────┘ │
                              └─────────────────────┘
```

### 1.2 배포 레벨

```
DOOGIE는 Dynamic 레벨:
  ✅ Vercel 자동 배포 (Git 연동)
  ✅ Bkend BaaS (서버리스 백엔드)
  ✅ CDN 배포 (Vercel Edge)
  ✅ HTTPS 자동 적용
```

---

## 2. 환경 구성

### 2.1 환경 분류

```
┌──────────┬──────────────────────┬──────────────────────┐
│ 환경      │ 용도                  │ 접근 방법             │
├──────────┼──────────────────────┼──────────────────────┤
│ Local    │ 개발자 로컬 개발       │ localhost:3000       │
│ Preview  │ PR/Branch 미리보기    │ *.vercel.app (자동)   │
│ Prod     │ 실서비스              │ doogie-diary.vercel  │
│          │                      │ .app                 │
└──────────┴──────────────────────┴──────────────────────┘
```

### 2.2 환경별 설정값

```
┌──────────────────┬──────────────────┬──────────────────┐
│ 설정              │ Local (Vite)     │ Prod (Vercel)    │
├──────────────────┼──────────────────┼──────────────────┤
│ API Proxy        │ vite.config.js   │ vercel.json      │
│                  │ /api → Bkend     │ /api → Bkend     │
│ Port             │ 3000             │ N/A (Edge)       │
│ HTTPS            │ ❌ (http)         │ ✅ (자동)         │
│ 빌드             │ 불필요 (HMR)      │ vite build       │
│ Bkend 환경       │ dev              │ dev              │
│ Project ID       │ 소스코드 내장     │ 소스코드 내장      │
└──────────────────┴──────────────────┴──────────────────┘
```

### 2.3 환경 변수

```
DOOGIE의 환경 변수 현황:

소스코드 내장 (index.html):
  PROJECT_ID  = 'xmw50no31alhxozn08ex'
  ENVIRONMENT = 'dev'
  BASE_URL    = '/api'

외부 환경 변수: 없음
  → Single HTML File + BaaS 구조로 환경 변수 분리 불필요
  → 향후 환경 분리 시 Vite의 import.meta.env 활용 가능

향후 환경 분리 시:
  .env.local:
    VITE_PROJECT_ID=xmw50no31alhxozn08ex
    VITE_ENVIRONMENT=dev
    VITE_API_BASE_URL=/api

  index.html에서:
    const PROJECT_ID = import.meta.env.VITE_PROJECT_ID;
```

---

## 3. 빌드 설정

### 3.1 Vite 설정 (vite.config.js)

```javascript
import { defineConfig } from 'vite';

export default defineConfig({
  server: {
    port: 3000,
    open: true,
    proxy: {
      '/api': {
        target: 'https://api-enduser.bkend.ai',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
});
```

### 3.2 빌드 명령어

```bash
# 개발 서버 (로컬)
npm run dev          # → vite (HMR, 프록시)

# 프로덕션 빌드
npm run build        # → vite build (dist/ 폴더 생성)

# 빌드 미리보기
npm run preview      # → vite preview (빌드 결과 확인)
```

### 3.3 빌드 출력

```
dist/
├── index.html          # 번들된 HTML (CSS+JS 인라인)
├── image/
│   ├── doogie.png
│   └── Doogie_diary2.png
├── gif.worker.js       # Web Worker
└── card_dos.gif        # 에셋
```

### 3.4 의존성

```json
{
  "devDependencies": {
    "vite": "^7.3.0"
  }
}

참고: 런타임 의존성 없음
      외부 라이브러리는 모두 CDN으로 로드
      - marked.js (마크다운)
      - mermaid.js (다이어그램)
      - html2canvas (캡처)
      - gif.js (GIF 생성)
      - omggif (GIF 파싱)
```

---

## 4. Vercel 배포 설정

### 4.1 vercel.json

```json
{
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://api-enduser.bkend.ai/:path*"
    }
  ]
}
```

### 4.2 Vercel 프로젝트 설정

```
Framework Preset: Vite
Build Command:    vite build (자동 감지)
Output Directory: dist (자동 감지)
Install Command:  npm install

Root Directory:   ./ (기본)
Node.js Version:  20.x (권장)
```

### 4.3 배포 흐름

```
GitHub Push (main)
    │
    ▼
Vercel 자동 감지
    │
    ├── npm install
    ├── vite build
    ├── dist/ 폴더 업로드
    │
    ▼
Vercel Edge Network 배포
    │
    ├── 정적 파일 CDN 배포
    ├── API Rewrites 규칙 적용
    ├── HTTPS 자동 적용
    │
    ▼
https://doogie-diary.vercel.app 서비스 시작
```

### 4.4 자동 배포 트리거

```
main 브랜치 Push → Production 배포
기타 브랜치 Push → Preview 배포 (고유 URL)
PR 생성         → Preview 배포 + 코멘트에 URL
```

---

## 5. 배포 전 체크리스트

### 5.1 빌드 검증

```
[ ] npm run build 성공
[ ] dist/ 폴더에 index.html 존재
[ ] dist/ 폴더에 image/ 폴더 존재
[ ] dist/ 폴더에 gif.worker.js 존재
[ ] npm run preview로 로컬 빌드 확인
```

### 5.2 기능 검증

```
[ ] 타이틀 화면 표시
[ ] 인증 (회원가입 + 로그인) 동작
[ ] 일기 작성 + 자동 저장 동작
[ ] 이미지 추가 + 픽셀화 동작
[ ] 달력 조회 동작
[ ] 마크다운 모드 동작
[ ] 엔트리 삭제 동작
[ ] 로그아웃 동작
[ ] 세션 복원 (새로고침 후 로그인 유지)
```

### 5.3 반응형 검증

```
[ ] 데스크톱 (1024px+) 레이아웃 정상
[ ] 태블릿 (768px) 레이아웃 정상
[ ] 모바일 (480px) 레이아웃 정상
[ ] 모바일 터치 인터랙션 정상
```

### 5.4 CDN/외부 리소스 검증

```
[ ] marked.js CDN 로드 확인
[ ] mermaid.js CDN 로드 확인
[ ] html2canvas CDN 로드 확인
[ ] gif.js CDN 로드 확인
[ ] 구글 폰트 로드 확인 (Silkscreen, VT323 등)
```

### 5.5 API 프록시 검증

```
[ ] /api/auth/signin/password 정상 (인증)
[ ] /api/v1/data/entries 정상 (CRUD)
[ ] CORS 에러 없음
[ ] 401 응답 시 자동 로그아웃
```

---

## 6. 보안 헤더 설정 (권장)

### 6.1 현재 vercel.json (최소 설정)

```json
{
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://api-enduser.bkend.ai/:path*"
    }
  ]
}
```

### 6.2 권장 vercel.json (보안 헤더 포함)

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

---

## 7. 모니터링

### 7.1 Vercel 내장 모니터링

```
Vercel Dashboard에서 확인 가능:

┌──────────────────────────────────────────┐
│ Analytics (무료 플랜 포함)                 │
│  ├── 방문자 수                            │
│  ├── 페이지 뷰                            │
│  ├── 국가별 분포                           │
│  └── 디바이스별 분포                        │
├──────────────────────────────────────────┤
│ Speed Insights                            │
│  ├── TTFB (Time To First Byte)           │
│  ├── FCP (First Contentful Paint)        │
│  ├── LCP (Largest Contentful Paint)      │
│  └── CLS (Cumulative Layout Shift)       │
├──────────────────────────────────────────┤
│ Logs                                      │
│  ├── 빌드 로그                            │
│  ├── 함수 실행 로그                        │
│  └── Edge 요청 로그                       │
└──────────────────────────────────────────┘
```

### 7.2 브라우저 DevTools 모니터링 (Zero Script QA)

```
개발자 도구 > Network 탭:

[확인 항목]
  ✅ index.html 로드 시간
  ✅ CDN 스크립트 로드 상태 (200 OK)
  ✅ /api/* 요청 상태 코드
  ✅ 이미지 Base64 크기
  ✅ 전체 페이지 로드 크기
```

---

## 8. 롤백 전략

### 8.1 Vercel 즉시 롤백

```
배포 문제 발생 시:

1. Vercel Dashboard → Deployments
2. 이전 성공 배포 선택
3. "Promote to Production" 클릭
4. 즉시 롤백 완료 (수초 내)

또는 Git 기반:
  git revert HEAD
  git push origin main
  → Vercel 자동 재배포
```

### 8.2 롤백 판단 기준

```
즉시 롤백 (P0):
  ❌ 인증 완전 실패
  ❌ 일기 저장 불가
  ❌ 전체 화면 빈 페이지

확인 후 롤백 (P1):
  ⚠️ 특정 기능 오작동
  ⚠️ 모바일만 문제
  ⚠️ CDN 리소스 로드 실패

모니터링 (P2):
  ℹ️ 성능 저하
  ℹ️ 비핵심 기능 이슈
  ℹ️ UI 깨짐 (일부)
```

---

## 9. 커스텀 도메인 (선택)

### 9.1 현재 도메인

```
Production: https://doogie-diary.vercel.app
Preview:    https://doogie-diary-{hash}.vercel.app
```

### 9.2 커스텀 도메인 연결 방법

```
1. 도메인 구매 (예: doogie.app)
2. Vercel Dashboard → Domains
3. 도메인 추가
4. DNS 설정:
   Type: CNAME
   Name: @
   Value: cname.vercel-dns.com

5. SSL 자동 발급 (Let's Encrypt)
```

---

## 10. 배포 히스토리

### 10.1 릴리즈 기록

| 날짜 | 버전 | 내용 | 커밋 |
|------|------|------|------|
| - | v1.0.0 | 초기 배포 | - |

### 10.2 배포 관리 규칙

```
브랜치 전략:
  main        → Production 자동 배포
  feature/*   → Preview 자동 배포 (PR 시)

커밋 컨벤션:
  Add:      새 기능
  Fix:      버그 수정
  Update:   기존 기능 개선
  Refactor: 코드 리팩토링
  Style:    UI/디자인 변경
  Docs:     문서 변경

태그 (선택):
  git tag v1.0.0
  git push origin v1.0.0
```

---

## 11. 트러블슈팅 가이드

### 11.1 빌드 실패

```
문제: npm run build 실패
원인: node_modules 없음 또는 Vite 버전 불일치

해결:
  rm -rf node_modules
  npm install
  npm run build
```

### 11.2 API 프록시 실패

```
문제: /api/* 요청 404 또는 CORS 에러

로컬 확인:
  1. vite.config.js proxy 설정 확인
  2. Bkend API 상태 확인 (https://api-enduser.bkend.ai)

Vercel 확인:
  1. vercel.json rewrites 설정 확인
  2. Vercel Logs에서 에러 확인
```

### 11.3 CDN 리소스 로드 실패

```
문제: marked.js / mermaid.js 등 CDN 로드 실패

확인:
  1. cdn.jsdelivr.net 서비스 상태 확인
  2. 브라우저 콘솔에서 에러 메시지 확인
  3. CSP 정책이 CDN 차단하는지 확인

대안:
  CDN 장애 시 npm 패키지로 전환 후 빌드에 포함
  npm install marked mermaid
  → import 방식으로 변경
```

### 11.4 인증 실패

```
문제: 로그인/회원가입 실패

확인:
  1. Network 탭에서 /api/auth/* 요청 확인
  2. 응답 상태 코드 확인 (401, 403, 500)
  3. Bkend 프로젝트 상태 확인
  4. X-Project-Id, X-Environment 헤더 확인

일반적 원인:
  • Bkend 프로젝트 비활성화
  • Project ID 변경
  • API 엔드포인트 변경
```

### 11.5 이미지 저장 실패

```
문제: 이미지 추가 후 저장 안 됨

확인:
  1. Base64 데이터 크기 확인
  2. Bkend 요청 Body 크기 제한 확인
  3. Network 탭에서 PUT/POST 요청 상태 확인

일반적 원인:
  • 이미지가 너무 큼 (Base64 > Bkend body 제한)
  • IndexedDB 저장 공간 부족 (로컬 모드)
```

---

## 12. 배포 최종 체크리스트

### 12.1 배포 전

```
코드:
  [ ] main 브랜치에 최신 코드 머지
  [ ] npm run build 성공
  [ ] npm run preview로 빌드 결과 확인

기능:
  [ ] 인증 (회원가입 + 로그인)
  [ ] 일기 CRUD (작성, 수정, 삭제, 조회)
  [ ] 이미지 (추가, 픽셀화, 모달)
  [ ] 달력 (월별 조회, 날짜 선택)
  [ ] 마크다운 (작성, 프리뷰, 내보내기)

인프라:
  [ ] Vercel 프로젝트 연결
  [ ] vercel.json rewrites 설정
  [ ] GitHub 연동 (자동 배포)
```

### 12.2 배포 후

```
검증:
  [ ] https://doogie-diary.vercel.app 접속 확인
  [ ] 타이틀 화면 정상 표시
  [ ] CDN 리소스 로드 성공
  [ ] API 프록시 동작 (인증 테스트)
  [ ] 모바일 화면 확인

모니터링:
  [ ] Vercel Dashboard 에러 없음
  [ ] 브라우저 콘솔 에러 없음
  [ ] Network 탭 요청 정상
```

---

## 13. 결론

### DOOGIE 배포 특성

```
장점:
  ✅ 단순한 배포 구조 (Single HTML + Vercel)
  ✅ 서버 관리 불필요 (BaaS + Edge)
  ✅ 자동 배포 (Git Push → 즉시 배포)
  ✅ 자동 HTTPS + CDN
  ✅ 즉시 롤백 가능 (Vercel Dashboard)
  ✅ 무료 플랜으로 운영 가능

제약:
  ⚠️ 환경 변수 분리 안 됨 (소스코드 내장)
  ⚠️ 보안 헤더 미설정 (vercel.json)
  ⚠️ CDN 의존성 (장애 시 영향)
  ⚠️ Bkend dev 환경만 사용 (prod 미분리)

향후 개선:
  • 환경 변수 분리 (Vite env + Vercel env)
  • 보안 헤더 추가 (vercel.json)
  • Bkend prod 환경 분리
  • 커스텀 도메인 연결
  • CDN 폴백 구현
```

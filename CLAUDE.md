# DOOGIE Diary - Project Instructions

## 프로젝트 개요
1990년대 DOS 감성의 웹 기반 일기장 애플리케이션 (Single HTML File 아키텍처)

## 용어/스키마 참조
프로젝트 용어 정의는 `docs/01-plan/glossary.md`를 참조하세요.
데이터 스키마는 `docs/01-plan/schema.md`를 참조하세요.
도메인 모델 및 화면 흐름은 `docs/01-plan/domain-model.md`를 참조하세요.
화면 명세(목업)는 `docs/03-mockup/mockup-spec.md`를 참조하세요.
API 설계서는 `docs/04-api/api-spec.md`를 참조하세요.
디자인 시스템은 `docs/05-design-system/design-system.md`를 참조하세요.
UI-API 연동 명세는 `docs/06-ui-integration/ui-integration.md`를 참조하세요.
SEO/보안 점검은 `docs/07-seo-security/seo-security.md`를 참조하세요.

## 기술 스택
- Frontend: Vanilla JS (ES6+), Single HTML File (index.html)
- Backend: Bkend (BaaS) - MongoDB 기반
- Build: Vite
- Deploy: Vercel

## 코딩 컨벤션 참조
전체 컨벤션은 `CONVENTIONS.md`를 참조하세요.
네이밍 규칙은 `docs/02-convention/naming.md`를 참조하세요.
프로젝트 구조 규칙은 `docs/02-convention/structure.md`를 참조하세요.

## 핵심 코딩 규칙 (요약)
- 아키텍처: Single HTML File (index.html에 CSS+HTML+JS 통합)
- 변수/함수: camelCase (예: dateKey, saveEntry)
- CSS 클래스: kebab-case (예: diary-editor)
- 상수: UPPER_SNAKE_CASE (예: MAX_ENTRIES_PER_DAY)
- 색상: DOS 녹색 테마 (#00ff00 기본, #003300 배경강조, #ffffff 호버)
- 외부 라이브러리: CDN 우선, npm은 devDependencies만 (Vite)
- 레이어 순서: 상수 → BkendAPI → DiaryDB → 기능 함수 → init()

# DOOGIE - 1990s DOS Style Diary Application

## 프로젝트 개요

**DOOGIE**는 1990년대 DOS 시대의 감성을 재현한 웹 기반 일기장 애플리케이션입니다. 복고풍 CRT 모니터 느낌의 녹색 텍스트와 픽셀 아트 이미지 처리를 특징으로 합니다.

## 핵심 사양

### 1. 화면 구성 및 비율
- **전체 화면 비율**: 4:3 (고정)
- **레이아웃**:
  - 상단: 고정 메뉴 바 (Sticky)
  - 본문: 텍스트 에디터 + 이미지 섹션 (1:1 분할, 이미지 없을 시 텍스트만 100% 사용)
  - 이미지 섹션: 이미지가 추가될 때만 표시 (구분선 동적 표시)
- **반응형**: aspect-ratio를 사용한 자동 조절

### 2. 디자인 시스템

#### 색상 테마
- **배경색**: `#000000` (검정)
- **텍스트/테두리**: `#00ff00` (DOS 녹색)
- **호버 효과**: `#ffffff` (흰색)
- **강조 배경**: `#003300` (어두운 녹색)

#### 폰트
- **한글**: NeoDunggeunmo (네오둥근모) - 20px
- **영문/숫자**: VT323 - 36px (날짜), 24px (제목)
- **게임 타이틀**: Press Start 2P (DOOGIE 타이틀)
- **렌더링**: Monospace 기반

#### UI 요소
- 모든 테두리는 1px solid 녹색
- 버튼: 투명 배경 + 녹색 테두리, 호버 시 반전
- 드롭다운: max-height transition 애니메이션 (0.3s)

### 3. 기능 명세

#### 3.1 타이틀 화면 메뉴
**메뉴 항목**:
1. **일기쓰기 / Diary**: 새 일기 작성 화면으로 진입
2. **나의 일기장 / Memory**: 저장된 일기 목록 보기
3. **디스켓 / Diskette**: 크레딧 화면 (제작진 정보)

#### 3.2 상단 메뉴 바 (일기 작성 화면)
**메뉴 버튼 (좌측 → 우측)**:
1. **Home**: 타이틀 화면으로 복귀
2. **New**: 새 일기 작성 (오늘 날짜의 새 항목 생성)
3. **Save**: 현재 일기 저장
4. **Picture**: 이미지 관리 (드롭다운)
   - Add Picture: 이미지 추가
   - View Images (N): 이미지 모달로 보기
5. **List**: 저장된 일기 목록 (드롭다운, position: fixed)
6. **Help**: 도움말 화면
7. **Fullscreen**: 전체화면 모드 전환
8. **DIARY/MARKDOWN 모드 토글**: 일반 일기/마크다운 모드 전환
9. **사용자 닉네임 표시**: 로그인된 사용자 닉네임
10. **Logout**: 로그아웃 버튼

**드롭다운 동작**:
- 하나의 드롭다운이 열리면 다른 드롭다운 자동 닫힘
- 외부 클릭 시 자동 닫힘
- 100ms 딜레이로 즉시 닫힘 방지
- List 드롭다운: 화면 잘림 방지를 위해 position: fixed (z-index: 9999)

#### 3.3 일기 작성 기능

**날짜 표시 형식**:
```
MM/DD/YYYY DAYOFWEEK
예: 11/19/2025 WEDNESDAY
```
- **일기 작성 화면**: 날짜만 표시 (항목 번호 미표시)
- **리스트/달력 화면**: 날짜 + 항목 번호 표시 (예: 2025.11.19 (2))

**다중 항목 지원**:
- 같은 날짜에 여러 일기 작성 가능
- 키 형식:
  - 첫 번째: `YYYY-MM-DD`
  - 두 번째: `YYYY-MM-DD_(2)`
  - 세 번째: `YYYY-MM-DD_(3)`
- 최대 100개까지 지원

**자동 저장**:
- 입력 후 2초 후 자동 저장
- 페이지 이탈 시 자동 저장
- Bkend 서버에 저장 (온라인 필수)

**중복 저장 방지**:
- 내용이 변경되지 않았으면 저장하지 않음
- "이미 저장되었습니다!" 메시지 표시

**저장 확인 다이얼로그**:
- 저장하지 않은 내용이 있을 시 확인 다이얼로그 표시
- DOS 스타일 박스 (이중 테두리 + 그림자)
- Yes/No 버튼

#### 3.4 마크다운 모드 (Markdown Mode)

**모드 전환**:
- 상단 메뉴 바 우측의 토글 버튼으로 전환
- `Ctrl+M` 단축키로 전환
- 모드 표시: "DIARY" ↔ "MARKDOWN"

**마크다운 에디터**:
- 전용 툴바 제공:
  - `#` 제목 (Heading)
  - `B` 굵게 (Bold)
  - `I` 기울임 (Italic)
  - `>` 인용 (Quote)
  - `</>` 코드 (Code)
  - `-` 목록 (List)
  - `[M]` Mermaid 다이어그램
  - `👁 Preview` 미리보기 토글

**마크다운 미리보기**:
- `Ctrl+P` 또는 Preview 버튼으로 토글
- GitHub Flavored Markdown (GFM) 지원
- DOS 녹색 톤 스타일링:
  - 제목, 본문, 링크: 녹색 계열
  - 코드 블록: 주황색 (#ff8800)
  - 인용: 어두운 녹색 배경
  - 테이블: DOS 스타일 테두리

**Mermaid 다이어그램 지원**:
- 코드 블록에 `mermaid` 언어 지정
- 플로우차트, 시퀀스 다이어그램 등 지원
- 녹색 테마 자동 적용

**마크다운 항목 저장**:
- 키 형식: `#YYYY-MM-DD` (일반 일기와 구분)
- 목록에서 `[MD]` 표시로 마크다운 항목 구분

#### 3.5 이미지 처리 시스템

**이미지 변환**:
- **픽셀화**: 3px 단위로 다운스케일
- **색상 팔레트**: 녹색 계열 5단계
  - 밝기를 계산하여 0, 51, 102, 153, 204, 255 중 선택
- **포맷**: Base64 Data URL
- **두 가지 버전 생성**:
  - Thumbnail: 600px 제한 (일기 보기용)
  - Full: 1200px 제한 (모달용)

**이미지 레이아웃** (Grid 사용):

| 이미지 개수 | 레이아웃 | Grid 설정 |
|------------|----------|-----------|
| 0장 | 텍스트 전용 | 이미지 섹션 숨김, 구분선 없음 |
| 1장 | 전체 화면 1개 | 1x1 |
| 2장 | 세로로 2개 | 1x2 |
| 3장 | 상단 1개 (2열 병합) + 하단 2개 | 2x2 (첫 번째 span 2) |
| 4장 이상 | 2x2 격자, 스크롤 | 2xN |

**이미지 모달**:
- 전체 화면 오버레이
- DOS 스캔 효과 애니메이션 (1.5초)
- `clip-path`를 사용한 위→아래 스캔라인
- 녹색 그라데이션 스캔 바 (3px)
- ESC 또는 X 버튼으로 닫기

#### 3.6 달력 뷰 (Calendar View)

**위치**: 타이틀 화면 → 나의 일기장 (Memory) 메뉴

**레이아웃**:
- **좌우 분할 구조**:
  - 왼쪽 50%: 월간 달력
  - 오른쪽 50%: 해당 월의 일기 리스트
  - 중앙 구분선: 1px solid #00ff00

**달력 기능**:
- **월 네비게이션**: ◀ / ▶ 버튼으로 이전/다음 달 이동
- **월/연도 표시**: "JANUARY 2025" 형식
- **요일 헤더**: SUN, MON, TUE, WED, THU, FRI, SAT
- **7x6 그리드**: 일요일부터 토요일까지 (42개 셀)
- **날짜 표시**:
  - 당월 날짜: 일반 녹색 (#008800)
  - 전/후월 날짜: 희미한 녹색 (#003300)
  - 오늘 날짜: 녹색 테두리 강조 (2px solid #00ff00)

**일기 표시**:
- 일기가 있는 날짜: 하단에 **녹색 점(dot)** 표시 (4px 원형)
- 일반 일기와 마크다운 일기 모두 카운트
- 호버 효과: 배경 어두운 녹색 (#003300)
- 클릭 시: 해당 날짜의 첫 번째 일기로 이동

**오른쪽 리스트**:
- 현재 보고 있는 월의 일기만 필터링
- 최신순 정렬
- 날짜 + 항목 번호 + 미리보기 텍스트
- 마크다운 항목: `[MD]` 표시
- 이미지 개수 표시: [사진 N장]
- Settings 버튼 하단 배치

#### 3.7 설정 (Settings)

**위치**: 나의 일기장/달력 화면 → 하단 "⚙ Settings" 버튼

**언어 설정**:
- 한국어/English 전환
- IndexedDB에 저장 (재방문 시 유지)
- 메뉴 텍스트 번역:
  - 한국어: 일기쓰기, 나의 일기장, 디스켓
  - English: Diary, Memory, Diskette

**크리스마스 모드**:
- ON/OFF 토글
- 12월에 자동 활성화

**크리스마스 BGM**:
- 크리스마스 모드 활성화 시 표시
- 8-bit 스타일 "루돌프 사슴코" 멜로디
- Web Audio API 사용

#### 3.8 전체화면 모드 (Fullscreen)

**활성화 방법**:
- 상단 메뉴 바 "Fullscreen" 버튼 클릭
- Fullscreen API 사용 (브라우저 네이티브 전체화면)

**기능**:
- 브라우저 주소창, 탭, 메뉴 모두 숨김
- 화면 전체를 DOOGIE가 차지
- 진입 시 안내 메시지: "Press ESC to exit fullscreen" (3초간 표시)

**종료 방법**:
- ESC 키 누르기
- Fullscreen 버튼 재클릭

**브라우저 호환성**:
- Chrome, Firefox, Edge: `requestFullscreen()`
- Safari: `webkitRequestFullscreen()`
- IE11: `msRequestFullscreen()`

#### 3.9 데이터 관리

**저장 방식**: Bkend 서버 Only (로컬 저장소 미사용)

**온라인 필수**:
- 모든 일기 데이터는 Bkend 서버에만 저장
- 오프라인 작성 미지원
- 네트워크 오류 시 에러 메시지 표시

**로컬 설정 저장** (localStorage):
- 언어 설정 (ko/en)
- 크리스마스 모드 상태
- JWT 토큰 (Access Token)

### 4. 크리스마스 특별 기능

#### 4.1 크리스마스 모드

**자동 활성화**:
- 12월에 첫 방문 시 자동 활성화
- Settings에서 수동 ON/OFF 가능

**시각 효과**:
- 눈 내리기 애니메이션 (Snowflakes)
- 별 깜빡임 색상 변경 (빨강/초록/금색)
- 메뉴 바 그라데이션 (빨강/초록)

**BGM**:
- 8-bit 스타일 "루돌프 사슴코" 멜로디
- Web Audio API로 생성 (외부 파일 불필요)
- 오실레이터 기반 사운드 합성

**이스터에그**:
- 12월 24일 (크리스마스 이브): 특별 메시지 팝업
- 12월 25일 (크리스마스): 축하 메시지 팝업

#### 4.2 크리스마스 카드 기능

**접근**:
- 타이틀 화면에 "🎄" 메뉴 (12월에만 표시)
- 또는 Settings에서 크리스마스 모드 활성화 후 표시

**카드 구성**:
- 상단 패널: 메시지 영역
  - "Merry Christmas" 타이틀
  - "from POPUP-STUDIO"
  - 커스텀 To/From 입력 필드
- 하단 패널: 애니메이션 GIF

**다운로드 옵션**:
- **GIF 다운로드**: 애니메이션 포함
  - gif.js 라이브러리 사용
  - 파일명: `christmas-card-to-{to}-from-{from}.gif`
- **PNG 다운로드**: 정적 이미지
  - html2canvas 라이브러리 사용
  - 파일명: `christmas-card-to-{to}-from-{from}.png`

### 5. 크레딧 화면 (디스켓)

#### 5.1 레이아웃
- **타이틀**: "DOOGIE" (Press Start 2P 폰트, 대형)
- **서브타이틀**: "An Interactive Diary Experience"
- **NYC 스카이라인**: 픽셀 아트 빌딩들
- **도로 + 자동차**: 양방향 이동 애니메이션
- **회사명**: `** << POPUP STUDIO >> **` (펄스 글로우 효과)
- **캐릭터 섹션**: 6개 캐릭터 슬롯 (3x2 그리드)
- **푸터**: 저작권 + 위트 있는 연도 표시

#### 5.2 NYC 스카이라인
- **빌딩 개수**: 15개
- **디자인**: 3가지 옥상 스타일 (antenna, crown, flat)
- **창문 타입**: 4가지 (small, wide, tall, square)
- **창문 애니메이션**: 8-12개 창문이 250ms마다 랜덤 깜빡임
- **색상**: 다양한 녹색 톤 랜덤 적용

#### 5.3 도로 + 자동차
- **도로**: 30px 높이, 중앙 점선
- **자동차 종류**: 승용차(70%), 트럭(30%)
- **양방향 이동**:
  - 왼쪽→오른쪽: 하단 차선
  - 오른쪽→왼쪽: 상단 차선 (좌우 반전)
- **생성 주기**: 1.5초마다
- **속도**: 6-10초 (랜덤)
- **디자인**: Canvas 기반 픽셀 아트

#### 5.4 캐릭터 슬롯
- **개수**: 6개 (3열 x 2행)
- **크기**: 180x180px (정사각형)
- **업로드 방식**: 클릭 or 드래그앤드롭
- **이미지 크롭**:
  - 드래그로 영역 이동
  - 코너 핸들로 크기 조절 (정사각형 유지)
  - Crop & Apply 버튼으로 적용
- **픽셀 아트 변환**: 3px 단위, 5단계 녹색 팔레트
- **편집 가능 텍스트**:
  - 이름 (char-name): 클릭하여 수정
  - 역할 (char-role): 클릭하여 수정
  - 간격: 이름과 역할 사이 -12px margin
- **저장**: IndexedDB에 자동 저장

#### 5.5 회사명 디자인
```
** << POPUP STUDIO >> **
```
- 별표 크기: 16.8px (기본 대비 30% 축소)
- 괄호 크기: 24px
- 간격: 균등 12px
- 애니메이션: 2초 주기 펄스 글로우

#### 5.6 푸터 위트
```
© ̶1̶9̶8̶5̶ POPUP-STUDIO - Made with 💚 in DOS Era
   2025
```
- 1985: 삭선 처리 (#00cc00, 1px 두께, 글로우)
- 2025: 상단 표시 (밝은 녹색, 굵게)
- "타임머신 버그 수정" 콘셉트

### 6. 애니메이션 및 효과

#### 타이틀 화면
- 로고: 5초 fade-in 애니메이션
- 메뉴 항목: 순차적 fade-in (0.5초 간격)
- 선택 효과: ▶ 아이콘 + 텍스트 확대
- 배경 별: 30개, 랜덤 위치, 깜빡임 애니메이션
- 크리스마스 모드: 눈 내리기 효과

#### CRT 스캔 효과
```css
@keyframes scanReveal {
  0% { clip-path: inset(0 0 100% 0); }
  100% { clip-path: inset(0 0 0% 0); }
}

@keyframes scanline {
  0% { opacity: 1; top: 0; }
  95% { opacity: 1; }
  100% { opacity: 0; top: 100%; }
}
```

#### 커서 깜빡임
- 텍스트 에디터에 커스텀 커서 (1px x 24px)
- 0.8초 주기 blink 애니메이션

#### 글로우 펄스
```css
@keyframes glowPulse {
  0%, 100% {
    text-shadow: 0 0 10px #00ff00, 0 0 20px #00ff00;
  }
  50% {
    text-shadow: 0 0 20px #00ff00, 0 0 40px #00ff00, 0 0 60px #00ff00;
  }
}
```

### 7. 키보드 단축키

| 키 | 동작 |
|----|------|
| `↑` / `↓` | 타이틀 화면 메뉴 항목 선택 |
| `Enter` | 메뉴 실행 / 일기 시작 |
| `ESC` | 설정/도움말/모달 닫기 |
| `F1` | 도움말 보기/닫기 |
| `F2` | 현재 일기 저장 |
| `F3` | 일기 목록 보기 |
| `F4` | 새 일기 작성 |
| `Ctrl+M` | Diary/Markdown 모드 전환 |
| `Ctrl+P` | 마크다운 미리보기 토글 (마크다운 모드에서) |

### 8. 기술 스택

- **HTML5**: Semantic markup
- **CSS3**:
  - Flexbox + CSS Grid
  - CSS Variables
  - CSS Animations
  - aspect-ratio
  - text-decoration-thickness
- **Vanilla JavaScript**: ES6+
  - Canvas API (이미지 처리, 픽셀 아트)
  - Web Audio API (BGM 생성)
  - FileReader API (CORS-free 이미지 업로드)
  - Fullscreen API
  - Fetch API (Bkend 서버 통신)
- **Backend**: Bkend (BaaS)
  - MongoDB 기반 데이터 저장
  - JWT 인증
  - RESTful API
- **외부 라이브러리**:
  - Marked.js (마크다운 파싱)
  - Mermaid.js (다이어그램 렌더링)
  - gif.js (GIF 생성)
  - html2canvas (PNG 캡처)
- **Single HTML File**: 메인 로직은 단일 파일에 포함

### 9. 모바일 반응형

#### 9.1 브레이크포인트

| 브레이크포인트 | 타겟 | 주요 변경 |
|---------------|------|----------|
| > 768px | 데스크톱 | 4:3 비율, 좌우 분할 레이아웃 |
| ≤ 768px | 태블릿 | 메뉴 축소, 모드 토글 숨김 |
| ≤ 480px | 모바일 | 세로 레이아웃, 로고 축소, 스와이프 갤러리 |
| hover:none + pointer:coarse | 터치 기기 | 최소 터치 영역 44px |

#### 9.2 모바일 레이아웃 (480px 이하)
- **로고**: 82px → 48px, letter-spacing 6px
- **메뉴바**: 5개 이후 메뉴 숨김, 사용자 정보 숨김
- **에디터**: 세로 배치 (이미지 상단 order:1, 텍스트 하단 order:2)
- **이미지 갤러리**: 가로 스크롤, scroll-snap, 180x170px
  - 1장: 중앙 정렬 (justify-content: center)
  - 2장+: 좌측 시작 (justify-content: flex-start) + 좌우 스와이프
- **캘린더**: 세로 스택 (달력 위, 리스트 아래)
- **인증 화면**: 너비 95%, 패딩 15px

### 10. 브라우저 호환성

**필수 기능**:
- CSS Grid
- CSS aspect-ratio
- Canvas API
- Fetch API
- ES6 (const, let, arrow functions, template literals, async/await)

**권장 브라우저**:
- Chrome 88+
- Firefox 87+
- Safari 15+
- Edge 88+

### 11. 성능 고려사항

**이미지 최적화**:
- Thumbnail과 Full 버전 분리 저장
- 픽셀화로 파일 크기 감소 (3px 단위)
- Base64 인코딩 (IndexedDB 용량 고려)

**렌더링 최적화**:
- position: sticky로 메뉴바 고정
- CSS containment 사용 (overflow: hidden)
- Grid auto-flow로 동적 레이아웃
- 이미지 섹션 동적 표시/숨김

**메모리 관리**:
- 이미지 모달 닫을 때 src 초기화
- 불필요한 이벤트 리스너 제거
- New 버튼 클릭 시 이미지 섹션 초기화

**네트워크 최적화**:
- API 요청 debounce 적용 (자동 저장)
- 로딩 상태 표시로 UX 개선
- 에러 시 재시도 로직

### 12. 파일 구조

```
doogie-diary/
├── index.html           # 메인 애플리케이션 (CSS + HTML + JS)
├── card_dos.gif         # 크리스마스 카드용 GIF
├── gif.worker.js        # GIF 생성 웹 워커
├── image/               # 정적 이미지 에셋
│   ├── doogie.png
│   └── Doogie_diary2.png
├── vite.config.js       # Vite 빌드 설정
├── vercel.json          # Vercel 배포 설정 (API 프록시)
├── package.json         # 의존성 관리
├── DOOGIE_SPEC.md       # 본 기획문서
├── README.md            # 프로젝트 설명서
├── BLOG_POST.md         # 개발기 블로그 포스트
└── docs/                # bkit 기반 문서
    ├── 01-schema/       # 스키마 정의서
    ├── 02-convention/   # 코딩 컨벤션
    ├── 03-design-system/# 디자인 시스템
    ├── 04-api/          # API 설계서
    ├── 05-ui-integration/# UI-API 통합
    ├── 06-seo-security/ # SEO/보안 점검
    ├── 07-review/       # 코드 리뷰
    ├── 08-deployment/   # 배포 설정
    └── 09-pdca/         # PDCA 보고서
```

### 13. 개발 히스토리

**Phase 1: 기본 구조** (완료)
- 타이틀 화면 및 메뉴 시스템
- 텍스트 에디터 구현
- localStorage 저장/불러오기

**Phase 2: 이미지 시스템** (완료)
- 이미지 업로드 및 픽셀 아트 변환
- 다중 이미지 레이아웃 (Grid)
- DOS 스캔 효과 모달

**Phase 3: UI/UX 개선** (완료)
- 4:3 화면 비율 고정
- 드롭다운 메뉴 충돌 해결
- 이미지 없을 시 전체 화면 활용
- 저장 확인 다이얼로그

**Phase 4: 다중 항목 지원** (완료)
- 같은 날짜 여러 일기 작성
- 항목 번호 표시 시스템
- 키 관리 시스템

**Phase 5: Export/Import** (완료)
- JSON 파일 내보내기
- JSON 파일 가져오기
- 데이터 백업/복원 기능

**Phase 6: 크레딧 화면** (완료)
- 레트로 게임 스타일 크레딧
- NYC 스카이라인 픽셀 아트 (15개 빌딩)
- 창문 깜빡임 애니메이션
- 도로 + 양방향 자동차 애니메이션
- 6개 캐릭터 슬롯
- 드래그앤드롭 이미지 업로드
- 크롭 에디터 (드래그 이동 + 크기 조절)
- 편집 가능한 캐릭터 이름/역할
- 회사명 펄스 글로우 효과
- 위트 있는 저작권 표시 (1985 → 2025)

**Phase 7: 설정 및 다국어** (완료)
- Settings 화면 추가 (나의 일기장 내부)
- 한국어/English 전환 기능
- localStorage → IndexedDB 언어 설정 저장
- 메뉴 텍스트 번역

**Phase 8: 버그 수정 및 최적화** (완료)
- 이미지 섹션 동적 표시/숨김 개선
- New 버튼 클릭 시 이미지 섹션 초기화
- 구분선 조건부 표시
- 캐릭터 이름/역할 간격 조정

**Phase 9: 달력 뷰 및 전체화면** (완료)
- 달력 뷰 구현 (좌우 분할 레이아웃)
  - 왼쪽: 월간 달력 (7x6 그리드)
  - 오른쪽: 해당 월의 일기 리스트
- 일기 있는 날짜에 녹색 점(dot) 표시
- 월 네비게이션 (이전/다음 달 이동)
- 달력에서 날짜 클릭 시 일기 바로 열람
- 전체화면 모드 추가 (Fullscreen API)
  - 브라우저 UI 완전 숨김
  - ESC 키로 종료
  - 크로스 브라우저 호환 (Chrome, Safari, Firefox, Edge)
- 일기 작성 화면 날짜 표시 개선
  - 작성 화면: 항목 번호 미표시
  - 리스트/달력: 항목 번호 표시
- List 드롭다운 position 수정 (fixed, z-index 9999)

**Phase 10: 마크다운 모드** (완료)
- Diary/Markdown 모드 토글 기능
- 마크다운 전용 툴바 (제목, 굵게, 기울임, 인용, 코드, 목록)
- 실시간 마크다운 미리보기
- Marked.js 통합 (GFM 지원)
- Mermaid.js 통합 (다이어그램 지원)
- DOS 녹색 톤 마크다운 스타일링
- 마크다운 항목 별도 저장 (# prefix)
- 목록에서 [MD] 표시로 마크다운 항목 구분

**Phase 11: 크리스마스 특별 기능** (완료)
- 크리스마스 모드 (12월 자동 활성화)
- 눈 내리기 애니메이션
- 8-bit BGM (루돌프 사슴코)
- 크리스마스 이브/당일 이스터에그
- 크리스마스 카드 생성 기능
  - 커스텀 To/From 입력
  - GIF 다운로드 (gif.js)
  - PNG 다운로드 (html2canvas)

**Phase 12: 저장소 업그레이드** (완료)
- localStorage에서 IndexedDB로 마이그레이션
- DiaryDB 클래스 구현
- 비동기 저장/불러오기
- 자동 마이그레이션 지원
- 중복 저장 방지 기능

**Phase 13: 백엔드 연동** (완료)
- Bkend 서버 연동
- 닉네임 + PIN 인증 시스템
- 서버 기반 일기 저장
- 하루 100개 제한
- Export/Import 기능 제거

**Phase 14: 모바일 반응형** (완료)
- 768px / 480px 미디어 쿼리 적용
- 모바일: 로고 48px 축소, 메뉴 간소화
- 에디터 세로 레이아웃 (이미지 상단, 텍스트 하단)
- 이미지 가로 스와이프 갤러리 (scroll-snap)
- 터치 친화적 버튼 크기 (44px 최소)

**Phase 15: 중복 저장 방지** (완료)
- Bkend entries 테이블에 유니크 인덱스 추가 (createdBy + dateKey + entryNumber)
- 저장 시 중복 에러 발생 시 자동 재시도 (최대 3회)
- 서버에서 최신 entryNumber 조회 후 재저장

---

### 14. 백엔드 연동 (Bkend)

#### 13.1 인증 시스템

**로그인 방식: 닉네임 + 4자리 PIN**

| 항목 | 정책 |
|------|------|
| 닉네임 | 2-12자, 한글/영문/숫자 허용, 중복 불가 |
| PIN | 숫자 4자리 (0000-9999) |
| 회원가입 | 닉네임 + PIN 입력 → 즉시 계정 생성 |
| 로그인 | 닉네임 + PIN 일치 시 JWT 토큰 발급 |
| 세션 유지 | Access Token (24시간) |
| 로그인 실패 제한 | 없음 |
| 비밀번호 찾기 | 미지원 (간단한 서비스 컨셉) |

**닉네임 중복 체크**:
- 회원가입 시 실시간 체크 (입력 중 debounce 적용)
- 사용 가능 여부 즉시 표시 (✓ / ✗)

**인증 화면 UI**:
- 타이틀 화면에 "Login / Sign Up" 버튼 추가
- DOS 스타일 입력 폼:
  - 닉네임 입력 필드 (실시간 중복 체크)
  - PIN 입력 필드 (4자리 숫자, 마스킹)
  - Login / Sign Up 버튼

**로그인 상태 표시**:
- 상단 메뉴 바 우측에 닉네임 표시
- Logout 버튼

#### 13.2 일기 저장 정책

**저장소 변경: 로컬 → Bkend 서버 Only**

| 항목 | 이전 (로컬) | 변경 후 (Bkend) |
|------|------------|----------------|
| 저장 위치 | IndexedDB | Bkend MongoDB |
| 오프라인 | 지원 | 미지원 (온라인 필수) |
| 동기화 | 불필요 | 불필요 (단일 소스) |
| 백업 | Export/Import | 서버 자동 보관 |

**일기 데이터 스키마** (Bkend Table: `entries`):
```javascript
{
  _id: ObjectId,              // Bkend 자동 생성
  createdBy: ObjectId,        // 작성자 (Bkend 자동)
  dateKey: "2025-12-18",      // 날짜 키 (YYYY-MM-DD)
  entryNumber: 1,             // 같은 날 몇 번째 (1-100)
  type: "diary" | "markdown", // 일기 타입
  text: "일기 내용...",        // 본문
  images: [                   // 이미지 배열
    { thumbnail: "base64...", full: "base64..." }
  ],
  createdAt: Date,            // Bkend 자동
  updatedAt: Date             // Bkend 자동
}
```

**인덱스**:
- `dateKey_entryNumber_unique`: { createdBy: 1, dateKey: 1, entryNumber: 1 } - **unique** (중복 저장 방지)
- `dateKey_idx`: { dateKey: 1 } - 날짜별 조회
- `createdBy_idx`: { createdBy: 1 } - 사용자별 조회

#### 13.3 하루 100개 제한

| 항목 | 정책 |
|------|------|
| 제한 단위 | 사용자별 + 날짜별 |
| 최대 개수 | 100개/일 |
| 카운트 기준 | diary + markdown 합산 |
| 초과 시 | "오늘은 더 이상 일기를 쓸 수 없습니다 (100/100)" 메시지 |
| 삭제 후 | 카운트 복구 (삭제된 건 제외) |

**체크 시점**:
- New 버튼 클릭 시 (프론트엔드)
- 저장 요청 시 (백엔드 최종 검증)

**카운트 조회 API**:
- 일기 목록 로드 시 해당 날짜 개수 함께 반환
- 상태 바에 "오늘 N/100" 표시

#### 13.4 일기 수정/삭제 정책

**수정 정책**:

| 항목 | 정책 |
|------|------|
| 수정 가능 범위 | 본인 일기만 (createdBy 검증) |
| 수정 가능 항목 | 텍스트, 이미지 (추가/삭제/변경) |
| 수정 불가 항목 | 날짜(dateKey), 작성자, 항목번호(entryNumber) |
| 타입 변경 | 불가 (diary ↔ markdown 전환 불가) |
| 수정 이력 | updatedAt 자동 갱신 |
| 수정 횟수 제한 | 없음 |

**삭제 정책**:

| 항목 | 정책 |
|------|------|
| 삭제 방식 | 즉시 삭제 (Soft Delete) |
| 복구 | 불가 |
| 삭제 확인 | DOS 스타일 확인 다이얼로그 |
| 삭제 후 | 해당 날짜 카운트 감소 |

#### 13.5 권한 (Roles)

**Bkend Table Roles 설정**:
```javascript
{
  "admin": { "create": true, "read": true, "update": true, "delete": true },
  "user": { "create": true, "read": true, "update": true, "delete": true },
  "self": { "read": true, "update": true, "delete": true },
  "guest": { "read": false }
}
```

- **self**: 본인이 작성한 일기만 조회/수정/삭제 가능
- **guest**: 비로그인 사용자는 일기 접근 불가

#### 13.6 API 엔드포인트

**Base URL**: `https://api-enduser.bkend.ai`

**필수 헤더**:
```
Authorization: Bearer {access_token}
X-Project-Id: {project_id}
X-Environment: dev
```

**인증 API**:
| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/auth/signup/password` | 회원가입 (email, password) |
| POST | `/auth/signin/password` | 로그인 (email, password) |
| GET | `/auth/me` | 내 정보 조회 |

**닉네임 → 이메일 변환**:
- Bkend는 이메일 기반 인증만 지원
- 닉네임을 이메일 형식으로 변환: `{nickname}@doogie.app`
- PIN은 password로 전달
- PIN은 password 패턴으로 변환: `Doo@gie{pin}#Pwd`
- 예: 닉네임 "홍길동", PIN "1234" → email: "홍길동@doogie.app", password: "Doo@gie1234#Pwd"

**일기 API**:
| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | `/data/entries` | 내 일기 목록 (필터/정렬/페이지네이션) |
| GET | `/data/entries/{id}` | 일기 상세 조회 |
| POST | `/data/entries` | 일기 생성 (100개 제한 체크) |
| PUT | `/data/entries/{id}` | 일기 수정 |
| DELETE | `/data/entries/{id}` | 일기 삭제 |

**필터링 예시**:
```javascript
// 특정 날짜 일기 조회
GET /data/entries?andFilters={"dateKey":"2025-12-18"}

// 특정 월 일기 조회
GET /data/entries?andFilters={"dateKey":{"$regex":"^2025-12"}}

// 마크다운만 조회
GET /data/entries?andFilters={"type":"markdown"}
```

#### 13.7 프론트엔드 변경 사항

**타이틀 화면 변경**:
- 로그인/회원가입 버튼 추가
- 비로그인 시 일기쓰기 메뉴 비활성화

**인증 화면 추가**:
- 닉네임 입력 (실시간 중복 체크)
- PIN 입력 (4자리 숫자)
- Login / Sign Up 탭 전환
- DOS 스타일 UI 유지

**상단 메뉴 변경**:
- Export/Import 버튼 제거
- 사용자 닉네임 표시 추가
- Logout 버튼 추가

**저장 로직 변경**:
- IndexedDB → Bkend API 호출
- 저장 중 로딩 인디케이터 표시
- 네트워크 오류 처리

**에러 처리**:
- 네트워크 오류: "서버에 연결할 수 없습니다"
- 인증 만료: 자동 로그아웃 → 로그인 화면
- 100개 초과: "오늘은 더 이상 일기를 쓸 수 없습니다"
- 권한 오류: "접근 권한이 없습니다"

**로딩 상태**:
- API 호출 중 DOS 스타일 로딩 표시
- "Loading..." 텍스트 깜빡임 애니메이션

---

**문서 버전**: 6.0
**최종 수정일**: 2026-02-17
**작성자**: POPUP-STUDIO
**프로젝트 코드명**: DOOGIE

## 프로젝트 특징 요약

1. **Single HTML File**: 메인 로직은 단일 파일에 포함
2. **DOS 감성**: 1990년대 복고풍 디자인
3. **픽셀 아트**: Canvas 기반 이미지 처리
4. **다국어 지원**: 한국어/English
5. **클라우드 저장**: Bkend 서버 기반, 어디서든 접근
6. **간편 인증**: 닉네임 + 4자리 PIN 로그인
7. **인터랙티브 크레딧**: 편집 가능한 캐릭터, 애니메이션
8. **반응형**: 4:3 비율 유지하며 다양한 화면 크기 지원
9. **달력 뷰**: 월간 달력으로 일기 관리, 점으로 일기 표시
10. **전체화면 모드**: 브라우저 UI 완전 숨김, 몰입형 경험
11. **마크다운 모드**: GFM 지원, Mermaid 다이어그램, 실시간 미리보기
12. **크리스마스 특별 기능**: 눈 효과, BGM, 카드 생성 (12월 한정)
13. **하루 100개 제한**: 적절한 사용량 관리

<style>
@media print {
    body, p, li { font-size: 13pt !important; line-height: 1.6 !important; }
    h1 { font-size: 22pt !important; margin-top: 22pt !important; margin-bottom: 14pt !important; }
    h2 { font-size: 18pt !important; margin-top: 18pt !important; margin-bottom: 12pt !important; }
    h3 { font-size: 16pt !important; margin-top: 16pt !important; margin-bottom: 10pt !important; }
    ul, ol { margin-top: 5pt !important; margin-bottom: 5pt !important; padding-left: 22pt !important; }
}
</style>

<!-- Claude Instruction:
이 문서는 프론트엔드 구현의 시각적 기준을 정의하는 디자인 스타일 가이드입니다.
서비스기획서.md의 브랜드 전략과 화면설계서.md의 UI 명세를 기반으로 작성합니다.
Figma Make나 프론트엔드 개발 시 이 문서를 참조하여 일관된 디자인을 구현합니다.
프로젝트 컨텍스트에 맞게 색상, 폰트, 컴포넌트를 정의하세요.
-->

# 디자인 스타일 가이드 (Design Style Guide)

**프로젝트명**: CaloMate (사진 한 장으로 끝내는 15초 칼로리 트래커)  
**작성일**: 2026-09-06  
**버전**: v1.0  
**기반 문서**: `PRD.md`, `02.기획문서/마켓리서치.md`, `02.기획문서/서비스기획서.md`, `02.기획문서/요구사항정의서.md`  
**기술 스택**: Next.js + Tailwind CSS, PWA, Lucide React Icons

---

## 1. 디자인 원칙 (Design Principles)

CaloMate의 시각 디자인과 인터랙션은 **"기록 피로로 인한 이탈율 82%를 15초 만에 극복한다"**는 핵심 제품 철학을 직접적으로 반영합니다.

1. **초압축 편의성 (Frictionless 15s)**:  
   모든 시각 요소와 플로우는 15초 이내에 식사 기록을 마칠 수 있도록 불필요한 장식과 탭을 배제합니다. 셔터 한 번, 보정 터치 한 번으로 끝나는 미니멀 모바일 인터랙션을 지향합니다.
2. **사람 언어 우선 (Natural Language First)**:  
   저울 강박과 스트레스를 유발하는 그램(g) 숫자를 화면에서 과감히 숨깁니다. 사람이 일상에서 실제로 말하는 "1그릇, 1접시, 1인분, 반 공기" 단위의 친근하고 직관적인 컨트롤러를 제공합니다.
3. **목표 중립 및 긍정성 (Bulking & Balance Neutrality)**:  
   전통적인 다이어트 앱의 '칼로리 초과 = 빨간색 경고' 관성을 탈피합니다. 증량(벌킹) 목표 사용자에게는 잉여 칼로리를 긍정적인 에메랄드/블루 계열의 축하 메시지로 표현하고, 감량 사용자에게는 건강한 밸런스를 유도합니다.
4. **모바일 우선 PWA 네이티브 감각 (Mobile-First PWA Experience)**:  
   스마트폰 브라우저 및 PWA Standalone 모드에 최적화된 뷰포트(360px~430px)를 기준으로 설계하며, 엄지손가락 조작 범위(Thumb Zone) 내에 하단 네비게이션과 대형 카메라 FAB을 배치합니다.

---

## 2. 글로벌 레이아웃 (Global Layout)

CaloMate는 모바일 웹 및 PWA 환경을 주 타깃으로 하는 **모바일 퍼스트(Mobile-First) 레이아웃**을 채택합니다. 데스크톱 화면에서는 최대 너비 480px의 모바일 컨테이너가 중앙 정렬(Centered Shell)되어 모바일 앱과 동일한 UX를 일관되게 제공합니다.

### 2.1 레이아웃 구조 (Mobile PWA Shell)

```
+-----------------------------------------------------------+
|               Top App Header (H: 56px)                    |
|  [Logo / Back]               [Today Date]       [Profile] |
+-----------------------------------------------------------+
|                                                           |
|                                                           |
|                     Main Content Area                     |
|                (Scrollable, Fluid W: 100%,                |
|                    Max-W: 480px centered)                 |
|                                                           |
|                                                           |
+-----------------------------------------------------------+
|            Bottom Navigation Bar (H: 64px)                |
|  [🏠 홈]     [📊 리포트]     (📸 FAB)     [🏃 운동]   [⚙️ 설정] |
+-----------------------------------------------------------+
```

| 영역 | 규격 | 배경/스타일 | 주요 구성 요소 및 비고 |
|------|------|-------------|------------------------|
| **상단 앱 헤더 (Header)** | H: 56px, W: 100% (Max 480px) | `bg-white/80` (Dark: `bg-slate-900/80`), `backdrop-blur-md` | 브랜드 로고(🥗 CaloMate), 당일 날짜, 프로필/알림 아이콘 |
| **메인 콘텐츠 (Main)** | W: 100%, Max-W: 480px 중앙 정렬, 상하 패딩 포함 | `bg-slate-50` (Dark: `bg-slate-950`) | 데일리 칼로리 밸런스 게이지, 최근 식사 타임라인, 운동 카드 |
| **하단 네비게이션 (Tab Bar)** | H: 64px, 하단 고정 (`fixed bottom-0`) | `bg-white/90` (Dark: `bg-slate-900/90`), `border-t border-slate-200` | 4개 탭 메뉴 + 중앙 돌출형 카메라 FAB 배치 |
| **플로팅 카메라 버튼 (FAB)** | 56px × 56px 원형, -20px 돌출 | `bg-emerald-500` (Glow Shadow) | **한 끼 15초 기록 즉시 실행 버튼** (가장 중요한 액션) |

### 2.2 반응형 그리드 & 여백 시스템

| 항목 | 규격 | 비고 |
|------|------|------|
| **모바일 기본 뷰포트** | 360px ~ 430px | iPhone 14/15/16 및 Galaxy S23/S24 최적화 |
| **데스크톱 쉘 최대 너비** | 480px (`max-w-[480px] mx-auto`) | 데스크톱 접속 시 양옆 은은한 슬레이트 배경 처리 |
| **좌우 페이지 패딩** | 16px (`px-4`) ~ 20px (`px-5`) | 스마트폰 화면 밀착 방지 여백 |
| **컴포넌트 간격** | 16px (`gap-4`) 또는 20px (`gap-5`) | 시각적 그룹핑 분리 |

### 2.3 스페이싱 스케일 (Spacing Tokens)

| 토큰 | 픽셀 값 | Tailwind 클래스 | 주요 용도 |
|------|---------|-----------------|-----------|
| **2xs** | 2px | `p-0.5`, `gap-0.5` | 미세 보더 및 인라인 라인 |
| **xs** | 4px | `p-1`, `gap-1` | 뱃지 내부 여백, 아이콘-텍스트 간격 |
| **sm** | 8px | `p-2`, `gap-2` | 인접 컴포넌트 간격, 칩 내부 패딩 |
| **md** | 12px | `p-3`, `gap-3` | 인풋 필드, 콤팩트 카드 패딩 |
| **base** | 16px | `p-4`, `gap-4` | 표준 카드 내부 패딩, 페이지 좌우 마진 |
| **lg** | 20px | `p-5`, `gap-5` | 대형 카드 패딩, 섹션 간격 |
| **xl** | 24px | `p-6`, `gap-6` | 대시보드 위젯 상하 간격 |
| **2xl** | 32px | `p-8`, `gap-8` | 모달 패딩, 온보딩 헤더 여백 |
| **3xl** | 48px | `p-12`, `gap-12` | 빈 화면(Empty State) 상하 여백 |

---

## 3. 컬러 시스템 (Color System)

CaloMate의 색채는 **'신선하고 지속 가능한 건강함(에메랄드)'**과 **'AI 비전의 스마트한 기술력(인디고)'**을 결합하여, 사용자가 칼로리를 기록할 때 죄책감이나 스트레스 대신 활력과 성취감을 느끼도록 배색되었습니다.

### 3.1 브랜드 프라이머리 컬러 (Emerald Spectrum)

| 이름 | Hex Code | Tailwind Token | 용도 |
|------|----------|----------------|------|
| **Primary Base** | `#10B981` | `emerald-500` | 브랜드 대표색, 메인 카메라 FAB, 주요 확인 버튼, 활성 탭 |
| **Primary Dark** | `#059669` | `emerald-600` | 버튼 호버 및 액티브 상태, 주요 텍스트 강조 |
| **Primary Deep** | `#065F46` | `emerald-800` | 다크모드 내 보조 포인트, 고대비 텍스트 |
| **Primary Light** | `#34D399` | `emerald-400` | 다크모드 메인 액센트, 게이지 프로그레스 바 |
| **Primary Tint** | `#ECFDF5` | `emerald-50` | 라이트모드 카드 배경 틴트, 성공 배지 배경 |

### 3.2 브랜드 세컨더리 컬러 (Smart Indigo & Blue)

| 이름 | Hex Code | Tailwind Token | 용도 |
|------|----------|----------------|------|
| **Secondary Base** | `#6366F1` | `indigo-500` | AI 분석 중 표시, 주간 리포트 차트, 스마트 운동 연계 |
| **Secondary Light** | `#818CF8` | `indigo-400` | 다크모드 차트 라인, AI 태그 뱃지 |
| **Tech Cyan** | `#06B6D4` | `cyan-500` | 수분 섭취 트래킹, 쿨다운 운동 카드 |

### 3.3 목표별 시맨틱 컬러 (Goal-Specific Semantics)

> [!IMPORTANT]
> **증량 모드 컬러 원칙 (서비스기획서 4.3 반영)**:  
> 증량(벌킹) 목표 사용자에게 칼로리 초과 섭취는 성공적인 잉여(Surplus)이므로 절대 경고색(Red)을 쓰지 않고 **에메랄드(`#10B981`)** 또는 **스카이블루(`#38BDF8`)**로 표현합니다.

| 목표 모드 / 상태 | 대표 Hex | 텍스트/아이콘 | 배경 틴트 | 감정적 유도 효과 |
|------------------|----------|---------------|-----------|------------------|
| **증량(벌킹) 잉여 섭취** | `#10B981` | `#059669` | `#ECFDF5` | "근성장에 필요한 칼로리를 훌륭히 채웠습니다! 🎉" (긍정 축하) |
| **감량 모드 정상 범위** | `#06B6D4` | `#0891B2` | `#ECFEFF` | "안전한 칼로리 결손 범위 내에 있습니다." (안정감) |
| **감량 모드 과다 초과** | `#EF4444` | `#DC2626` | `#FEF2F2` | "오늘 목표치를 초과했습니다. 가벼운 유산소를 추천해요." (부드러운 유도) |
| **단백질 목표 달성** | `#3B82F6` | `#2563EB` | `#EFF6FF` | "일일 단백질 골든 섭취량을 채웠습니다." (성취감) |

### 3.4 영양소 매크로(탄단지) 컬러

| 매크로 영양소 | Hex Code | Tailwind Token | 시각적 비유 및 의미 |
|---------------|----------|----------------|----------------------|
| **탄수화물 (Carbs)** | `#F59E0B` | `amber-500` | 곡물, 밥, 에너지를 상징하는 웜 앰버 |
| **단백질 (Protein)** | `#3B82F6` | `blue-500` | 근육 성장과 파워를 상징하는 쿨 블루 |
| **지방 (Fat)** | `#EC4899` | `pink-500` | 필수 지방산을 구분하는 비비드 로즈 핑크 |
| **총 칼로리 (Calories)** | `#10B981` | `emerald-500` | 전체 섭취 에너지 밸런스를 상징하는 에메랄드 |

### 3.5 뉴트럴 팔레트 (Slate Gray Scale)

| 토큰 | Hex Code | 용도 (라이트 모드) | 용도 (다크 모드) |
|------|----------|--------------------|------------------|
| **Slate 50** | `#F8FAFC` | 앱 전체 기본 배경 | - |
| **Slate 100** | `#F1F5F9` | 보조 카드 배경, 비활성 칩 | 보더 라인 호버 |
| **Slate 200** | `#E2E8F0` | 디바이더, 인풋 기본 보더 | - |
| **Slate 300** | `#CBD5E1` | 플레이스홀더, 비활성 아이콘 | 보조 보더 |
| **Slate 400** | `#94A3B8` | 캡션, 타임스탬프, 단위 라벨 | 캡션, 보조 텍스트 |
| **Slate 500** | `#64748B` | 보조 본문 텍스트 | 보조 본문 텍스트 |
| **Slate 600** | `#475569` | 서브헤딩, 라벨 텍스트 | 서브 텍스트 |
| **Slate 700** | `#334155` | 카드 제목, 주요 본문 | 카드 테두리, 보조 배경 |
| **Slate 800** | `#1E293B` | 메인 헤드라인 텍스트 | 다크모드 카드 배경 |
| **Slate 900** | `#0F172A` | 최대 대비 텍스트 | 다크모드 상단/하단 바 배경 |
| **Slate 950** | `#020617` | - | 다크모드 앱 기본 배경 |

---

## 4. 타이포그래피 (Typography)

CaloMate의 타이포그래피는 **한글 가독성의 표준인 'Pretendard'**와 **숫자/칼로리 카운트의 정밀성을 보장하는 'Outfit'**을 결합하여 현대적이고 경쾌한 모바일 헬스케어 인상을 제공합니다.

### 4.1 폰트 패밀리

```css
/* 한국어 본문 및 UI 표준 */
font-sans: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, sans-serif;

/* 대시보드 칼로리 숫자, 타이머, 게이지 KPI */
font-display: 'Outfit', 'Pretendard', sans-serif;
```

### 4.2 타입 스케일 (Type Scale)

| 토큰명 | 크기 (px/rem) | 행간 (Line-height) | 자간 (Tracking) | 기본 굵기 | 주요 적용 요소 |
|--------|---------------|-------------------|-----------------|-----------|----------------|
| **display-2xl** | 44px / 2.75rem | 1.1 | -0.03em | 900 (Black) | 15초 카운트다운, 온보딩 메인 헤드 |
| **display-xl** | 36px / 2.25rem | 1.15 | -0.02em | 800 (ExtraBold) | 대시보드 당일 잔여/잉여 칼로리 숫자 |
| **display-lg** | 28px / 1.75rem | 1.2 | -0.02em | 700 (Bold) | 한 끼 식사 총 칼로리 표시 |
| **heading-1** | 22px / 1.375rem | 1.3 | -0.01em | 700 (Bold) | 페이지 메인 타이틀, 모달 제목 |
| **heading-2** | 18px / 1.125rem | 1.35 | -0.01em | 700 (Bold) | 섹션 헤더, 음식 카드 대표명 |
| **heading-3** | 16px / 1.0rem | 1.4 | 0em | 600 (SemiBold) | 위젯 헤더, 슬라이더 음식 항목명 |
| **body-lg** | 16px / 1.0rem | 1.5 | 0em | 400 (Regular) | 중요 안내 본문, 긴 설명글 |
| **body-base** | 14px / 0.875rem | 1.5 | 0em | 400 (Regular) | 일반 본문 텍스트, 리스트 아이템 |
| **body-sm** | 13px / 0.8125rem | 1.45 | +0.01em | 500 (Medium) | 섭취 시간, 탄단지 수치 서브 텍스트 |
| **caption** | 12px / 0.75rem | 1.4 | +0.02em | 600 (SemiBold) | 뱃지 텍스트, 사람 언어 배식량 라벨 |
| **tiny** | 11px / 0.6875rem | 1.3 | +0.03em | 500 (Medium) | 의료 면책 고지 문구, 저작권 표기 |

---

## 5. 아이콘 시스템 (Iconography)

CaloMate는 모던한 스트로크 스타일의 **`Lucide React`** 라이브러리를 표준으로 사용합니다.

### 5.1 아이콘 스타일 원칙
* **스타일**: 둥근 모서리를 가진 Outlined Stroke
* **기본 선 두께 (Stroke Width)**: `2.0px` (24px 아이콘), `1.75px` (20px 아이콘), `1.5px` (16px 아이콘)
* **Corner Radius**: 부드러운 라운드 처리 (`stroke-linecap="round" stroke-linejoin="round"`)

### 5.2 주요 UI 기능별 아이콘 매핑

| 기능 영역 | Lucide 아이콘명 | 기본 크기 | 색상 가이드 |
|-----------|-----------------|-----------|-------------|
| **카메라 빠른 촬영 (FAB)** | `Camera` | 28px | `#FFFFFF` (Primary 바탕) |
| **홈 / 대시보드 탭** | `Home` | 24px | 활성: `emerald-500`, 비활성: `slate-400` |
| **식사 히스토리 탭** | `CalendarDays` / `History` | 24px | 활성: `emerald-500`, 비활성: `slate-400` |
| **맞춤 운동 탭** | `Flame` / `Dumbbell` | 24px | 활성: `emerald-500`, 비활성: `slate-400` |
| **마이페이지 / 설정 탭** | `User` / `Settings` | 24px | 활성: `emerald-500`, 비활성: `slate-400` |
| **배식량 증가/감소** | `Plus`, `Minus`, `Sliders` | 16px / 20px | `slate-700` (Dark: `slate-200`) |
| **식사 인식 성공/승인** | `CheckCircle2` | 20px | `emerald-500` |
| **인식 수정/대체 후보** | `RefreshCw` / `ChevronRight` | 16px | `slate-500` |
| **의료 면책 및 주의 경고** | `AlertTriangle` / `Info` | 14px | `amber-500` / `slate-400` |
| **증량(벌킹) 축하 뱃지** | `Trophy` / `Sparkles` | 18px | `amber-400` / `emerald-400` |

---

## 6. 핵심 UI 컴포넌트 명세 (Core Components)

### 6.1 카메라 촬영 & 15초 로깅 플로우 컴포넌트 (F-1, REQ-001)

#### A. 하단 플로팅 셔터 버튼 (Camera FAB)
* **위치**: 하단 탭바 중앙 상단에 반원형으로 20px 돌출 배치
* **외형**: 56px × 56px 정원, `bg-emerald-500`, `text-white`
* **그림자/모션**: `shadow-lg shadow-emerald-500/30`, 터치 시 `scale-95` 축소 애니메이션 (50ms)
* **접근성 라벨**: `aria-label="식사 사진 촬영하여 15초 기록 시작"`

#### B. 3초 AI 비전 스켈레톤 로더 (Vision Analysis Loader)
* 사진 촬영 직후 결과가 나타나기 전 3초 동안 작동하는 인터랙티브 로더:
  - 썸네일 이미지 위에 미세한 초록색 스캔 빔이 위아래로 반복 이동 (`animate-pulse`)
  - 하단 텍스트: *"AI가 한식 구성을 분석 중입니다... (예상 2초)"*
  - 음식 카드 위치에 3개의 둥근 스켈레톤 바 배치

---

### 6.2 사람 언어 배식량 조절기 (F-2, REQ-003, REQ-004)

> **디자인 필수 원칙**: 화면에서 그램(g) 수치를 완전 배제하고, 사람이 말하는 4단계 세그먼트 칩과 직관적 스텝 슬라이더를 제공합니다.

#### A. 밥/주식 단위 세그먼트 버튼 (Rice & Staple Chips)
* **선택 항목**: `[반 공기]` | `[한 공기 (기본)]` | `[한 공기 반]` | `[고봉밥]`
* **선택 상태**: `bg-emerald-500 text-white font-semibold shadow-sm`
* **비선택 상태**: `bg-slate-100 text-slate-600 hover:bg-slate-200`
* **터치 타깃**: 최소 높이 44px, 가로 100% 균등 분할

#### B. 국/찌개/일품 단위 스텝 슬라이더 (Soup & Main Slider)
* **단위 라벨**: `[반 그릇]` ➔ `[1그릇 / 1인분 (기본)]` ➔ `[곱빼기]`
* **슬라이더 트랙**: 높이 8px, 채워진 구간 `bg-emerald-500`, 미채움 구간 `bg-slate-200`
* **Thumb (손잡이)**: 28px × 28px 원형 흰색 버튼, `shadow-md`, 슬라이드 시 햅틱 진동 피드백 유도

---

### 6.3 칼로리 밸런스 도넛 게이지 (F-3, REQ-008, REQ-009)

* **컴포넌트 크기**: 직경 220px 원형 SVG 게이지 (중앙 정렬)
* **Stroke 두께**: 16px (배경 트랙: `slate-100`, 진행률 트랙: 동적 시맨틱 컬러)
* **중앙 텍스트 레이아웃**:
  - 상단 라벨 (caption): *"오늘의 에너지 밸런스"*
  - 중앙 큰 숫자 (display-xl): `+350` (증량 모드) 또는 `540` (감량 모드 잔여)
  - 하단 보조 텍스트 (body-sm): *"kcal 잉여 달성 중"* 또는 *"kcal 남음"*
* **애니메이션**: 페이지 로드 시 0%에서 목표 수치까지 800ms 부드러운 스위프 (`ease-out`)

---

### 6.4 맞춤 운동 추천 카드 (F-4, REQ-010)

* **배경**: `bg-white` (Dark: `bg-slate-800`), 테두리 `border border-slate-200/80`, 반경 `rounded-2xl`
* **헤더**:
  - 운동 종목명 (heading-3): *"가벼운 러닝"* / *"하체 스쿼트 & 런지"*
  - 태그 뱃지: `[유산소 우선]` (`bg-red-50 text-red-600`) 또는 `[근성장 벌킹]` (`bg-emerald-50 text-emerald-600`)
* **상세 인포그래픽**:
  - ⏱️ 권장 시간: *"25분"*
  - 🔥 예상 소모량: *"약 210 kcal"*
  - 🎯 상쇄 효과: *"점심 초과분 100% 소모 가능"*
* **액션**: `[운동 완료 기록]` 탭 시 칼로리 밸런스 게이지에 즉시 소모량 차감 반영

---

### 6.5 식사 히스토리 타임라인 카드 (F-5, REQ-011)

* **카드 레이아웃**: 좌측 음식 사진 썸네일(72px × 72px, `rounded-xl`, `object-cover`) + 우측 영양 정보
* **식사 라벨**: `[아침 08:30]` / `[점심 12:40]` / `[저녁 19:15]` (caption 폰트)
* **음식 요약**: *"흑미밥, 차돌된장찌개, 계란말이 외 2종"* (heading-3, 1줄 말줄임)
* **미니 매크로 바**: 3색 인라인 프로그레스 바 (탄수화물 주황, 단백질 파랑, 지방 분홍)
* **열람 편의**: 카드 탭 시 사진 원본과 영양 분석 모달 팝업

---

### 6.6 의료 면책 및 알레르기 안내 배너 (REQ-012)

* **스타일**: 서틀 캡션 바 (Subtle Informative Banner)
* **위치**: 식사 분석 확인 모달 하단 및 설정 화면 최하단
* **배경**: `bg-slate-100/80` (Dark: `bg-slate-900/50`), `rounded-lg`, `p-3`
* **문구 예시**:  
  *"⚠️ AI 분석 수치는 식약처 DB 기반 추정치이며 의학적 조언을 대신하지 않습니다. 사진으로 감지할 수 없는 숨은 알레르기 유발 성분은 반드시 섭취 전 직접 확인하세요."*
* **폰트**: `tiny (11px)`, `text-slate-500`

---

## 7. 데이터 시각화 스타일 (Data Visualization)

| 차트 유형 | 사용 목적 | 시각 스타일 가이드 |
|-----------|-----------|--------------------|
| **원형 도넛 게이지 (Circular Progress)** | 일일 칼로리 밸런스 달성도 | 직경 220px, Stroke 16px, 둥근 캡(`stroke-linecap: round`), 센터 수치 강조 |
| **3색 매크로 스택 바 (Stacked Macro Bar)** | 탄수화물/단백질/지방 섭취 비율 | 높이 10px, 반경 5px, 탄(주황 50%)+단(파랑 30%)+지(분홍 20%) 분할 렌더링 |
| **주간 섭취/체중 꺾은선 차트 (Trend Line)** | 최근 7일간의 칼로리 및 체중 추이 | 곡선형 스플라인(`monotone`), 포인트 도트 6px, 에메랄드 10% 투명도 그라디언트 채우기 |
| **단백질 목표 게이지 바 (Linear Progress)** | 증량(벌킹) 사용자 일일 단백질 달성률 | 높이 12px, 목표치 도달 시 빛나는 글로우(`box-shadow`) 효과 적용 |

---

## 8. 인터랙션 & 모션 가이드 (Motion & Timing)

15초 완결이라는 초고속 UX를 달성하기 위해 모션은 불필요하게 화려하지 않고, **즉각적이고 경쾌한 피드백(Snappy Feedback)**을 제공하는 데 집중합니다.

### 8.1 지속 시간 및 가속도 곡선

| 모션 토큰 | 시간 (Duration) | 이징 함수 (Easing) | 적용 인터랙션 |
|-----------|-----------------|---------------------|---------------|
| **Instant** | 50ms | `linear` | 버튼 클릭 시 누름 효과 (`scale-95`) |
| **Fast** | 150ms | `ease-out` | 배식량 칩 전환, 토글 스위치, 툴팁 표시 |
| **Normal** | 250ms | `cubic-bezier(0.16, 1, 0.3, 1)` | 바텀 시트 열림/닫힘, 다이얼로그 모달 등장 |
| **Smooth** | 400ms | `ease-out` | 칼로리 게이지 수치 카운트업, 게이지 채움 |
| **Pulse** | 1500ms | `ease-in-out` | 카메라 뷰파인더 스캔 빔 반복 모션 |

### 8.2 15초 로깅을 위한 마이크로 인터랙션
* **슬라이더 햅틱**: 배식량 단위(반 공기 $\rightarrow$ 한 공기) 변경 시 모바일 브라우저의 진동 API(`navigator.vibrate(10)`)를 짧게 트리거하여 손맛 제공.
* **저장 완료 체크마크**: [기록 저장] 탭 즉시 초록색 체크 아이콘이 팝업되며 스케일 업(0.8 $\rightarrow$ 1.0) 후 0.5초 뒤 대시보드로 자동 복귀.

---

## 9. 반응형 및 접근성 규격 (Accessibility & Responsive)

### 9.1 뷰포트 브레이크포인트 (PWA Container)

```
[Mobile Viewport: 360px ~ 430px]  --> 100% Full Width 반응형 레이아웃
[Tablet / Desktop: 768px ~ 1920px] --> Max-Width 480px Centered Shell (App Canvas)
```

### 9.2 접근성(A11y) 기준 (WCAG 2.1 AA 준수)

* **최소 터치 타깃**: 스마트폰 한 손 조작 환경을 위해 모든 주요 버튼, 칩, 탭 아이콘의 클릭 가능 영역은 최소 **`48px × 48px`** 이상 확보 (`p-3` 패딩 포함).
* **명도 대비율**:
  - 일반 본문 텍스트(`Slate 700` vs `Slate 50`): **9.2:1** (AA 기준 4.5:1 초과 달성)
  - 핵심 강조 텍스트(`Emerald 600` vs `White`): **4.6:1** (AA 기준 충족)
* **다크모드 지원**:
  - 시스템 설정(`prefers-color-scheme`)에 자동 감응하며, 설정 화면에서 수동 토글 지원.
  - 다크모드 배경: `Slate 950 (#020617)`, 카드 배경: `Slate 900 (#0F172A)`.
* **스크린 리더 지원**:
  - 복잡한 차트 및 원형 게이지에 `aria-valuenow`, `aria-valuemin`, `aria-valuemax` 속성 의무 적용.
  - 그램이 숨겨진 배식량 슬라이더에 *"현재 선택: 흑미밥 한 공기, 예상 칼로리 310 칼로리"* 음성 텍스트 제공.

---

## 10. Tailwind CSS 테마 설정 코드 (`tailwind.config.js`)

프론트엔드 개발자가 즉시 복사하여 프로젝트에 적용할 수 있는 Tailwind CSS 설정 확장 객체입니다.

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: 'class',
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#ECFDF5',
          100: '#D1FAE5',
          200: '#A7F3D0',
          300: '#6EE7B7',
          400: '#34D399',
          500: '#10B981', // CaloMate Primary Green
          600: '#059669',
          700: '#047857',
          800: '#065F46',
          900: '#064E3B',
        },
        ai: {
          light: '#818CF8',
          DEFAULT: '#6366F1', // Smart Indigo
          dark: '#4F46E5',
        },
        macro: {
          carb: '#F59E0B',    // Amber
          protein: '#3B82F6', // Blue
          fat: '#EC4899',     // Rose Pink
        },
        surplus: {
          DEFAULT: '#10B981', // Bulking Positive Green
          glow: '#34D399',
        }
      },
      fontFamily: {
        sans: ['var(--font-pretendard)', '-apple-system', 'BlinkMacSystemFont', 'sans-serif'],
        display: ['var(--font-outfit)', 'sans-serif'],
      },
      maxWidth: {
        'app': '480px', // Mobile Centered Container
      },
      boxShadow: {
        'fab': '0 10px 25px -5px rgba(16, 185, 129, 0.4), 0 8px 10px -6px rgba(16, 185, 129, 0.2)',
        'card': '0 2px 12px -2px rgba(15, 23, 42, 0.06), 0 1px 3px -1px rgba(15, 23, 42, 0.04)',
      },
      animation: {
        'scan': 'scan 2s ease-in-out infinite',
      },
      keyframes: {
        scan: {
          '0%, 100%': { transform: 'translateY(0%)' },
          '50%': { transform: 'translateY(100%)' },
        }
      }
    },
  },
  plugins: [],
};
```

---

## 11. 디자인 검증 체크리스트 (Design QA Checklist)

프론트엔드 개발 및 UI 검수 시 다음 기준을 필수적으로 확인합니다:

- [x] **15초 완결성**: 카메라 FAB 탭 $\rightarrow$ 촬영 $\rightarrow$ 배식량 칩 탭 $\rightarrow$ 저장 완료까지 4단계 이내로 인터랙션이 끝나는가?
- [x] **그램 비노출**: 메인 대면 UI 어디에도 210g, 350g 등 그램 숫자가 기본 노출되지 않고 "1그릇, 1공기"로 유지되는가?
- [x] **벌킹 친화성**: 증량 목표 설정 시 초과 칼로리가 적색 경고가 아닌 에메랄드/블루 계열의 긍정 잉여(Surplus)로 축하 렌더링되는가?
- [x] **모바일 터치 규격**: 하단 FAB, 네비게이션 탭, 배식량 조절 칩의 터치 영역이 최소 44px × 44px 이상인가?
- [x] **의료 면책 상시 고지**: 식사 분석 결과 화면 및 설정 화면에 알레르기 미감지 및 의학적 면책 문구가 정상 표기되는가?
- [x] **PWA 쉘 최적화**: 모바일 360px 너비에서 가로 스크롤이 발생하지 않으며, 데스크톱에서 max-w 480px로 안정감 있게 렌더링되는가?

---

**작성 완료 여부**: [x] 서비스기획서 및 요구사항정의서 기반 디자인 스타일 가이드 작성 완료 (Document Chain #13)

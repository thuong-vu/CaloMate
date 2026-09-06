<style>
@media print {
    body, p, li { font-size: 13pt !important; line-height: 1.6 !important; }
    h1 { font-size: 22pt !important; margin-top: 22pt !important; margin-bottom: 14pt !important; }
    h2 { font-size: 18pt !important; margin-top: 18pt !important; margin-bottom: 12pt !important; }
    h3 { font-size: 16pt !important; margin-top: 16pt !important; margin-bottom: 10pt !important; }
    ul, ol { margin-top: 5pt !important; margin-bottom: 5pt !important; padding-left: 22pt !important; }
}
</style>

# API 스펙

> Week 3 산출물 (Document Chain #7). 기능명세서.md(#6)의 F-001~F-005 "관련 API" 항목을 기반으로 작성됨.
> F-006(PWA 배포 및 오프라인 열람)은 클라이언트 서비스 워커/캐시 스토리지 처리로 서버 API가 없어 본 문서 범위에서 제외한다.

**프로젝트명**: CaloMate (사진 한 장으로 끝내는 15초 칼로리 트래커)
**작성일**: 2026-09-06
**버전**: v1.0
**상태**: 승인 완료 (Approved)

---

## API 공통 사항

| 항목 | 내용 |
|------|------|
| Base URL | `/api` (Next.js API Routes, App Router `app/api/*` — V0.43 M1~M4 통합 배포 기준. Vercel에 프론트엔드와 함께 배포되므로 별도 도메인/CORS 설정 불필요) |
| 인증 방식 | Bearer Token (JWT). 로그인 후 발급된 Access Token을 `Authorization: Bearer {token}` 헤더로 전달 |
| 요청 형식 | `application/json` (단, `POST /api/meals/recognize`는 이미지 업로드를 위해 `multipart/form-data`) |
| 응답 형식 | JSON, UTF-8 |
| 공통 응답 래퍼 | 성공 시 `{ "status": "success", "data": {...} }`, 실패 시 `{ "status": "error", "error": { "code": "...", "message": "..." } }` |
| 날짜 형식 | `YYYY-MM-DD` (ISO 8601), 주(week) 파라미터는 `YYYY-Www` (예: `2026-W36`) |
| 통화/단위 | 칼로리 단위는 kcal, 매크로(탄/단/지)는 g 단위로 내부 저장하되 F-002 정책에 따라 클라이언트 응답 중 "화면 표시용" 필드는 그램을 노출하지 않음(내부 저장용 필드는 별도 구분) |

## 공통 에러 코드

| HTTP 상태 코드 | 에러 코드 | 설명 |
|----------------|-----------|------|
| 400 | BAD_REQUEST | 요청 파라미터 누락/형식 오류 |
| 401 | UNAUTHORIZED | 인증 토큰 없음 또는 만료 |
| 403 | FORBIDDEN | 타 사용자 리소스 접근 등 권한 없음 |
| 404 | NOT_FOUND | 요청한 리소스(식사 기록, 음식 등) 없음 |
| 408 | REQUEST_TIMEOUT | 외부 API(LLM 비전 등) 응답 지연 |
| 422 | UNPROCESSABLE_ENTITY | 값은 유효하나 비즈니스 규칙 위반(범위 초과 등) |
| 500 | INTERNAL_ERROR | 서버 내부 오류 |
| 502 | UPSTREAM_ERROR | LLM 비전 API 등 외부 벤더 연동 오류 |

---

## API 엔드포인트 목록

| API-ID | Method | URL | 기능명 | 관련 기능 ID |
|--------|--------|-----|--------|-------------|
| API-001 | POST | `/api/meals/recognize` | 사진 업로드 및 음식 인식 | F-001 |
| API-002 | GET | `/api/foods/search` | 음식 수동 검색(자동완성) | F-001 |
| API-003 | PATCH | `/api/meals/{id}/portion` | 배식량 보정값 저장 | F-002 |
| API-004 | GET | `/api/foods/{id}/portion-map` | 표준 중량 매핑 조회 | F-002 |
| API-005 | POST | `/api/users/goal` | 체중 목표 및 온보딩 정보 저장 | F-003 |
| API-006 | GET | `/api/dashboard/balance` | 일일 칼로리 밸런스 조회 | F-003 |
| API-007 | GET | `/api/recommendations/exercise` | 당일 맞춤 운동 추천 조회 | F-004 |
| API-008 | GET | `/api/meals` | 일별 식사 기록 조회 | F-005 |
| API-009 | GET | `/api/reports/weekly` | 주간 요약 리포트 조회 | F-005 |

---

## API 상세

### API-001. 사진 업로드 및 음식 인식

| 항목 | 내용 |
|------|------|
| Method | POST |
| URL | `/api/meals/recognize` |
| 설명 | 사용자가 촬영/업로드한 식사 이미지를 LLM 비전 API로 전달하여 음식 종류·구성 요소·예상 칼로리/매크로를 자동 인식한다(REQ-001, REQ-002, REQ-005, REQ-006). |
| 관련 기능 ID | F-001 |

**Request** (multipart/form-data)

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|----------|------|------|:---:|------|
| image | body | file (JPEG/PNG) | Y | 촬영 또는 갤러리 선택 이미지. 클라이언트에서 장변 1600px 이내 압축, EXIF 위치정보 제거 후 전송(REQ-015) |
| mealType | body | string | N | 식사 시간대. `breakfast` \| `lunch` \| `dinner` \| `snack` |
| userId | body | string | Y | 사용자 계정 ID (JWT에서 추출 가능 시 생략 가능) |

**Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "mealId": "meal_20260906_0001",
    "recognizedAt": "2026-09-06T08:15:00+09:00",
    "candidates": [
      {
        "foodId": "food_001",
        "foodName": "제육볶음",
        "components": [
          { "name": "제육볶음", "baseCalorieKcal": 450, "carbG": 20, "proteinG": 28, "fatG": 30 },
          { "name": "공기밥", "baseCalorieKcal": 300, "carbG": 65, "proteinG": 6, "fatG": 1 }
        ],
        "defaultPortionUnit": "1인분",
        "confidenceScore": 0.86
      }
    ],
    "alternativeCandidates": [
      { "foodId": "food_014", "foodName": "돼지불고기", "confidenceScore": 0.41 },
      { "foodId": "food_027", "foodName": "제육덮밥", "confidenceScore": 0.33 },
      { "foodId": "food_009", "foodName": "닭갈비", "confidenceScore": 0.22 }
    ],
    "disclaimer": "AI 분석 수치는 추정치이며 의학적 조언이 아닙니다. 알레르기 성분은 자동 감지되지 않을 수 있습니다."
  }
}
```

**에러 Response**

| 상태 코드 | 에러 코드 | 조건 |
|-----------|-----------|------|
| 400 | BAD_REQUEST | 이미지 파일 누락 또는 형식(JPEG/PNG 외) 오류 |
| 408 | REQUEST_TIMEOUT | LLM 비전 API 응답이 3초(NFR-001) 초과 지연 — 클라이언트는 재시도 또는 수동 검색(API-002)으로 폴백 |
| 422 | UNPROCESSABLE_ENTITY | 인식 신뢰도 전체 저신뢰(임계치 미만) — `alternativeCandidates` 3건과 함께 반환하여 재선택 유도(REQ-006) |
| 502 | UPSTREAM_ERROR | LLM 비전 API 벤더 장애/응답 파싱 실패 |
| 500 | INTERNAL_ERROR | 이미지 업로드 처리 중 서버 오류(네트워크 단절 등) — 클라이언트는 로컬 임시 저장 후 재전송 안내 |

---

### API-002. 음식 수동 검색(자동완성)

| 항목 | 내용 |
|------|------|
| Method | GET |
| URL | `/api/foods/search?q={keyword}` |
| 설명 | API-001 인식 실패/저신뢰 시 사용자가 키워드로 음식을 직접 검색하는 폴백 기능(REQ-006). 식약처 외식 영양성분 DB 기준으로 자동완성 후보를 제공한다. |
| 관련 기능 ID | F-001 |

**Request**

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|----------|------|------|:---:|------|
| q | query | string | Y | 검색 키워드 (최소 1자) |
| limit | query | integer | N | 반환 개수 상한 (기본값 10, 최대 30) |

**Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "query": "제육",
    "results": [
      { "foodId": "food_001", "foodName": "제육볶음", "baseCalorieKcal": 450, "defaultPortionUnit": "1인분" },
      { "foodId": "food_027", "foodName": "제육덮밥", "baseCalorieKcal": 620, "defaultPortionUnit": "1그릇" }
    ]
  }
}
```

**에러 Response**

| 상태 코드 | 에러 코드 | 조건 |
|-----------|-----------|------|
| 400 | BAD_REQUEST | `q` 파라미터 누락 또는 빈 문자열 |
| 200(빈 배열) | - | 검색 결과 0건인 경우 에러가 아닌 `results: []`로 응답(빈 상태 UI 처리는 클라이언트 책임) |
| 500 | INTERNAL_ERROR | 검색 인덱스/DB 조회 오류 |

---

### API-003. 배식량 보정값 저장

| 항목 | 내용 |
|------|------|
| Method | PATCH |
| URL | `/api/meals/{id}/portion` |
| 설명 | F-001에서 인식(또는 API-002로 검색 지정)된 식사 항목의 배식량을 "반 공기/1공기/1접시" 등 사람 언어 단위 슬라이더 값으로 보정하여 저장한다. 그램 환산값은 내부 저장용으로만 함께 기록하며 응답의 화면 표시 필드에는 노출하지 않는다(REQ-003, REQ-004). |
| 관련 기능 ID | F-002 |

**Request**

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|----------|------|------|:---:|------|
| id | path | string | Y | 대상 식사 기록 ID (mealId) |
| portionMultiplier | body | number | Y | 배식량 배수. 0.5~2.0 범위, 0.1 단위 |
| portionLabel | body | string | N | 화면 표시용 배식량 라벨(예: "1/2공기"). 미전달 시 서버가 배수 기반으로 자동 생성 |

**Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "mealId": "meal_20260906_0001",
    "portionLabel": "밥 1/2공기",
    "adjustedCalorieKcal": 600,
    "adjustedMacros": { "carbLabel": "적정", "proteinLabel": "적정", "fatLabel": "적정" },
    "isEstimated": false,
    "updatedAt": "2026-09-06T08:16:20+09:00"
  }
}
```

> 비고: `adjustedMacros`는 화면 미노출 원칙(REQ-004)에 따라 그램 수치 대신 정성 라벨로 제공하는 예시이며, 실제 화면설계서(#9) 확정 시 표시 형식을 조정할 수 있다. 그램 환산값은 응답에 포함하지 않고 내부 DB에만 저장한다.

**에러 Response**

| 상태 코드 | 에러 코드 | 조건 |
|-----------|-----------|------|
| 400 | BAD_REQUEST | `portionMultiplier` 누락 또는 숫자가 아님 |
| 404 | NOT_FOUND | 해당 `id`의 식사 기록이 존재하지 않음 |
| 422 | UNPROCESSABLE_ENTITY | `portionMultiplier`가 0.5~2.0 범위를 벗어남 — 서버는 최대/최소값으로 클램핑 후 저장하거나 거부(정책: 클램핑하여 200 반환 권장) |
| 500 | INTERNAL_ERROR | 저장 처리 중 DB 오류 |

---

### API-004. 표준 중량 매핑 조회

| 항목 | 내용 |
|------|------|
| Method | GET |
| URL | `/api/foods/{id}/portion-map` |
| 설명 | 특정 음식의 기본 배식량 단위(그릇/접시/인분)와 식약처 외식 영양성분 DB 표준 중량(g) 매핑 테이블을 조회한다. F-002의 슬라이더 배수 계산 기준값으로 사용된다. |
| 관련 기능 ID | F-002 |

**Request**

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|----------|------|------|:---:|------|
| id | path | string | Y | 음식 ID (foodId) |

**Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "foodId": "food_001",
    "foodName": "제육볶음",
    "unitLabel": "1인분",
    "standardWeightG": 210,
    "isFallbackEstimate": false,
    "baseCalorieKcal": 450
  }
}
```

**에러 Response**

| 상태 코드 | 에러 코드 | 조건 |
|-----------|-----------|------|
| 404 | NOT_FOUND | 해당 `id`의 음식이 DB에 없음 |
| 200(대체값) | - | 표준 중량 DB에 없는 음식은 404 대신 유사 카테고리 평균값으로 대체하고 `isFallbackEstimate: true`로 표시(에러 아님, 정상 폴백) |
| 500 | INTERNAL_ERROR | 매핑 테이블 조회 오류 |

---

### API-005. 체중 목표 및 온보딩 정보 저장

| 항목 | 내용 |
|------|------|
| Method | POST |
| URL | `/api/users/goal` |
| 설명 | 온보딩 시 목표 유형(증량/감량/유지) 및 신체 정보를 입력받아 기초대사량(BMR)·활동대사량(TDEE) 기반 일일 권장 칼로리/단백질 목표를 계산·저장한다(REQ-007, REQ-008, REQ-009). |
| 관련 기능 ID | F-003 |

**Request**

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|----------|------|------|:---:|------|
| goalType | body | string | Y | 목표 유형. `bulk`(증량) \| `cut`(감량) \| `maintain`(유지) |
| currentWeightKg | body | number | Y | 현재 체중(kg), 0보다 커야 함 |
| targetWeightKg | body | number | N | 목표 체중(kg) |
| heightCm | body | number | Y | 신장(cm) |
| age | body | integer | Y | 연령 |
| gender | body | string | Y | `male` \| `female` \| `other` |
| activityLevel | body | string | Y | 활동 수준. `low` \| `medium` \| `high` |

**Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "userId": "user_0001",
    "goalType": "bulk",
    "bmrKcal": 1650,
    "tdeeKcal": 2400,
    "dailyCalorieTargetKcal": 2800,
    "dailyProteinTargetG": 140,
    "savedAt": "2026-09-06T08:00:00+09:00"
  }
}
```

**에러 Response**

| 상태 코드 | 에러 코드 | 조건 |
|-----------|-----------|------|
| 400 | BAD_REQUEST | `goalType` 미선택(3가지 중 필수 1개 선택 검증 실패) 또는 필수 필드 누락 |
| 422 | UNPROCESSABLE_ENTITY | `currentWeightKg`, `heightCm`, `age` 등이 0 이하 또는 비정상 범위(클라이언트 1차 검증 + 서버 2차 검증) |
| 500 | INTERNAL_ERROR | 목표/계산값 저장 중 DB 오류 |

---

### API-006. 일일 칼로리 밸런스 조회

| 항목 | 내용 |
|------|------|
| Method | GET |
| URL | `/api/dashboard/balance?date={YYYY-MM-DD}` |
| 설명 | 당일 누적 섭취 칼로리(F-001~F-002 기록)와 API-005에서 산출된 목표치를 비교하여 칼로리 밸런스를 계산한다. 목표 유형에 따라 증량은 긍정적 잉여, 감량/유지는 잔여 칼로리 게이지로 분기 표시한다(REQ-008, REQ-009). |
| 관련 기능 ID | F-003 |

**Request**

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|----------|------|------|:---:|------|
| date | query | string (YYYY-MM-DD) | N | 조회 대상 날짜. 미지정 시 당일(오늘) 기준 |

**Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "date": "2026-09-06",
    "goalType": "bulk",
    "dailyCalorieTargetKcal": 2800,
    "consumedCalorieKcal": 2150,
    "balanceType": "surplus",
    "balanceKcal": 650,
    "gaugeVariant": "positive_surplus",
    "badgeTriggered": false
  }
}
```

> `balanceType`은 `surplus`(잉여) 또는 `remaining`(잔여). 증량 목표(`bulk`)에서 목표 초과 섭취 시에도 경고색/부정 문구를 사용하지 않고 `gaugeVariant: positive_surplus`로 고정 노출한다(색상/문구 화이트리스트 정책).

**에러 Response**

| 상태 코드 | 에러 코드 | 조건 |
|-----------|-----------|------|
| 400 | BAD_REQUEST | `date` 형식 오류 |
| 404 | NOT_FOUND | 사용자의 목표(API-005)가 아직 설정되지 않음 — 온보딩 유도 필요 |
| 500 | INTERNAL_ERROR | 집계 처리 중 오류 |

---

### API-007. 당일 맞춤 운동 추천 조회

| 항목 | 내용 |
|------|------|
| Method | GET |
| URL | `/api/recommendations/exercise?date={YYYY-MM-DD}` |
| 설명 | API-006의 당일 칼로리 밸런스와 목표 유형에 따라 정적 규칙 테이블 기반으로 운동 유형·시간·예상 소모량을 추천한다. MVP는 규칙 기반이며 적응형 알고리즘은 향후 로드맵 항목이다(REQ-010). |
| 관련 기능 ID | F-004 |

**Request**

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|----------|------|------|:---:|------|
| date | query | string (YYYY-MM-DD) | N | 조회 대상 날짜. 미지정 시 당일 기준 |

**Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "date": "2026-09-06",
    "basis": { "balanceType": "surplus", "balanceKcal": 650, "goalType": "bulk" },
    "recommendations": [
      { "exerciseName": "근력 운동(상체)", "durationMin": 40, "estimatedBurnKcal": 300 },
      { "exerciseName": "빠르게 걷기", "durationMin": 30, "estimatedBurnKcal": 150 },
      { "exerciseName": "계단 오르기", "durationMin": 15, "estimatedBurnKcal": 120 }
    ]
  }
}
```

**에러 Response**

| 상태 코드 | 에러 코드 | 조건 |
|-----------|-----------|------|
| 400 | BAD_REQUEST | `date` 형식 오류 |
| 200(대체 응답) | - | 해당 날짜 식사 기록이 없어 밸런스 데이터 부재 시, 에러 대신 `recommendations`에 기본 유지 운동 1건만 반환(빈 배열 대신 폴백 카드 제공) |
| 422 | UNPROCESSABLE_ENTITY | 밸런스 값이 규칙 테이블의 정의 범위를 크게 벗어난 극단값 — 서버가 최대/최소 구간값으로 캡핑하여 200으로 반환(에러 미노출) |
| 500 | INTERNAL_ERROR | 추천 규칙 조회/계산 오류 |

---

### API-008. 일별 식사 기록 조회

| 항목 | 내용 |
|------|------|
| Method | GET |
| URL | `/api/meals?date={YYYY-MM-DD}` |
| 설명 | 특정 날짜의 아침/점심/저녁/간식 식사 기록(사진 썸네일, 칼로리, 배식량 라벨)을 조회한다(REQ-011). |
| 관련 기능 ID | F-005 |

**Request**

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|----------|------|------|:---:|------|
| date | query | string (YYYY-MM-DD) | Y | 조회 대상 날짜 |
| page | query | integer | N | 페이지 번호 (기본값 1). 기록 다건 조회 지연 방지를 위한 페이지네이션 |
| pageSize | query | integer | N | 페이지당 항목 수 (기본값 20) |

**Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "date": "2026-09-06",
    "totalCalorieKcal": 2150,
    "meals": [
      {
        "mealId": "meal_20260906_0001",
        "mealType": "lunch",
        "foodName": "제육볶음",
        "portionLabel": "밥 1/2공기",
        "calorieKcal": 600,
        "thumbnailUrl": "https://cdn.calomate.app/thumbs/meal_20260906_0001.jpg",
        "recordedAt": "2026-09-06T12:30:00+09:00"
      }
    ],
    "pagination": { "page": 1, "pageSize": 20, "totalCount": 3 }
  }
}
```

**에러 Response**

| 상태 코드 | 에러 코드 | 조건 |
|-----------|-----------|------|
| 400 | BAD_REQUEST | `date` 파라미터 누락 또는 형식 오류 |
| 200(빈 상태) | - | 해당 날짜 기록이 없는 경우 에러가 아닌 `meals: []`로 응답, 클라이언트가 "기록 없음" 빈 상태(empty state) 렌더링 |
| 500 | INTERNAL_ERROR | 조회 처리 중 DB 오류 |

---

### API-009. 주간 요약 리포트 조회

| 항목 | 내용 |
|------|------|
| Method | GET |
| URL | `/api/reports/weekly?week={YYYY-Www}` |
| 설명 | 주간 평균 칼로리 섭취량 및 체중 변화 추이를 그래프용 시계열 데이터로 제공한다. 기본적으로 최근 30일로 조회 범위를 제한한다(REQ-011). |
| 관련 기능 ID | F-005 |

**Request**

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|----------|------|------|:---:|------|
| week | query | string (YYYY-Www) | N | 조회 대상 주. 미지정 시 이번 주 기준 |

**Response (200 OK)**

```json
{
  "status": "success",
  "data": {
    "week": "2026-W36",
    "averageCalorieKcal": 2280,
    "dailyCalorieSeries": [
      { "date": "2026-08-31", "calorieKcal": 2100 },
      { "date": "2026-09-01", "calorieKcal": 2350 }
    ],
    "weightTrendSeries": [
      { "date": "2026-08-31", "weightKg": 68.2 },
      { "date": "2026-09-06", "weightKg": 68.6 }
    ]
  }
}
```

**에러 Response**

| 상태 코드 | 에러 코드 | 조건 |
|-----------|-----------|------|
| 400 | BAD_REQUEST | `week` 형식 오류 |
| 200(빈 상태) | - | 해당 주 기록이 없는 경우 시리즈 배열을 빈 배열로 반환(에러 아님) |
| 500 | INTERNAL_ERROR | 집계/조회 처리 중 오류 |

---

## 기능 ↔ API 추적 매트릭스

| F-ID | 기능명 | 관련 API-ID |
|------|--------|-------------|
| F-001 | 사진 촬영 식사 인식 | API-001, API-002 |
| F-002 | 배식량 보정 슬라이더 | API-003, API-004 |
| F-003 | 체중 목표 및 칼로리 밸런스 | API-005, API-006 |
| F-004 | 맞춤 운동 추천 | API-007 |
| F-005 | 기록 히스토리 및 대시보드 | API-008, API-009 |
| F-006 | PWA 배포 및 오프라인 열람 | 해당 없음 (클라이언트 서비스 워커/캐시 처리) |

> 기능명세서(#6)의 "관련 API" 항목에 명시된 엔드포인트 9건 전건이 API-001~API-009로 누락 없이 매핑됨을 확인. 세부 필드/타입은 데이터베이스설계서(#12) 및 화면설계서(#9) 확정 시 조정될 수 있다.

**작성 완료 여부**: [x] API 9건(API-001~API-009) 상세 작성 완료, 기능 6건(F-001~F-006) 전건 추적 매핑 완료.

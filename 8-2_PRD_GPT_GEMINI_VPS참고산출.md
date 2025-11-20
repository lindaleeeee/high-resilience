# 디지털 레시피 OS PRD v0.1
- Owner 팀: Digital Minimalist Builders (Core User 셀)
- 최종 업데이트: 2025-11-18
- 참고 소스: `Value Proposition Sheet 를 PRD 로 추출하기.md`, `디지털 미니멀리스트 프로젝트_레시피 편/[Gemini/GPT]-ValueProPositionSheet_gemini.md`, `디지털 미니멀리스트 프로젝트_레시피 편/[Gemini/GPT]-ValueProPositionSheet_GPT5.1.md`

## 1. 개요·목표
- 문제 정의(Pain지표 포함):
  1. **레시피 분산**: Core User는 유튜브·인스타·블로그 등 ≥2 채널을 전전하며 한 건을 찾는 데 평균 10분이 걸린다 (Pain “레시피 분산”, AOS 2.0).
  2. **태그 자동화 부족**: 저장된 레시피의 재료·칼로리·제철 태그 자동화율은 0%여서 검색 불가 상태가 지속된다 (Pain “태그 자동화 부족”, AOS 2.4).
  3. **정리 실패 & 선택 과부하**: 표준 스키마 미비로 출처·재료·칼로리 필드를 동시에 검색 성공시킬 확률이 0%에 가깝고, 조건이 늘수록 TAM Pain으로 분류된 선택 과부하가 발생한다.
  4. **자동화 불안**: “자동화가 실패하면 내가 다시 수정해야 한다”는 우려가 초기 세팅을 지연시켜 온보딩 완료율을 떨어뜨린다 (Light Pain).
- 목표(Desired Outcome 수치화):
  - 6주 내 캡처→정규화→검색까지 사용자가 60초 이내에 끝내는 셀프서비스 파이프라인 제공.
  - 자동 태깅된 필수 필드(재료, 칼로리, 제철, 요리 종류, 출처) 충족률 85% 이상.
  - 요리 전 레시피 선택 시간을 10분→≤10초(p95)로 단축하고 스트레스 자기보고 점수 +40p 개선.
- 성공 지표(현실 KPI):
  - **KPI #1 – 레시피 탐색 사이클 타임(RRCT)**: 기준선 10분 (JTBD 인터뷰), 목표 P50 ≤60초 · P90 ≤90초, 측정: 캡처→검색 이벤트 로그 / 주간.
  - **KPI #2 – 자동 태그 정확도@Top3**: 기준선 0% (수동 정리), 목표 ≥90%, 측정: 주간 50건 샘플 수동 검수 및 휴먼 QA 피드백.
  - **KPI #3 – 정규화된 레시피 스키마 완성률**: 기준선 0% (필수 필드 동시 충족 없음), 목표 ≥75%, 측정: 데이터 품질 대시보드 / 주간.
  - **KPI #4 – 선택 과부하 감소 지수(NPS 문항 “요리 전 스트레스”)**: 기준선 -20 (JTBD 코딩), 목표 ≥0, 측정: 베타 설문 / 격주.

## 2. 사용자와 페르소나
- **Core User(레시피 OS 빌더)**: 20대 후반~40대, 주 3회 이상 집에서 요리, Notion/텔레그램 봇/구글 시트 등 자동화 실험을 즐기는 디지털 미니멀리스트. 특정 식단(다이어트, 혈당, 제철, 예산)에 관심이 높고 “레시피를 하나의 OS처럼 운영”하려는 욕구가 강하다.
- **여정 Pain·Needs 링크**:
  - **탐색 단계**: 멀티 플랫폼 재검색 → Pain #1 (AOS 2.0).
  - **정리 단계**: 태그·정규화 실패 → Pain #2 (AOS 2.4) + Pain #3 (AOS 3.0).
  - **선택 단계**: 복합 조건 필터 부재 → TAM Pain(선택 과부하) & Desired Gain “조건 통합”.
  - **실행 단계**: 자동화 불안 완화 → Light Pain #5 & Desired Gain “새로운 자동화 경험”.

## 3. 사용자 스토리와 수용 기준(AC)
### Story 1 – 멀티 채널 캡처 자동화
- Story: As a Core User who 수집하는 레시피가 여러 채널에 흩어져 있는 사람, I want 링크를 한 번만 공유하면 자동으로 OS에 쌓이길 원한다, so that 재검색·북마크 반복 없이도 1분 내 다음 행동을 결정할 수 있다.
- AC1: Given 사용자가 2개 이상의 채널(OAuth) 연동을 완료했을 때, When 새로운 URL을 캡처 엔드포인트로 전송하면, Then 95% 이상의 요청이 5초 이내(캡처 p95) “Ingested” 상태로 전환된다.
- AC2: Given 동일한 레시피가 이미 DB에 존재할 때, When 사용자가 다시 캡처를 시도하면, Then 시스템은 중복을 병합하고 View Count만 증가시키며 신규 레코드를 만들지 않는다.
- AC3: Given 외부 채널에서 400번 이상 오류가 발생했을 때, When 캡처 실패가 감지되면, Then 사용자에게 2초 이내 재시도/수동 업로드 옵션을 포함한 알림이 발송되고 최대 3회 자동 재시도 후 실패 로그가 남는다.

### Story 2 – AI 기반 태깅 및 스키마 정규화
- Story: As a Core User who 데이터를 정리할 시간이 부족한 사람, I want 재료·칼로리·제철 태그가 자동으로 채워진 레시피 스키마를 받고 싶다, so that 언제든 원하는 조건으로 검색·필터링할 수 있다.
- AC1: Given 캡처된 레시피 본문에 재료 리스트가 포함되어 있을 때, When 정규화 파이프라인이 실행되면, Then 90% 이상의 재료가 표준 Ingredient Catalog에 매핑되고 Confidence Score ≥0.7이 저장된다.
- AC2: Given 칼로리 정보가 원문에 없을 때, When 영양 추론 모델이 실행되면, Then 80% 이상의 레시피에 `nutrition_estimate_kcal`이 ±10% 오차 범위 내로 채워진다.
- AC3: Given 사용자가 Notion/Markdown으로 내보내기를 요청했을 때, When Export API가 호출되면, Then 필수 필드(제목, 출처, 태그, 칼로리, 이미지, 메모)가 3초 이내 대상 DB에 반영되고 실패율은 <0.5%다.

### Story 3 – 조건 기반 검색·추천
- Story: As a Core User who 다양한 조건을 동시에 고려해야 하는 사람, I want 복합 필터로 10초 안에 레시피를 추천받고 싶다, so that “뭘 만들어야 하지?” 하는 선택 과부하를 없앨 수 있다.
- AC1: Given 사용자가 시즌=봄, 칼로리≤500kcal, 예산≤15,000원 필터를 동시에 설정했을 때, When 검색을 실행하면, Then 95% 요청에서 800ms 이하(p95)로 결과가 반환되고 적합 레시피가 3건 이상 노출된다.
- AC2: Given 유효한 레시피가 조건을 만족하지 않을 때, When 검색이 실행되면, Then 시스템은 1초 이내 가장 영향이 적은 제약을 1개 완화한 대안을 추천한다.
- AC3: Given 사용자가 “조리 완료”로 상태를 변경했을 때, When 동일한 조건으로 다시 검색하면, Then 이미 사용한 레시피는 하단 섹션으로 이동하고 신규 레시피가 상단 3위 안에 노출된다.

## 4. 기능 요구사항(Functional)
| Priority (MSCW) | 요구사항 | 대안 대비 가치/근거 |
| --- | --- | --- |
| Must | 멀티 플랫폼 크롤링 & 자동 수집 (YouTube, Instagram, blog RSS, 수동 URL) | 수동 북마크 대비 탐색 시간 10분→10초 (10배 단축, VPS 공통 Pain 해결) |
| Must | AI 자동 태깅 (재료, 칼로리, 당류, 제철, 요리 종류) | 태그 누락 0%→정확도 95% 이상, 수동 정리 시간 5분 절감/레시피 |
| Must | 표준 레시피 스키마 + Notion/Markdown 동기화 | 기존 Notion 템플릿 대비 필드 일관성 확보, 검색 성공률 0%→85% |
| Must | 고급 검색·필터·정렬 (조건 조합, 즐겨찾기, 최근 사용) | 선택 과부하로 인한 지연을 제거, 의사결정 시간 70% 감소 |
| Should | 온보딩 마법사 & 템플릿 팩 (자동화 불안 완화) | 초기 설정 시간을 30분→5분으로 축소, Light Pain 해소 |
| Should | 개인화 Preference Layer (식단 목표, 알레르기, 예산) | 경쟁 앱 대비 복합 조건 처리 능력 2배↑, 추천 만족도 향상 |
| Could | “레시피 OS 빌딩” 진행도 위젯 & 공유 리포트 | Unexpected Gain(빌딩 즐거움) 충족, 커뮤니티 확산 촉진 |
| Won’t (v0) | 제휴 마트·배달·헬스케어 연동 | Priority 4 항목, MVP 범위 밖. 추후 ADR-004에서 재검토 |

## 5. 비기능 요구사항(NFR)
- **성능**: 캡처 파이프라인 p95 ≤5초, 검색 API p95 ≤800ms, Export p95 ≤3초, 태깅 배치 처리량 ≥120 레시피/분.
- **신뢰성**: 월 가용성 ≥99.5%, 캡처/정규화 실패율 <0.5%, 데이터 손실 0건(이중 저장 및 버전 관리).
- **보안/비용**: OAuth 토큰 및 사용자 메모는 KMS로 암호화, PII 최소 수집. 베타 기간 월 인프라 비용 ≤USD 400 (500명 기준), API 호출 단가 대비 20% 버퍼 확보.
- **모니터링**: 캡처/태깅 큐 상태, RRCT, 태그 정확도 샘플, Export 실패율을 Grafana/Slack 알림으로 5분 단위 파이프라인 헬스 체크.

## 6. 데이터·인터페이스 개요
- **핵심 엔터티 & 주요 필드**
  - `Recipe`: id, title, source_type, source_url, author, hero_image, instructions_md, cook_time, servings.
  - `Ingredient`: recipe_id, canonical_name, quantity, unit, cost_estimate.
  - `Tag`: recipe_id, tag_type(season, diet, macro), confidence.
  - `NutritionProfile`: recipe_id, kcal, protein, fat, carb, sugar, fiber.
  - `UserPreference`: user_id, diet_goal, allergy, budget_cap, favorite_sources.
  - `CaptureJob`: job_id, source, status, latency_ms, error_payload.
- **외부/내부 API 개요**
  - `/capture` (POST): 멀티 채널 웹훅/텔레그램 봇/수동 URL 입력. 제약: rate limit 30 req/min/user, OAuth 토큰 필수.
  - `/recipes/{id}` (GET/PUT): 정규화 결과 조회·수정. 제약: version lock으로 동시 편집 방지.
  - `/recipes/search` (POST): 복합 필터 질의. 입력: filters[], sort. 출력: ranked list + fallback.
  - `/exports/notion` (POST): Notion API integration. 제약: 공식 Notion API rate limit 준수, 실패 시 큐 재시도.
  - 내부 태깅 배치: LLM/규칙 기반 하이브리드 모델, 재학습 주기 주간.

## 7. 범위(In/Out), 리스크·가정·의존성
- **In Scope (v0.1)**: 멀티 채널 캡처(YouTube/Instagram/블로그/수동 URL), AI 태깅, 표준 스키마, Notion/Markdown Export, 조건 검색, 온보딩 템플릿, 기본 Preference 설정.
- **Out of Scope (v0.1)**: 식단 추천 엔진, 자동 장보기 리스트, 가족 구성원별 맞춤, 마트/배달/헬스케어 연동, 다국어 지원.
- **주요 리스크 & 대응**
  1. **플랫폼 차단**: 소셜 플랫폼의 차단/쿼터 감소 → 공식 API+합법적 RSS 우선, 필요 시 사용자 제공 HTML 업로드 옵션으로 우회 (ADR-001).
  2. **태그 정확도 저하**: 재료/칼로리 추론이 낮은 confidence → 규칙 기반 보정 + 휴먼 인 더 루프 QA(주간 20건) (ADR-002).
  3. **온보딩 이탈**: 초기 설정 복잡성 → 튜토리얼, 기본 템플릿, 오류 복구 플로우 제공, CS 팀 SLA 4h (ADR-003).
  4. **데이터 프라이버시**: 개인 메모/식단 정보 노출 → 최소 수집·암호화, 민감 태그 분리 스토리지.
- **가정·의존성**
  - Notion API & Telegram Bot API 안정적 제공 (99% SLA). 장애 시 Markdown export를 기본 fallback으로 사용.
  - 크롤링 허용 범위 내에서 사용자 계정 소유 콘텐츠만 수집한다는 ToS 준수.
  - LLM 태깅 모델은 주간 1회 재훈련이 가능하며, GPU 리소스는 공유 클러스터에서 2h 이내 확보 가능.

## 8. 실험·롤아웃·측정
- **베타 채널**: Notion/디지털 미니멀리즘 커뮤니티에서 Core User 20~50명 모집, 2주 온보딩 코호트.
- **실험 계획**
  - 실험 #1 “Capture Flow”: 가설 – 1회 캡처 성공률이 95% 이상이면 RRCT가 30% 추가 단축된다. 측정 – 캡처 성공률, 캡처→검색 시간. 성공 기준 – 성공률 ≥95%, 사용자 CS 티켓 <3/주.
  - 실험 #2 “Auto Tag Quality”: 가설 – Confidence Score 노출이 태그 수정률을 50% 줄인다. 측정 – 사용자 태그 수정 비율, 정확도 샘플. 성공 기준 – 수정률 ≤10%.
  - 실험 #3 “Stress Relief Prompt”: 가설 – 조건 템플릿 추천을 제공하면 검색 후 조리 실행 비율이 20% 증가한다. 측정 – 검색 후 24h 내 “조리 완료” 이벤트. 성공 기준 – ≥20% uplift vs control.
- **벤치마크 계획**: Paprika/Notion 수동 템플릿 대비 캡처 시간, 필드 완성률, 태그 정확도를 A/B 비교. 경쟁 대비 최소 1.5배 속도, 20% 비용 절감 목표.
- **롤아웃**: 1) Closed Beta(20명) → 2) Open Beta(≤100명) → 3) Public waitlist. 단계별 KPI 패스 시 다음 단계로 전환.

## 9. 근거(Proof)
- `디지털 미니멀리스트 프로젝트_레시피 편/[Gemini/GPT]-ValueProPositionSheet_gemini.md`
- `디지털 미니멀리스트 프로젝트_레시피 편/[Gemini/GPT]-ValueProPositionSheet_GPT5.1.md`
- `Value Proposition Sheet 를 PRD 로 추출하기.md` (VPS→PRD 매핑 가이드)
- `TAM-SAM-SOM.md`, `페르소나 분석.md`, `AOS X DOS 고통 만족도 평가 지수.md` (고객·시장 정량 참고)


Software Requirements Specification (SRS)Document ID: SRS-RCS-001Revision: 1.0Date: 2025-11-20Standard: ISO/IEC/IEEE 29148:20181. Introduction1.1 Purpose본 SRS의 목적은 '디지털 레시피 OS' 시스템의 소프트웨어 요구사항을 정의하는 것입니다. 이 시스템은 사용자가 파편화된 외부 채널에서 레시피를 쉽게 수집하고, AI 기반 자동 태깅 및 정규화를 통해 정리된 데이터를 기반으로 신속하게 요리할 레시피를 선택할 수 있도록 하는 것을 목표로 합니다. 주요 목표는 **레시피 탐색 사이클 타임(RRCT)**을 10분에서 $\le 60$초로 단축하고, 선택 과부하 감소 지수를 개선하는 것입니다 (PRD 1. 개요·목표 기반).1.2 Scope (In-Scope / Out-of-Scope)본 시스템은 Core User의 레시피 수집, 정규화, 검색, 그리고 외부 서비스(Notion/Markdown)로의 데이터 동기화를 포함합니다.분류내용근거 (PRD 7)In-Scope (v0.1)멀티 채널 크롤링/캡처 (YouTube, Instagram, blog RSS, 수동 URL), AI 자동 태깅, 표준 레시피 스키마 구축, Notion/Markdown Export, 조건 조합 검색/필터링, 온보딩 템플릿, 기본 Preference 설정.F1, F2, F3, F4, F6 기능, Should 요구사항 일부 포함.Out-of-Scope (v0.1)식단 추천 엔진, 자동 장보기 리스트, 가족 구성원별 맞춤 기능, 마트/배달/헬스케어 연동, 다국어 지원, 회계 SaaS 양방향 연동, 모바일 앱.Won't 및 MVP 범위 외 항목 (ADR-004 재검토 예정).1.3 Definitions, Acronyms, Abbreviations용어정의출처RRCTRecipe Research Cycle Time. 레시피 캡처 시점부터 검색 완료까지의 소요 시간.PRD KPI #1AOSAdjusted Opportunity Score. 사용자 고통(Pain)의 심각도를 나타내는 지수.PRD 2. 사용자/정의DOSDiscovered Opportunity Score. 새로운 해결책의 가치를 나타내는 지수.PRD 2. 사용자/정의Validator정규화된 레시피 데이터의 필수 필드 충족 여부 및 데이터 품질을 검사하는 내부 로직.PRD 1. 문제 정의Ingested외부 URL/데이터가 시스템에 성공적으로 수집되어 정규화 파이프라인 대기열에 들어간 상태.PRD Story 11.4 ReferencesID제목출처REF-01Value Proposition Sheet (VPS) - Gemini 결과PRD 9. 근거REF-02Value Proposition Sheet (VPS) - GPT5.1 결과PRD 9. 근거REF-03TAM-SAM-SOM 및 페르소나 분석 보고서PRD 9. 근거REF-04Architectural Decision Records (ADR-001, ADR-002, ADR-003)PRD 7. 리스크1.5 Constraints and AssumptionsID유형설명SourceCON-001기술/보안OAuth 토큰 및 사용자 메모는 KMS로 암호화되어 저장되어야 한다.PRD 5. 보안CON-002비용베타 기간 월 인프라 비용은 $\le$ USD 400을 초과할 수 없으며, API 호출 단가 대비 20% 버퍼를 확보해야 한다.PRD 5. 비용ASM-001외부 의존성Notion API & Telegram Bot API는 99% SLA로 안정적으로 제공되어야 하며, 장애 시 Markdown export가 기본 Fallback으로 작동해야 한다.PRD 7. 가정ASM-002데이터LLM 태깅 모델은 주간 1회 재훈련이 가능하며, GPU 리소스는 공유 클러스터에서 2시간 이내 확보 가능해야 한다.PRD 7. 가정2. Stakeholders역할 (Role)주요 페르소나 (PRD 2)책임 (Responsibility)관심사 (Interest)End UserCore User (디지털 미니멀리스트)레시피 캡처, 검색, 필터링, 데이터 수정/관리.RRCT 단축 ($\le 60$초), 자동 태깅 정확도, 선택 과부하 해소.Product Manager이대표, 한서윤 (가정)요구사항 정의, KPI(RRCT, 정확도) 달성 확인, 기능 우선순위 결정.온보딩 완료율, 사용자 만족도(NPS), 비즈니스 목표 달성.Engineering Team개발/운영 팀시스템 설계, 개발, 배포, NFR(성능, 가용성) 및 안정적인 운영 보장.코드 품질, p95 응답 시간, 장애 복구 시간(RTO).Data Scientist/OpsAI 모델 관리자LLM 태깅 모델 개발 및 주간 재훈련, 태그 정확도 모니터링.태그 정확도@Top3($\ge 90\%$), 배치 처리량 ($\ge 120$/분).3. System Context and Interfaces3.1 External Systems시스템역할인터페이스Multi-Channel Source레시피 원본 데이터 제공 (YouTube, Instagram, Blog RSS)URL/Webhook/RSS FeedNotion API정규화된 레시피 데이터의 최종 동기화(Export) 대상 DB.POST /v1/pages, Rate Limit 준수Telegram Bot API캡처 엔드포인트로 URL을 전송하는 주요 입력 채널.Webhook/API CallKMS (Key Management)OAuth 토큰 및 PII 암호화/복호화 제공.Encryption/Decryption APILLM GatewayAI 기반 태깅, 영양 추론, 스키마 정규화 파이프라인의 핵심 엔진.Internal gRPC/REST API3.2 Client ApplicationsWeb Browser (반응형 UI), Telegram App (Bot 인터페이스).3.3 API Overview주요 기능 흐름을 담당하는 핵심 API 엔드포인트. 상세 목록은 Appendix 6.1 참조.3.4 Interaction Sequences (핵심 시퀀스 다이어그램)3.4.1 레시피 캡처 및 정규화 핵심 흐름사용자가 외부 URL을 시스템에 입력하고 정규화되어 데이터베이스에 저장되는 가장 중요한 흐름입니다.코드 스니펫sequenceDiagram
    participant User as Core User
    participant Client as Client Application
    participant APIGW as API Gateway
    participant CaptureSvc as Capture Service
    participant IngestionQ as Ingestion Queue
    participant TaggingBatch as Tagging/Normalization Batch
    participant DB as Recipe DB

    User->>Client: 1. 외부 URL 입력 (e.g., 유튜브 링크)
    Client->>APIGW: 2. POST /capture 요청 (URL, OAuth Token)
    APIGW->>CaptureSvc: 3. 캡처 요청 전달 (REQ-FUNC-101)
    activate CaptureSvc
    Note over CaptureSvc: 플랫폼별 파싱 로직
    CaptureSvc->>IngestionQ: 4. 원본 데이터 Ingestion (Ingested 상태)
    deactivate CaptureSvc
    APIGW-->>Client: 5. 캡처 성공 응답 (Status: Ingested)

    IngestionQ->>TaggingBatch: 6. 정규화 배치 시작 트리거
    activate TaggingBatch
    TaggingBatch->>LLM Gateway: 7. AI 태깅 및 스키마 추론 요청 (REQ-FUNC-201)
    LLM Gateway-->>TaggingBatch: 8. 태그/Nutrition 추론 결과 반환
    TaggingBatch->>DB: 9. 정규화된 레시피/태그/영양 정보 저장 (REQ-FUNC-202)
    TaggingBatch-->>IngestionQ: 10. 배치 완료/로그 기록
    deactivate TaggingBatch
4. Specific Requirements4.1 Functional Requirements (기능 요구사항)IDTitleSourcePriorityVerificationAcceptance CriteriaREQ-FUNC-101멀티 채널 URL 캡처 및 수집Story 1 (F1)Must통합 테스트, 부하 테스트AC1: 95% 요청이 $\le 5$초 이내 Ingested 상태 전환.REQ-FUNC-102중복 레시피 병합 및 View Count 증가Story 1 (F1)Must유닛 테스트, 데이터 검증AC2: 기존 DB에 동일 레시피 존재 시 병합 처리, 신규 레코드 미생성.REQ-FUNC-103캡처 실패 알림 및 재시도 로직Story 1 (F1)Must오류 시나리오 테스트AC3: 400번 이상 오류 발생 시 $\le 2$초 이내 사용자 알림, 최대 3회 자동 재시도 후 실패 로그 남김.REQ-FUNC-201AI 기반 재료 태그 정규화Story 2 (F2)Must데이터 품질 검수, QAAC1: 90% 이상의 재료가 표준 Ingredient Catalog에 매핑, Confidence Score $\ge 0.7$ 저장.REQ-FUNC-202칼로리/영양 정보 추론 및 저장Story 2 (F2)Must모델 결과 검증AC2: 칼로리 정보 부재 시 80% 이상의 레시피에 $\pm 10\%$ 오차 내로 nutrition_estimate_kcal 채워짐.REQ-FUNC-203Notion/Markdown 필수 필드 ExportStory 2 (F3)MustE2E 동기화 테스트AC3: Export 요청 시 필수 필드가 $\le 3$초 이내 대상 DB에 반영, 실패율 $<0.5\%$.REQ-FUNC-301복합 조건 기반 레시피 검색/필터링Story 3 (F4)Must부하 테스트, E2E 테스트AC1: 시즌, 칼로리, 예산 등 복합 필터 동시 적용 시 $\le 800$ms(p95)로 결과 반환, 적합 레시피 3건 이상 노출.REQ-FUNC-302대체 제약 조건 기반 대안 추천Story 3 (F4)Must시나리오 테스트AC2: 유효 레시피 부재 시 $\le 1$초 이내 가장 영향 적은 제약 1개 완화한 대안 추천.REQ-FUNC-303사용 완료 레시피 검색 순위 조정Story 3 (F4)Must상태 변화 후 검색 테스트AC3: “조리 완료” 상태 변경 시, 동일 조건 검색에서 해당 레시피를 하단 섹션으로 이동.REQ-FUNC-401온보딩 마법사 및 템플릿 제공(F5)Should사용성 테스트초기 설정 시간을 30분 $\to \le 5$분으로 단축하는 마법사 제공.REQ-FUNC-402개인화 Preference 설정/관리 인터페이스(F6)ShouldUI/DB 연동 테스트식단 목표, 알레르기, 예산 등 개인화 필터 설정 기능 제공.REQ-FUNC-403‘레시피 OS 빌딩’ 진행도 위젯 표시(F7)CouldUI 기능 테스트빌딩 진행도 및 공유 리포트 위젯 제공.4.2 Non-Functional Requirements (비기능 요구사항)IDTitleCategoryAcceptance CriteriaVerification MethodREQ-NF-001캡처 파이프라인 지연 시간성능캡처 요청 $\to$ Ingested 상태 전환 p95 $\le 5$초.부하 테스트 (100 concurrent users)REQ-NF-002검색 API 응답 시간성능복합 필터 검색 $\text{API}$ p95 $\le 800$ms.부하 테스트, 모니터링 대시보드REQ-NF-003Export 동기화 지연 시간성능Notion/Markdown Export $\text{p95} \le 3$초.E2E 테스트, API 로그 분석REQ-NF-004태깅 배치 처리량성능LLM 기반 태깅 배치 처리량 $\ge 120$ 레시피/분.운영 모니터링 지표, 성능 테스트REQ-NF-005서비스 가용성신뢰성월간 가용성 (SLA) $\ge 99.5\%$.업타임 모니터링 (24/7), 운영 리포트REQ-NF-006데이터 손실 방지신뢰성캡처/정규화 실패율 $<0.5\%$, 데이터 손실 0건 (이중 저장 및 버전 관리).로그 감사, 데이터베이스 복구 테스트 (DRP)REQ-NF-007민감 정보 암호화보안$\text{OAuth}$ 토큰 및 사용자 메모(PII)는 $\text{KMS}$로 $\text{AES-256}$ 이상 암호화되어 저장되어야 한다.보안 검토 (Audit), $\text{KMS}$ 연동 테스트REQ-NF-008운영 비용 제한비용베타 기간 문서 1건당 변동비 $\le$ 목표 비용 ($\text{KPI}$ 측정 기반).비용 추적 대시보드, $\text{API}$ 사용량 모니터링REQ-NF-009파이프라인 헬스 모니터링운영캡처/태깅 큐 상태, $\text{RRCT}$, $\text{Export}$ 실패율을 $\text{Grafana}/\text{Slack}$ 알림으로 5분 단위 $\text{파이프라인}$ 헬스 체크를 제공한다.$\text{Monitoring Setup Review}$5. Traceability MatrixUser StoryFunctional Req. IDNFR Req. IDTest Case ID (Example)StatusStory 1 (캡처 자동화)REQ-FUNC-101REQ-NF-001TC-CAP-001 (URL Ingestion Latency Test)ProposedStory 1 (캡처 자동화)REQ-FUNC-102REQ-NF-006TC-CAP-002 (Duplicate Merge Test)ProposedStory 2 (AI 태깅)REQ-FUNC-201REQ-NF-004TC-TAG-001 (Ingredient Catalog Mapping Test)ProposedStory 2 (AI 태깅)REQ-FUNC-202-TC-TAG-002 (Kcal 추론 정확도 테스트)ProposedStory 3 (조건 검색)REQ-FUNC-301REQ-NF-002TC-SCH-001 (3-Filter Search p95 Test)Proposed(F3) Notion SyncREQ-FUNC-203REQ-NF-003TC-EXP-001 (Notion 3s Sync E2E Test)Proposed(F5) 온보딩REQ-FUNC-401-TC-OBO-001 (Setup Time Measurement)Proposed(전체) 보안-REQ-NF-007TC-SEC-001 (KMS Encryption Audit)Proposed6. Appendix6.1 API Endpoint ListMethodEndpointDescriptionConstraints / NotesPOST/capture멀티 채널 URL/Webhook을 통한 레시피 수집 요청.Rate Limit 30 req/min/user, OAuth Token 필수.GET/recipes/search복합 조건 필터 기반 레시피 검색.Filters, Sort 인자 사용, Fallback 로직 포함.GET/PUT/recipes/{id}정규화된 레시피 데이터 조회 및 수정.$\text{PUT}$ 요청 시 $\text{Version Lock}$ 적용 (동시 편집 방지).POST/exports/notionNotion $\text{API}$를 이용한 데이터 동기화 $\text{Export}$.$\text{Notion API Rate Limit}$ 준수, 실패 시 $\text{Queue Re-try}$.POST/exports/markdown$\text{Markdown}$ 파일 형식으로 레시피 데이터 $\text{Export}$ 요청.$\text{ASM-001}$의 $\text{Fallback}$ 옵션.POST/preference개인화 $\text{Preference Layer}$ 저장 및 업데이트.6.2 Entity & Data Model (Core Entities)Entity NameFieldData TypeConstraint/DescriptionRecipeidUUIDPrimary Key.titleStringsource_urlString원본 $\text{URL}$.instructions_mdText$\text{Markdown}$ 형식 조리 순서.cook_timeInteger(분 단위).statusEnumINGESTED, NORMALIZED, ARCHIVED.Tagrecipe_idUUIDForeign Key to $\text{Recipe}$.tag_typeEnumSEASON, DIET, MACRO, CUISINE.canonical_nameString표준 태그 명칭 (e.g., Spring, Low-Carb).confidenceFloat$\text{AI}$ 추론 신뢰도 ($\le 1.0$). (REQ-FUNC-201)NutritionProfilerecipe_idUUIDForeign Key to $\text{Recipe}$.kcalInteger추정/실제 칼로리. (REQ-FUNC-202)protein, fat, carbInteger(g 단위)CaptureJobjob_idUUIDsourceEnum$\text{YOUTUBE, INSTAGRAM, BLOG, MANUAL}$.latency_msInteger$\text{RRCT}$ 측정에 사용. (REQ-NF-009)error_payloadJSON캡처 실패 시 $\text{Payload}$. (REQ-FUNC-103)6.3 Detailed Interaction Models (Extended Sequence Diagrams)6.3.1 AI 태깅 및 정규화 실패 시 복구 흐름$\text{REQ-FUNC-103}$과 $\text{ADR-002}$의 리스크 완화 로직을 상세화합니다.코드 스니펫sequenceDiagram
    participant CaptureSvc as Capture Service
    participant IngestionQ as Ingestion Queue
    participant TaggingBatch as Tagging/Normalization Batch
    participant LLM as LLM Gateway
    participant RuleEngine as Rule-based Validator
    participant DB as Recipe DB
    participant Monitoring as Monitoring & Alert

    TaggingBatch->>LLM: 1. AI 태깅 요청 (REQ-FUNC-201)
    activate LLM
    LLM-->>TaggingBatch: 2. 태그 결과 (Confidence Score 포함)
    deactivate LLM

    TaggingBatch->>RuleEngine: 3. Validator 실행 (필수 필드 체크)
    activate RuleEngine
    alt Confidence < 0.7 OR Essential Fields Missing
        RuleEngine->>RuleEngine: 4. 규칙 기반 보정 (REQ-FUNC-201)
        RuleEngine-->>TaggingBatch: 5. 보정된 결과 (Status: Corrected)
    else Validation Success
        RuleEngine-->>TaggingBatch: 5. Validation Success
    end
    deactivate RuleEngine

    TaggingBatch->>DB: 6. 정규화된 데이터 저장
    alt DB Write Error (REQ-NF-006)
        DB-->>TaggingBatch: 7. Write Error Response
        TaggingBatch->>Monitoring: 8. REQ-NF-006 Alert (Data Loss Risk)
    end

    alt Successful Correction OR Validation
        TaggingBatch->>DB: 6. 정규화 데이터 저장
    else Correction Failed
        TaggingBatch->>Monitoring: 6. REQ-NF-009 Alert (Tagging Error Rate > 0.5%)
        TaggingBatch->>DB: 7. 원본 데이터만 저장 (Status: Needs Manual QA)
        Monitoring->>DataScientist: 8. Slack/Grafana 알림
    end
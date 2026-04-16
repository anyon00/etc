# Ai_miniOSS

**AI-powered mini Operation Support System**

프로젝트 태스크 정의서  |  v2.0  |  2026.04.16

---

## TASK 1

### 역량 및 기술 스택 확인

#### 1. 현재 보유 역량

| 기술/영역 | 수준 | 비고 |
|-----------|------|------|
| C | 고급 | 시스템 레벨 프로그래밍 경험 다수 |
| Java | 초급 | 기본 문법 및 OOP 이해 |
| Unix/Linux | 초~중급 | CLI, 쉘 스크립트, 프로세스 관리 |
| DB (Oracle, PostgreSQL 등) | 중급 이상 | 스키마 설계, 쿼리 최적화 경험 다수 |
| Python | 중급 | FastAPI 비동기 백엔드 구현 완료 |
| AI/ML | 초~중급 | LLM 기반 Multi-Agent 시스템 구현 완료 |
| LLM 활용 | 중급 이상 | GPT, Claude, Gemini, Grok 다수 활용 + 프롬프트 엔지니어링 |
| RAG/Agent 구현 | 중급 | pgvector RAG + 17개 에이전트 오케스트레이션 구현 완료 |

#### 2. 프로젝트 수행을 통해 습득한 기술

| 기술 영역 | 습득 내용 | 적용 위치 |
|-----------|-----------|-----------|
| FastAPI 비동기 백엔드 | async/await, Depends DI, 미들웨어, CORS | `main.py`, `api/` 전체 |
| SQLAlchemy 2.0 Async | 비동기 ORM, 관계 매핑, 마이그레이션 | `models/`, `core/state.py` |
| Multi-Agent 오케스트레이션 | LLM Planner → Agent Router → Synthesizer 패턴 | `core/orchestrator.py` |
| 유한 상태 머신 (FSM) | 13상태 × 19이벤트 = 42전이 규칙 설계 | `core/workflow.py` |
| RAG 파이프라인 | pgvector 코사인 유사도 검색 + 임베딩 | `utils/rag.py`, `utils/embedding.py` |
| 프롬프트 엔지니어링 | JSON 구조화 출력, 역할별 톤 분리, 컨텍스트 주입 | `prompt_templates/` |
| Redis 세션 관리 | 대화 이력 TTL 관리, 세션 컨텍스트 | `core/memory.py` |
| STT/TTS 통합 | Google Web Speech API, Edge-TTS, 발음 치환 | `core/stt_engine.py`, `core/tts_engine.py` |
| JWT 인증 | bcrypt 해싱, RBAC, 미들웨어 체인 | `core/auth.py`, `api/deps.py` |
| Docker Compose | 멀티 컨테이너 (app + PostgreSQL + Redis) | `Dockerfile`, `docker-compose.yml` |

#### 3. 실제 적용 기술 스택

| 구분 | 선정 기술 | 선정 사유 |
|------|-----------|-----------|
| Backend | Python 3.13 + FastAPI | 비동기 처리, 타입 안전성, 자동 API 문서화 |
| Orchestration | Custom Orchestrator + WorkflowFSM | Temporal 없이 경량 구현, LLM Planner로 동적 라우팅 |
| LLM | Gemini 2.0 Flash (OpenRouter) | 비용 효율, 빠른 응답, JSON 구조화 출력 우수 |
| Vector DB | pgvector (PostgreSQL 확장) | 별도 인프라 불필요, 코사인 유사도 검색 |
| Embedding | text-embedding-3-small (768차원) | OpenRouter 경유, 한국어 성능 양호 |
| Cache/Memory | Redis (aioredis) | 대화 이력 TTL 관리, 세션 컨텍스트 |
| STT | Google Web Speech API | 무료 티어, 한국어 인식 정확도 우수 |
| TTS | Edge-TTS (Microsoft) | 무료, 자연스러운 한국어 음성, LRU 캐싱 |
| Database | PostgreSQL + asyncpg | pgvector 확장, 비동기 드라이버 |
| Auth | JWT (PyJWT + bcrypt) | 무상태 인증, 역할 기반 접근 제어 |
| Frontend | HTML/CSS/JS (PWA) + Leaflet 지도 | 별도 빌드 없이 정적 서빙, 모바일 지원 |
| Mobile | Capacitor (Android) | WebView 기반 네이티브 앱 래핑 |
| Container | Docker Compose | app + db + redis 3-tier 로컬 실행 |

#### 4. 프로젝트 도메인

| 항목 | 내용 |
|------|------|
| 1순위 도메인 | 통신/ISP (Internet Service Provider) 현장 운영 지원 |
| 관심 사유 | 모뎀/라우터 장애, 기사 배정, 현장 사진 분석 등 실무 Pain Point가 명확하며 AI로 대체 가능한 반복 작업이 많음 |
| 2순위 도메인 | 제조/설비 유지보수 (Predictive Maintenance) |
| 관심 사유 | 센서 데이터 기반 고장 예측, 작업 지시서 자동 생성 등 Ai_miniOSS 구조와 유사한 도메인 |

---

## TASK 2

### 문제 정의 및 서비스 기획

#### 프로젝트명

**Ai_miniOSS** — AI-powered mini Operation Support System

#### 배경 및 문제 정의

**현재 상황 (As-Is)**

- 고객 장애 접수 후 기사 배정까지 수작업 처리로 평균 30~60분 소요
- 현장 기사가 장비 상태를 육안으로만 판단, 과거 유사 사례 참조 불가
- 완료 보고서 수작업 작성 → 지식 축적 없이 소멸
- SLA 위반 임박 상황에 대한 실시간 모니터링 부재

**Pain Point**

| 구분 | 문제 | 영향 |
|------|------|------|
| 시간 | 접수~배정 수작업 평균 45분 | 고객 불만, SLA 위반 위험 |
| 비용 | 반복 장애 동일 원인 재발 (지식 미축적) | 재방문율 25% 이상 |
| 품질 | 현장 판단 의존, 오진단 발생 | 불필요한 자재 소모 |
| 운영 | SLA 알림 없음 → 위반 사후 인지 | 패널티 비용 발생 |

#### 목표 사용자 (Target User)

| 사용자 유형 | 역할 | 주요 니즈 | 구현 상태 |
|-------------|------|-----------|-----------|
| 고객 (최종 사용자) | 음성/텍스트 장애 신고, 가입/변경/해지 | 빠른 접수, 자가진단, 실시간 상태 확인 | **구현 완료** (demo.html) |
| 현장 기사 (Field Engineer) | 일정 관리, 브리핑, 결과 등록, 안전 체크 | 최적 동선, 장비 확인, 간편한 보고 | **구현 완료** (demo.html 기사 모드) |
| 운영자 (관제) | 티켓 모니터링, SLA 관리, DLQ 처리 | 실시간 현황판, 알람 | **구현 완료** (ops.html) |
| 시스템 (AI Agent) | 자동 분류, 배정, 보고서 생성 | 정확한 추론, 낮은 오탐률 | **구현 완료** (17개 에이전트) |

#### 핵심 기능 정의 및 구현 현황

| 기능 | 설명 | 구현 상태 | 주요 파일 |
|------|------|-----------|-----------|
| 기능 1: 음성 자동 접수 및 의도 분류 | 고객 음성/텍스트 → STT → LLM 의도 분류 → 에이전트 라우팅 | **완료** | `stt_engine.py`, `planner.py`, `orchestrator.py` |
| 기능 2: 자가진단 + 에스컬레이션 | 서비스별 자가진단 가이드 → 해결/기사방문 판단 | **완료** | `counseling.py` |
| 기능 3: 기사 자동 배정 + 시간 제안 | 기술/지역/시간 역량 기반 매칭 → 3개 시간 제안 | **완료** | `assign.py` |
| 기능 4: 신규 가입 상담 | 5단계 대화형 가입 → 상품 추천 → 티켓 발행 | **완료** | `subscription.py` |
| 기능 5: 서비스 변경/해지 | 업그레이드/다운그레이드/해지 + 위약금 계산 | **완료** | `service_change.py`, `termination.py` |
| 기능 6: 요금/청구 조회 | 최근 6개월 청구서 + 미납/연체 안내 | **완료** | `billing_agent.py` |
| 기능 7: 기사 일정 + 최적 동선 | 당일 배정 → nearest-neighbor 동선 → 지도 표시 | **완료** | `schedule.py` |
| 기능 8: 작업 결과 등록 | 대화형 4단계 → 결과코드 → 보고서 저장 | **완료** | `report.py` |
| 기능 9: 장비 재고 관리 | 기사 보유 재고 vs 필요 장비 매칭 + 불출 신청 | **완료** | `inventory.py`, `schedule.py` |
| 기능 10: 현장 브리핑 | 고객정보 + 외부시스템(ADAMS/AQUA) + 현장 특이사항 | **완료** | `briefing.py`, `external.py` |
| 기능 11: 안전 관리 | 위험개소 알림 + 5개 안전 체크리스트 | **완료** | `safety.py` |
| 기능 12: 현장 사진 QA | 설치/수리별 필수 사진 검증 (BEFORE/AFTER/DEVICE) | **완료** | `vision.py` |
| 기능 13: 업셀 지원 | 업셀 대상 판별 → 브리핑 → 상담센터 이관 | **완료** | `upsell.py` |
| 기능 14: 예측 분석 | 장비 반복고장/고객빈도/지역집중/장비수명 분석 | **완료** | `predictive.py` |
| 기능 15: 이탈 예측 | 5요소 100점 만점 Churn 스코어 산출 | **완료** | `churn.py` |
| 기능 16: 부하 분산 + 장비 밸런싱 | 기사별 업무량 분석 + 지점 간 장비 이동 추천 | **완료** | `routing.py` |
| 기능 17: 관제 대시보드 | 티켓 현황, 워크플로우, DLQ, 감사 로그, 알림 | **완료** | `router_ops.py`, `ops.html` |
| 기능 18: TTS 음성 응답 | 응답 텍스트 → 한국어 음성 합성 + 캐싱 | **완료** | `tts_engine.py` |
| 기능 19: RAG 지식 검색 | pgvector 유사도 검색 → 상담 컨텍스트 주입 | **완료** | `rag.py`, `embedding.py` |
| 기능 20: JWT 인증 + RBAC | 로그인/역할별 접근 제어 (ADMIN/ENGINEER/CUSTOMER) | **완료** | `auth.py`, `router_auth.py` |

#### 정량적 목표 (KPI)

| 성과 지표 | 목표 수치 | 측정 방법 | 구현 현황 |
|-----------|-----------|-----------|-----------|
| 접수~배정 처리 시간 | 10분 이내 | 티켓 생성 → ASSIGNED 상태 전이 시간 | 자동 배정 흐름 구현 완료 |
| Intent 분류 정확도 | 90% 이상 | LLM Planner JSON 기반 라우팅 | 17개 에이전트 라우팅 규칙 구현 |
| 멀티턴 대화 지원 | 무제한 턴 | Redis 세션 (TTL 30분) | 세션 기반 상태 유지 구현 |
| 보고서 자동 생성률 | 95% 이상 | 완료 티켓 대비 WorkReport 생성 수 | 대화형 4단계 결과 등록 구현 |
| API 응답 캐싱 | 유사 요청 캐시 적중 | TTS LRU 캐시 (SHA256 키) | 64개 LRU 캐시 구현 |
| 에이전트 커버리지 | 17개 도메인 | 등록된 에이전트 수 | 17/17 구현 완료 |

---

## TASK 3

### 시나리오 수립 — 사용자 관점 시나리오 및 입출력 정의

#### 시나리오 1: 고객 장애 접수 → 자가진단 → 기사 자동 배정

| 항목 | 내용 |
|------|------|
| ID | SC-001 |
| 상황 | 고객이 인터넷 불통으로 음성/텍스트 장애 신고 |
| 목표 | 자가진단 안내 → 미해결 시 접수 확인 → 기사 배정 → 시간 3개 제안 → 고객 선택 → 확정 |
| 사전 조건 | 고객 DB 등록 + 가용 기사 존재 + Gemini API 정상 + Redis 정상 |

| 단계 | 사용자 행동 | 시스템 동작 | 관련 코드 |
|------|-----------|-----------|-----------|
| 1 | "인터넷이 안 돼요" 음성 입력 | STT 변환 → LLM Planner → `counseling` 라우팅 | `stt_engine.py`, `planner.py` |
| 2 | - | 고객 식별 (phone → customers → engineers → GUEST) | `router_customer.py:140-158` |
| 3 | - | CounselingAgent: 증상 파악 → 자가진단 안내 ("공유기 재부팅 해보세요") | `counseling.py:149` |
| 4 | "해봤는데 안돼요" | CounselingAgent: 다음 단계 안내 → 반복 | `counseling.py` (guiding) |
| 5 | "그래도 안돼요" | CounselingAgent: `needs_escalation: true` → AWAITING_CONFIRMATION | `orchestrator.py:527` |
| 6 | "네, 접수해주세요" | IssueAgent: AS-YYYYMMDD-NNNNNN 티켓 생성 | `issue.py:16` |
| 7 | - | AssignAgent(suggest): 방문 가능 시간 3개 제안 | `assign.py:66` |
| 8 | "두번째요" | AssignAgent(confirm): 배정 확정 + 고객 알림 발송 | `assign.py:310` |
| 9 | - | InventoryAgent: 기사 보유 장비 자동 확인 | `orchestrator.py:373` |

| 항목 | 내용 |
|------|------|
| 입력 예시 | "인터넷이 갑자기 안 되는데요. 모뎀 불빛이 빨간색이에요." (전화번호: 010-1234-5678) |
| 기대 출력 | session_id + 자가진단 가이드 → 접수 확인 → AS 티켓번호 + 방문 시간 3개 → 확정 응답 + TTS 음성 |
| 상태 전이 | NEW → INTENT_IDENTIFIED → DIAGNOSING → AWAITING_CONFIRMATION → TICKET_CREATED → AWAITING_SCHEDULE → ASSIGNED |

#### 시나리오 2: 기사 일과 — 브리핑 → 안전 → 작업 → 결과 등록

| 항목 | 내용 |
|------|------|
| ID | SC-002 |
| 상황 | 기사가 당일 일정 확인 후 현장 작업 수행 및 결과 등록 |
| 목표 | 일정 확인 → 브리핑 → 안전 체크 → 작업 완료 → 결과 등록 → 위험성 평가 |
| 사전 조건 | 기사 DB 등록 + 배정 존재 + 기사 전화번호로 식별 |

| 단계 | 사용자 행동 | 시스템 동작 | 관련 코드 |
|------|-----------|-----------|-----------|
| 1 | "오늘 일정 알려줘" | ScheduleAgent: 배정 목록 + 최적 동선 + 필요 장비 계산 | `schedule.py:111` |
| 2 | "다음 작업 브리핑" | BriefingAgent: 고객정보 + ADAMS/AQUA + 현장 특이사항 | `briefing.py:31` |
| 3 | "도착했어" | SafetyAgent: 위험개소 확인 + 5개 안전 체크리스트 | `safety.py:40` |
| 4 | "모두 체크했어" | SafetyAgent: SafetyChecklist 저장 → 작업 허용 | `safety.py:37` |
| 5 | "결과 등록할게" | ReportAgent: 4단계 대화 (티켓→코드→상세→확인) | `report.py:84` |
| 6 | (4단계 완료) | WorkReport 저장 + Ticket COMPLETED + DeviceSwap 기록 | `report.py:_save_report` |
| 7 | - | RiskAssessment: 위험성 평가 5개 질문 | `report.py:_risk_assessment` |

| 항목 | 내용 |
|------|------|
| 입력 예시 | 기사 전화번호로 식별 → "오늘 일정", "브리핑", "도착", "결과 등록" 순차 발화 |
| 기대 출력 | 일정 테이블 + 지도 + 브리핑 텍스트 + 안전 경고 + 결과 등록 확인 |
| 상태 전이 | INTENT_IDENTIFIED → (자유 분기) → REPORTING → COMPLETED → INTENT_IDENTIFIED (복귀) |

#### 시나리오 3: 신규 고객 가입 → 설치 기사 배정

| 항목 | 내용 |
|------|------|
| ID | SC-003 |
| 상황 | 신규 고객이 인터넷+TV 결합상품 가입을 원하는 상황 |
| 목표 | 5단계 가입 상담 → SB- 티켓 발행 → 기사 배정 → 시간 확정 |

| 단계 | 사용자 행동 | 시스템 동작 | 관련 코드 |
|------|-----------|-----------|-----------|
| 1 | "인터넷이랑 TV 가입하려고요" | LLM Planner → `subscription` 라우팅 | `planner.py` |
| 2 | "네, 맞아요" | Step 1: phone_confirm | `subscription.py:264` |
| 3 | "홍길동이요" | Step 2: name_confirm | `subscription.py` |
| 4 | "강남구 테헤란로 152" | Step 3: address_confirm → _save_customer_info() + _parse_region() | `subscription.py:355, 537` |
| 5 | "트리플 프리미엄으로" | Step 4: product_select (49,000원/월, 결합할인 8,000원) | `subscription.py:28` |
| 6 | "네, 가입할게요" | Step 5: confirmed → SB- 티켓 + CustomerService 생성 | `subscription.py:412` |
| 7 | - | 자동 assign(suggest): 설치 기사 시간 3개 제안 | `orchestrator.py:321` |
| 8 | "첫번째요" | assign(confirm): 배정 확정 | `assign.py:310` |

#### 시나리오 4: 기사 미할당 작업 자가 할당

| 항목 | 내용 |
|------|------|
| ID | SC-004 |
| 상황 | 기사가 여유 시간에 미할당 작업을 직접 수락 |

| 단계 | 사용자 행동 | 시스템 동작 | 관련 코드 |
|------|-----------|-----------|-----------|
| 1 | "남은 작업 있어?" | AssignAgent(unassigned): 미할당 티켓 + 장비 매칭 표시 | `assign.py:782` |
| 2 | "AS-20260416-000003 할당해줘" | AssignAgent(self_assign): 배정 생성 + 고객 SMS 발송 | `assign.py:890` |

#### 시나리오 우선순위 매트릭스

| 시나리오 | 내용 | 비즈니스 가치 | 구현 난이도 | 구현 상태 |
|----------|------|:----------:|:----------:|:---------:|
| SC-001 | 접수 → 자가진단 → 자동 배정 | 높음 | 높음 | **완료** |
| SC-002 | 기사 일과 (브리핑→안전→결과등록) | 높음 | 중간 | **완료** |
| SC-003 | 신규 가입 → 설치 배정 | 높음 | 중간 | **완료** |
| SC-004 | 기사 자가 할당 | 중간 | 낮음 | **완료** |
| SC-005 | 서비스 변경/해지 | 중간 | 중간 | **완료** |
| SC-006 | 요금/청구 조회 | 중간 | 낮음 | **완료** |
| SC-007 | 예측 분석 + 이탈 예측 | 중간 | 높음 | **완료** |

---

## TASK 4

### 상세 설계 및 개발 환경 구축

#### 1. Agent 페르소나 및 시스템 프롬프트

| 항목 | 정의 내용 |
|------|-----------|
| Agent 이름 | Ai_miniOSS Orchestrator |
| 주요 역할 | 통신사 현장 서비스 장애 접수~완료까지 전 과정을 자동화하는 멀티 에이전트 오케스트레이터 |
| 핵심 목표 | 고객 장애를 신속하게 분류·배정·완료하여 SLA를 준수하고 지식을 누적 |
| 톤앤매너 | 고객: 공감+존댓말 / 기사: 간결+실무적 / 운영: 수치 기반 |
| 제약 사항 | DB에 없는 정보 임의 생성 금지 / 고객 동의 없이 티켓 발행 금지 / JSON 구조화 출력 강제 |

#### 2. 워크플로우 및 오케스트레이션

**2.1 처리 로직 (한 턴의 10단계)**

| Step | 처리 내용 | 담당 | 파일 |
|:----:|-----------|------|------|
| 1 | 워크플로우 상태 로드/생성 | StateManager | `core/state.py` |
| 2 | 고객 서비스 정보 로드 (DB) | Orchestrator | `core/orchestrator.py:99` |
| 3 | 대화 이력 저장 (Redis) | ConversationMemory | `core/memory.py` |
| 4 | LLM Planner 호출 → 실행 계획 JSON | OpenRouterClient | `prompt_templates/planner.py` |
| 5 | 상태 전이 (WorkflowFSM) | StateManager | `core/workflow.py` |
| 6 | RAG 컨텍스트 검색 (pgvector) | EmbeddingService | `utils/rag.py` |
| 7 | 에이전트 순차 실행 | Agent.run() | `agents/*.py` |
| 8 | LLM Synthesizer 호출 → 자연어 응답 | OpenRouterClient | `prompt_templates/synthesizer.py` |
| 9 | 상태 영속화 (DB) | StateManager | `core/state.py:87` |
| 10 | TTS 변환 + 응답 반환 | TTSEngine | `core/tts_engine.py` |

**2.2 상태 머신 (WorkflowFSM)**

```
NEW → INTENT_IDENTIFIED → DIAGNOSING → DIAGNOSIS_RESOLVED → COMPLETED
                        → AWAITING_CONFIRMATION → (동의) → TICKET_CREATED
                                                → (거절) → DIAGNOSING
                        → TICKET_CREATED → AWAITING_SCHEDULE → ASSIGNED
                        → IN_PROGRESS → REPORTING → COMPLETED
                        → PARTIAL / FAILED
```

- 13개 상태, 19개 이벤트, 42개 전이 규칙
- 고객: 상태 머신 엄격 적용
- 기사: 자유 분기 모드 (COMPLETED 후 자동 복귀)

**2.3 LLM 이중 호출 패턴**

| 호출 | 담당 | 입력 | 출력 | 용도 |
|------|------|------|------|------|
| Planner | `orchestrator._get_plan()` | 사용자 입력 + 맥락 + 역할 + 상태 | JSON `{intent, plan, parameters}` | 에이전트 라우팅 |
| Agent | 에이전트별 `chat_json()` | 도메인 프롬프트 + 컨텍스트 | 도메인 JSON 결과 | 실제 업무 처리 |
| Synthesizer | `orchestrator._synthesize()` | 에이전트 결과 + 역할 + 호칭 | 자연어 텍스트 | 사용자 응답 |

#### 3. 에이전트 명세 (17개)

| 에이전트 | 파일 | 대상 | 핵심 기능 | DB 접근 |
|----------|------|------|-----------|---------|
| CounselingAgent | `counseling.py` | 고객 | 자가진단 가이드, 상담코드 분류, 이력 조회 | tickets, assignments (R) |
| IssueAgent | `issue.py` | 고객 | AS- 티켓 생성, 멱등성+중복 체크 | tickets (R/W) |
| AssignAgent | `assign.py` | 공통 | 5가지 배정 모드, 기사 매칭 알고리즘 | engineers, capabilities, assignments (R/W) |
| SubscriptionAgent | `subscription.py` | 고객 | 5단계 가입, 상품 카탈로그, SB- 티켓 | customers, customer_services, tickets (W) |
| ServiceChangeAgent | `service_change.py` | 고객 | 변경안 제안, SC- 티켓, 위약금 안내 | contracts, tickets (R/W) |
| TerminationAgent | `termination.py` | 고객 | 해지 조회/확정/취소, 위약금 계산 | customer_services, contracts, termination_requests (R/W) |
| BillingQueryAgent | `billing_agent.py` | 고객 | 청구서 조회, 미납 안내 | invoices, customer_services (R) |
| RequestAgent | `request.py` | 고객 | AS직접/이사/배선/택배 요청, MV-/WR-/DL- 티켓 | tickets (W) |
| ReportAgent | `report.py` | 기사 | 4단계 결과 등록, 위험성 평가 | work_reports, risk_assessments, device_swaps (W) |
| ScheduleAgent | `schedule.py` | 기사 | 당일 일정, nearest-neighbor 동선, 불출 신청 | assignments, engineer_inventory, office_inventory (R/W) |
| InventoryAgent | `inventory.py` | 기사 | 장비 매칭 (check/list/customer_device) | engineer_inventory, customer_services (R) |
| BriefingAgent | `briefing.py` | 기사 | 외부 시스템(ADAMS/AQUA) + 현장 특이사항 | assignments, site_info, customer_services (R) |
| SafetyAgent | `safety.py` | 기사 | 위험개소 알림, 5개 안전 체크리스트 | hazard_zones, safety_checklists (R/W) |
| UpsellAgent | `upsell.py` | 기사 | 업셀 대상 판별, 상담센터 이관 | upsell_targets, consultation_transfers (R/W) |
| VisionAgent | `vision.py` | 기사 | 사진 저장/QA검증 (설치3장, 수리2장) | field_photos (R/W) |
| PredictiveAgent | `predictive.py` | 운영 | 4가지 예측 분석 (고장/빈도/지역/수명) | work_reports, tickets, customer_services (R) |
| RoutingAgent | `routing.py` | 운영 | 부하 분산, 지점 간 장비 밸런싱 | engineers, assignments, engineer_inventory (R) |

#### 4. 지식 베이스 및 메모리 전략

**4.1 RAG 전략**

| 항목 | 구현 |
|------|------|
| 참조 데이터 | `knowledge_documents` 테이블 (지식 문서 + 장비 매뉴얼 + FAQ) |
| 임베딩 모델 | text-embedding-3-small (768차원, OpenRouter 경유) |
| Vector DB | pgvector (PostgreSQL 확장) |
| 검색 방식 | 코사인 유사도 `1 - (embedding <=> query_vector)` |
| 폴백 | 임베딩 실패 시 텍스트 기반 LIKE 검색 |
| 적용 위치 | Orchestrator에서 counseling 실행 전 자동 주입 |

**4.2 대화 메모리 (Redis)**

| 항목 | 구현 |
|------|------|
| 저장소 | Redis (aioredis) |
| 키 구조 | `conv:{session_id}:history` (List), `conv:{session_id}:ctx` (Hash) |
| TTL | 1800초 (30분) |
| 최대 조회 | 최근 10턴 (LRANGE -10 -1) |
| 초기화 | COMPLETED/FAILED 시 새 세션 자동 생성 |

#### 5. 데이터베이스 설계

| 분류 | 테이블 수 | 주요 테이블 |
|------|:---------:|------------|
| 마스터 데이터 | 8 | customers, engineers, products, regional_offices, office_coverage_areas, service_codes, sla_policies, knowledge_documents |
| 서비스/계약 | 5 | customer_services, contracts, invoices, payments, termination_requests |
| 티켓/배정 | 4 | tickets, assignments, work_reports, device_swaps |
| 기사 역량/재고 | 6 | engineer_time_slots, engineer_region_capabilities, engineer_skill_capabilities, engineer_inventory, office_inventory, inventory_requests |
| 현장 업무 | 6 | field_photos, hazard_zones, safety_checklists, risk_assessments, upsell_targets, consultation_transfers |
| 워크플로우/운영 | 7 | workflow_runs, workflow_tasks, audit_log, dead_letter_entries, alerts, customer_notifications, customer_satisfactions |
| 분석 | 3 | churn_scores, engineer_notes, engineer_locations |
| 인증 | 2 | user_accounts, site_info |
| **합계** | **41** | |

---

## TASK 5

### 구현 완료 모듈

#### 5.1 에이전트 오케스트레이션

| 항목 | 내용 |
|------|------|
| 구현 기능 | LLM Planner 기반 동적 에이전트 라우팅 + 순차 실행 + 자동 체이닝 |
| 동작 원리 | Planner가 사용자 입력/역할/상태를 분석 → plan 배열 반환 → Orchestrator가 순차 실행 |
| 자동 체이닝 | subscription → assign(suggest), request → assign(suggest), assign(confirm) → inventory(check) |
| 주요 기술 | FastAPI async, OpenRouter LLM, WorkflowFSM, Pydantic, SQLAlchemy 2.0 Async |

#### 5.2 도구(Tool) 및 함수 연동

| 도구명 | 기능 설명 | 입력 | 출력 |
|--------|-----------|------|------|
| `STTEngine.transcribe_base64` | WAV 음성 → 텍스트 변환 | audio_base64: str | text: str |
| `TTSEngine.synthesize_base64` | 텍스트 → MP3 음성 합성 | text: str | audio_base64: str |
| `OpenRouterClient.chat_json` | LLM JSON 구조화 출력 | messages: list | dict (파싱된 JSON) |
| `search_similar` | pgvector RAG 유사도 검색 | query: str, top_k: int | list[{title, content, score}] |
| `WorkflowStateManager.persist` | 워크플로우 상태 DB 저장 | state: WorkflowState | None |
| `NotificationService.dispatch` | 고객 알림 (SMS/Push) 발송 | customer_id, type, message | None |
| `ExternalSystemService.check_line` | ADAMS 회선 상태 조회 | customer_id, service_type | dict (line_status, port_status) |
| `ExternalSystemService.check_devices` | AQUA 장비 상태 조회 | customer_id, device_model | dict (device_status, firmware) |

#### 5.3 안전장치 및 멱등성

| 이슈 구분 | 문제 상황 | 해결 방법 |
|-----------|-----------|-----------|
| 티켓 중복 | 네트워크 재시도로 동일 티켓 2회 생성 | `idempotency_key` + DB 조회 체크 (SHA256 해시) |
| 예약 중복 | 같은 서비스 카테고리의 활성 티켓 존재 | `Ticket.status IN (활성)` + `category` 중복 체크 |
| 세션 재시작 | COMPLETED/FAILED 세션에 새 요청 | 자동 새 세션 생성 (session_id = None) |
| STT 오류 | 오디오 크기 초과, 포맷 불량 | WAV 헤더 검증 + 10MB 크기 제한 + 30초 타임아웃 |
| LLM 장애 | API 타임아웃, 네트워크 오류 | 3회 재시도 + exponential backoff (2^n초, 최대 10초) |
| TTS 장애 | Edge-TTS 서비스 불가 | 15초 타임아웃 + 오류 시 텍스트만 반환 (graceful degradation) |
| DB 정합성 | 에이전트 실행 중 오류 | 단일 트랜잭션 (commit/rollback 자동) |

---

## TASK 6

### 테스트 및 품질

#### 테스트 현황

| 테스트 모듈 | 테스트 수 | 커버리지 영역 |
|-------------|:---------:|--------------|
| `test_workflow.py` | 13개 | FSM 상태 전이 규칙 검증 |
| `test_state.py` | 13개 | StateManager create/transition/persist/load |
| `test_api_ops.py` | 13개 | 운영 API 엔드포인트 |
| `test_api_customer.py` | 9개 | 고객 API (chat, intake, ticket) |
| `test_agents.py` | 5개 | 에이전트 실행 기본 흐름 |
| `test_api_engineer.py` | 5개 | 기사 API (assignments, schedule) |
| **합계** | **58개** | |

#### 주요 문제 해결 및 기술 리서치

| 이슈 구분 | 문제 상황 | 해결 방법 |
|-----------|-----------|-----------|
| LLM 응답 파싱 | JSON 형식 불일치로 파싱 실패 | `response_format: {"type": "json_object"}` 강제 + 재시도 |
| Agent 통신 | Agent 간 직접 호출로 순환 의존 | 모든 Agent 통신을 Orchestrator 경유 원칙 적용 |
| 비동기 STT | Google Speech API가 blocking I/O | `asyncio.to_thread()`로 별도 스레드 실행 |
| TTS 발음 | 영어 약어를 영어식 발음 (AS→"에즈") | 정규식 기반 한국어 발음 치환 17개 규칙 적용 |
| 시간대 | KST/UTC 혼재로 배정 시간 오류 | `utils/timezone.py`로 KST 통일 + UTC 저장 원칙 |
| 기사 모드 | 기사가 REPORTING에서 다른 요청 불가 | 기사 자유 분기 모드 도입 (상태 머신 우회) |
| 장비 누적 | 일정에서 장비 부족 표시 부정확 | 방문 순서대로 누적 차감 방식으로 재구현 |

---

## TASK 7

### 최종 아키텍처 및 산출물

#### 1. 최종 아키텍처 요약

| 항목 | 내용 |
|------|------|
| 아키텍처 | Custom Orchestrator + WorkflowFSM 기반 Multi-Agent 시스템 |
| 산출물 | Docker Compose 로컬 실행 가능한 PoC / PWA 데모 / 관제 대시보드 / FastAPI 백엔드 |
| Agent 구조 | 17개 에이전트, LLM Planner 동적 라우팅, 모든 통신 Orchestrator 경유 |
| 프론트엔드 | PWA (demo.html + ops.html + scenario.html) + Capacitor Android 앱 |
| 코드 규모 | Python 69개 파일, 18,090줄 / DB 41개 테이블 / 테스트 58개 |

#### 2. 정량적 구현 현황

| 구분 | 수치 |
|------|------|
| 에이전트 | 17개 (고객 8, 기사 7, 운영 2) |
| API 엔드포인트 | 127개 (customer 11, engineer 20, billing 14, ops 69, auth 2, advanced 11) |
| DB 테이블 | 41개 (SQLAlchemy ORM) |
| FSM 전이 규칙 | 42개 (13상태 × 19이벤트) |
| 프롬프트 템플릿 | 2개 (Planner + Synthesizer) + 에이전트별 도메인 프롬프트 |
| 상품 카탈로그 | 11개 요금제 (인터넷 4, TV 2, 전화 1, 결합 4) |
| 지역 매핑 | 40+ 지역 (서울/경기) |
| 자가진단 가이드 | 3개 서비스 × 3~4단계 |
| TTS 발음 규칙 | 17개 |
| 테스트 | 58개 |

#### 3. 창출된 핵심 가치

**3-1. 비즈니스 가치**

- 음성 입력부터 기사 배정까지 완전 자동화 (수작업 45분 → 자동 처리)
- 자가진단 가이드로 불필요한 기사 방문 감소 (인터넷/TV/전화 3종)
- 대화형 결과 등록으로 보고서 자동 생성 + 지식 누적 (RAG KB)
- 실시간 운영 대시보드로 SLA 모니터링 + DLQ 관리

**3-2. 기술적 가치**

- LLM Planner → Agent → Synthesizer 이중 호출 패턴 확립
- WorkflowFSM 기반 결정적 상태 관리 (13상태, 42전이)
- 고객/기사 역할 분리 (상태 머신 vs 자유 분기)
- 멱등키 + 단일 트랜잭션 기반 데이터 정합성 보장
- pgvector RAG + 텍스트 폴백 이중 검색 전략
- Edge-TTS 한국어 발음 치환 + LRU 캐싱

#### 4. 운영 및 보안 고려사항

| 항목 | 구현 |
|------|------|
| 인증 방식 | JWT (PyJWT, TTL 24시간) + bcrypt 패스워드 해싱 |
| 권한 통제 | RBAC — ADMIN (전체), ENGINEER (기사 API), CUSTOMER (고객 API) |
| DB 보안 | SQLAlchemy ORM으로 SQL Injection 방지 + Pydantic 입력 검증 |
| 장애 대응 | LLM 3회 재시도 + STT/TTS 타임아웃 + DB 트랜잭션 롤백 |
| 감사 추적 | AuditLog 테이블에 모든 주요 작업 기록 |
| DLQ | DeadLetterEntry 테이블로 실패 작업 추적 + 재처리 API |

#### 5. 기술적 한계 및 향후 확장

**현재 한계**

| 항목 | 내용 |
|------|------|
| 외부 시스템 | ADAMS/AQUA는 Mock 어댑터 (실 연동 미완) |
| 경로 최적화 | Haversine 직선 거리 (실제 도로 기반 아님) |
| Vision AI | 사진 저장/QA만 구현, 이미지 분석 AI 미적용 |
| 부하 테스트 | 동시 사용자 대규모 테스트 미수행 |
| Predictive | DB 기반 통계 분석만 구현, ML 모델 미적용 |
| 배포 | Docker Compose 로컬 환경 (K8s 미적용) |

**Next Step**

| 우선순위 | 확장 내용 |
|:--------:|-----------|
| 1 | 실 T-map/Naver Map API 연동 (도로 기반 경로 최적화) |
| 2 | Gemini Vision API 연동 (현장 사진 AI 분석) |
| 3 | ADAMS/AQUA 실 API 연동 (Mock → Real 전환) |
| 4 | Kafka 이벤트 버스 도입 (비동기 처리 강화) |
| 5 | Kubernetes 전환 + 부하 테스트 |
| 6 | Hybrid Search (키워드 + Vector 결합) RAG 고도화 |
| 7 | Semantic Cache (Redis + 임베딩 유사도) 도입 |
| 8 | 도메인 특화 Fine-tuning (통신 장비 용어) |

---

## 최종 산출물 체크리스트

| 구분 | 산출물 | 상태 |
|------|--------|:----:|
| 설계 문서 | 기술 아키텍처 상세서 (`docs/00_technical_architecture.md`) | **완료** |
| 설계 문서 | 프로젝트 태스크 정의서 (본 문서) | **완료** |
| 설계 문서 | 기능별 프로세스 문서 6건 (`docs/01-06`) | **완료** |
| 설계 문서 | 문서 인덱스 (`docs/README.md`) | **완료** |
| 코드 | FastAPI 백엔드 서버 (`main.py`, `api/`, `core/`) | **완료** |
| 코드 | 멀티 에이전트 17개 (`agents/`) | **완료** |
| 코드 | 비즈니스 서비스 10개 (`services/`) | **완료** |
| 코드 | 유틸리티 (LLM, RAG, 임베딩, 타임존) | **완료** |
| 코드 | 시드 데이터 (`seed_data.py`) | **완료** |
| 인프라 | Docker Compose (app + PostgreSQL + Redis) | **완료** |
| DB | PostgreSQL 41개 테이블 + pgvector + 마이그레이션 | **완료** |
| 프론트 | PWA 데모 (`demo.html`) + 관제 (`ops.html`) + 시나리오 (`scenario.html`) | **완료** |
| 프론트 | Android 앱 (Capacitor) | **완료** |
| 테스트 | 유닛/통합 테스트 58건 | **완료** |
| 설정 | 환경 변수 (`.env.example`) | **완료** |

---

**Ai_miniOSS** — AI-powered mini Operation Support System

v2.0  |  2026.04.16

# ZIGZAC — 아키텍처 상세 문서

> 역분석 자산 기반 7레이어 AI 에이전트 통합 아키텍처

---

## 목차

1. [설계 철학](#1-설계-철학)
2. [전체 레이어 다이어그램](#2-전체-레이어-다이어그램)
3. [역분석 자산 목록](#3-역분석-자산-목록)
4. [통합 소스별 역할](#4-통합-소스별-역할)
5. [레이어별 상세 설계](#5-레이어별-상세-설계)
   - [L1 — 모델 라우터](#l1--모델-라우터)
   - [L2 — Function-Calling 프로토콜](#l2--function-calling-프로토콜)
   - [L3 — Tool Hub](#l3--tool-hub)
   - [L4 — Skills 레지스트리](#l4--skills-레지스트리)
   - [L5 — Self-Evolution 루프](#l5--self-evolution-루프)
   - [L6 — Handover 메모리 지속성](#l6--handover-메모리-지속성)
   - [L7 — 자율 오케스트레이터](#l7--자율-오케스트레이터)
6. [데이터 흐름](#6-데이터-흐름)
7. [모델 선택 전략](#7-모델-선택-전략)
8. [베타 헤더 주입 전략](#8-베타-헤더-주입-전략)
9. [기술 스택](#9-기술-스택)
10. [보안 고려사항](#10-보안-고려사항)
11. [구현 로드맵](#11-구현-로드맵)

---

## 1. 설계 철학

ZIGZAC의 핵심 원칙은 세 가지입니다.

1. **역분석 우위(Reverse-Engineering Advantage)** — 공개 API 문서가 아닌 실제 프로덕션 번들에서 추출한 비공개 모델 ID, 베타 헤더, 피처 플래그를 활용하여 일반 개발자보다 깊은 API 수준에서 동작합니다.

2. **조합 가능성(Composability)** — 각 레이어는 독립적으로 교체하거나 확장할 수 있습니다. openclaw의 플러그인 패턴, pi-skills의 스키마, hermes의 프로토콜은 모두 인터페이스로 추상화되어 특정 구현에 종속되지 않습니다.

3. **자기 확장(Self-Extension)** — hermes self-evolution 루프를 통해 에이전트는 능력이 부족한 상황을 스스로 감지하고 새 도구를 생성하여 Skills 레지스트리에 등록합니다. 운영자의 개입 없이 성장합니다.

---

## 2. 전체 레이어 다이어그램

```
사용자 / 외부 트리거
        │
        ▼
┌───────────────────────────────────────────────────────────────┐
│  [L7] 자율 오케스트레이터 (KAIROS · UltraPlan · AFK)          │
│  - 크론 스케줄 자동 실행                                       │
│  - 복잡 계획 사용자 승인 후 자율 수행                           │
│  - 장시간 무감독(AFK) 세션                                     │
│  - BYOC: 사용자 자체 클라우드에서 에이전트 실행                 │
└───────────────────────────┬───────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  [L6] Handover 메모리 지속성 시스템                            │
│  - 컨텍스트 창 75% → 자동 compact 트리거                       │
│  - continuation summary 생성 → 다음 세션 시스템 프롬프트 주입  │
│  - context-management API로 서버 측 자동 압축                  │
│  - 1M 토큰 컨텍스트 창 활성화                                  │
└───────────────────────────┬───────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  [L5] Self-Evolution 루프 (hermes-agent-self-evolution)        │
│  - 반복 실패 패턴 감지                                          │
│  - 새 도구/스킬 코드 자동 생성                                  │
│  - Tool Hub 핫 로드 (재시작 없이 등록)                          │
│  - advisor-tool로 사용자에게 새 능력 제안                       │
└───────────────────────────┬───────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  [L4] Skills 레지스트리 (pi-skills 스키마)                     │
│  - skill.json: 이름 · 버전 · 의존 도구 · 입출력 타입           │
│  - 의존성 그래프 기반 스킬 조합                                  │
│  - skills-2025-10-02 베타 헤더 → Anthropic Skills API 연동     │
│  - 내장 스킬: web-research · code-review · computer-automation │
└───────────────────────────┬───────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  [L3] Tool Hub (openclaw 플러그인 아키텍처)                    │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │
│  │ SDK Tools    │  │ Browser Tools│  │ OS Tools             │ │
│  │ (21개)       │  │ (18개)       │  │ (24개)               │ │
│  │ Agent        │  │ javascript   │  │ screenshot / zoom    │ │
│  │ Bash         │  │ read_page    │  │ left_click           │ │
│  │ FileRead/Edit│  │ navigate     │  │ type / key           │ │
│  │ WebFetch     │  │ computer     │  │ open_application     │ │
│  │ TodoWrite    │  │ gif_creator  │  │ computer_batch       │ │
│  │ …            │  │ …            │  │ …                    │ │
│  └──────────────┘  └──────────────┘  └──────────────────────┘ │
│                                                               │
│  + tool-search-tool 베타 → 에이전트가 도구를 직접 검색         │
└───────────────────────────┬───────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  [L2] Function-Calling 프로토콜 (hermes-agent)                │
│  - <tool_call> / <tool_response> XML 파싱                      │
│  - advanced-tool-use 베타 → 병렬 도구 호출                     │
│  - structured-outputs 베타 → 도구 응답 JSON 스키마 강제        │
│  - 권한 분류기(classifier): 허용/거부 자동 판단                 │
└───────────────────────────┬───────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  [L1] 모델 라우터 (Brain.md 역분석 자산)                       │
│                                                               │
│  모델 레지스트리 (n66)        베타 헤더 주입 (27개)             │
│  ┌──────────────────────┐    ┌─────────────────────────────┐  │
│  │ haiku-4-5            │    │ interleaved-thinking        │  │
│  │ sonnet-4-5 / 4-6     │    │ context-1m                  │  │
│  │ opus-4 / 4-1         │    │ fast-mode / afk-mode        │  │
│  │ × firstParty         │    │ advanced-tool-use           │  │
│  │ × Bedrock            │    │ structured-outputs          │  │
│  │ × Vertex             │    │ prompt-caching-scope        │  │
│  │ × Foundry            │    │ …                           │  │
│  └──────────────────────┘    └─────────────────────────────┘  │
│                                                               │
│  에이전트 상태 기계 (agent_dna_schema, 89개 필드)               │
└───────────────────────────────────────────────────────────────┘
```

---

## 3. 역분석 자산 목록

| 파일 | 원본 위치 | 핵심 내용 | 적용 레이어 |
|------|----------|----------|------------|
| `Brain.md` | `cli.js v2.1.88 b_5/n66/x38` | 11개 모델 ID × 4 프로바이더, 27개 베타 헤더, 토큰 제한 테이블 | L1 |
| `control.md` | `cli.js pc 배열 + getComputerTools()` | 브라우저 18종 + macOS 24종 = 42개 제어 도구 | L3 |
| `GEMINI_TOOLS_MANIFEST.json` | `sdk-tools.d.ts` | 21개 SDK 도구 OpenAPI 스키마 | L3 |
| `agent_dna_schema.json` | `cli.js Vj7() G8 singleton` | 전역 상태 89개 필드 (비용·시간·권한·훅) | L1, L2 |
| `Handover.md` | `cli.js vH7/aWK/gWK/P18` | continuation summary 프롬프트 전문, 압축 아키텍처 | L6 |
| `other.json` | `cli.js 전체 스캔` | 420개 피처 플래그 (tengu_*/KAIROS/UltraPlan) | L7 |

---

## 4. 통합 소스별 역할

### ZIGZAC (역분석 자산) — 핵심 엔진
모든 레이어의 **실제 구현 데이터**를 제공합니다. 공개 문서에 없는 모델 ID, 베타 헤더, 내부 프롬프트, 피처 플래그가 ZIGZAC의 차별점입니다.

### hermes-agent (NousResearch)
- **hermes-agent:** Hermes 스타일 XML Function-Calling 프로토콜 구현체. L2 프로토콜 레이어의 파서/직렬화기로 채택합니다.
- **hermes-agent-self-evolution:** 에이전트가 새 도구를 스스로 작성·테스트·등록하는 자기 진화 루프. L5에서 직접 실행됩니다.

### pi-mono / pi-skills (badlogic)
- **pi-mono:** 코어/스킬/어댑터/허브를 단일 모노레포에서 관리하는 프로젝트 구조를 채택합니다.
- **pi-skills:** `skill.json` 스키마와 의존성 그래프 시스템을 L4 Skills 레지스트리의 표준으로 사용합니다.

### openclaw / clawhub / skills (openclaw)
- **openclaw:** 도구를 `.tool.ts` 파일로 플러그인화하여 디렉토리 스캔 시 자동 등록하는 아키텍처를 L3에 적용합니다.
- **clawhub:** 도구 배포(publish)/설치(install)/검색(discover) 패턴을 Tool Hub의 외부 확장 모델로 채택합니다.

---

## 5. 레이어별 상세 설계

### L1 — 모델 라우터

**소스:** `docs/ZIGZAC/Brain.md`, `docs/ZIGZAC/agent_dna_schema.json`

#### 5.1.1 모델 자동 선택 로직

```
입력 복잡도 평가
       │
       ├─ 단순 질의 / 빠른 응답 필요
       │   └→ claude-haiku-4-5-20251001
       │       + fast-mode-2026-02-01 베타 헤더
       │
       ├─ 일반 코딩 / 멀티스텝 작업
       │   └→ claude-sonnet-4-5-20250929
       │       + advanced-tool-use 베타 헤더
       │
       ├─ 대용량 컨텍스트 (>200K 토큰)
       │   └→ claude-sonnet-4-5-20250929
       │       + context-1m-2025-08-07 베타 헤더
       │
       └─ 복잡 추론 / ultrathink 키워드 감지
           └→ claude-opus-4-1-20250805
               + interleaved-thinking-2025-05-14
               + effort-2025-11-24 (effort=high)
```

#### 5.1.2 멀티 프로바이더 지원

동일 모델을 4개 프로바이더로 라우팅합니다.

```
firstParty  →  api.anthropic.com
Bedrock     →  us-east-1 (기본) + b_5 리전 오버라이드
Vertex AI   →  us-central1 (기본) + b_5 리전 오버라이드
Foundry     →  Azure AI Foundry 엔드포인트
```

#### 5.1.3 에이전트 상태 기계

`agent_dna_schema.json`의 89개 필드를 전역 싱글톤 `G8`으로 관리합니다. 주요 필드:

| 필드 | 역할 |
|-----|------|
| `totalCostUSD` | 세션 전체 누적 비용 추적 |
| `turnToolCount` | 현재 턴 도구 호출 횟수 |
| `turnClassifierDurationMs` | 권한 분류기 실행 시간 |
| `projectRoot` | CLAUDE.md 탐색 기준 경로 |
| `isKairosActive` | KAIROS 자율 실행 모드 플래그 |

---

### L2 — Function-Calling 프로토콜

**소스:** hermes-agent, `docs/ZIGZAC/agent_dna_schema.json`

Hermes XML 포맷을 파싱하여 도구를 호출하고 결과를 구조화합니다.

```xml
<!-- 도구 호출 예시 -->
<tool_call>
  <name>WebFetch</name>
  <arguments>{"url": "https://example.com", "prompt": "주요 내용 요약"}</arguments>
</tool_call>

<!-- 도구 응답 예시 -->
<tool_response>
  <name>WebFetch</name>
  <content>{"result": "페이지 내용..."}</content>
</tool_response>
```

활성화 베타 헤더:
- `advanced-tool-use-2025-11-20` — 병렬 도구 호출
- `structured-outputs-2025-12-15` — 도구 응답 JSON 스키마 강제
- `tool-search-tool-2025-10-19` — Vertex/Bedrock 환경에서 도구 탐색

---

### L3 — Tool Hub

**소스:** `docs/ZIGZAC/control.md`, `docs/ZIGZAC/GEMINI_TOOLS_MANIFEST.json`, openclaw

#### 5.3.1 도구 카테고리

| 카테고리 | 도구 수 | 출처 |
|---------|--------|------|
| Agent 오케스트레이션 | 3 | GEMINI_TOOLS_MANIFEST (Agent, TaskOutput, TaskStop) |
| 파일시스템 | 5 | GEMINI_TOOLS_MANIFEST (FileRead, FileEdit, FileWrite, Glob, Grep) |
| 웹 | 2 | GEMINI_TOOLS_MANIFEST (WebFetch, WebSearch) |
| UI 상호작용 | 2 | GEMINI_TOOLS_MANIFEST (AskUserQuestion, ExitPlanMode) |
| MCP | 3 | GEMINI_TOOLS_MANIFEST (ListMcpResources, ReadMcpResource, Mcp) |
| 작업 관리 | 1 | GEMINI_TOOLS_MANIFEST (TodoWrite) |
| 기타 SDK | 5 | GEMINI_TOOLS_MANIFEST (Bash, Config, NotebookEdit, EnterWorktree, ExitWorktree) |
| 브라우저 제어 | 18 | control.md (chrome MCP 기반) |
| macOS 제어 | 24 | control.md (네이티브 접근성 API) |
| **합계** | **63** | |

#### 5.3.2 플러그인 등록 방식 (openclaw 패턴)

```
src/tools/
├── web/
│   ├── web-fetch.tool.ts
│   └── web-search.tool.ts
├── browser/
│   ├── navigate.tool.ts
│   └── computer.tool.ts
└── os/
    ├── screenshot.tool.ts
    └── computer-batch.tool.ts
```

- 시작 시 `tools/**/*.tool.ts` 스캔 → 자동 등록
- 각 도구는 `name / description / parameters / execute()` 인터페이스 구현

---

### L4 — Skills 레지스트리

**소스:** pi-skills, `skills-2025-10-02` 베타 헤더

스킬은 여러 도구를 조합하여 재사용 가능한 능력 단위로 캡슐화합니다.

#### 5.4.1 skill.json 스키마

```json
{
  "name": "web-research",
  "version": "1.0.0",
  "description": "웹 검색 후 내용 요약 및 구조화",
  "tools": ["WebSearch", "WebFetch", "TodoWrite"],
  "input": { "query": "string", "depth": "number" },
  "output": { "summary": "string", "sources": "string[]" }
}
```

#### 5.4.2 내장 스킬 목록

| 스킬 | 사용 도구 | 설명 |
|------|---------|------|
| `web-research` | WebSearch + WebFetch + TodoWrite | 웹 조사 및 구조화된 보고서 생성 |
| `code-review` | Bash + FileRead + Grep + Agent | 코드 품질 분석 및 개선 제안 |
| `computer-automation` | 42개 브라우저·OS 도구 | GUI 자동화 시나리오 |
| `file-refactor` | FileRead + FileEdit + Bash + Glob | 코드베이스 대규모 리팩토링 |
| `data-pipeline` | Bash + FileWrite + WebFetch | 데이터 수집·변환·저장 |

---

### L5 — Self-Evolution 루프

**소스:** hermes-agent-self-evolution, `advisor-tool-2026-03-01` 베타 헤더

```
작업 실행
    │
    ▼
실패 / 능력 부재 감지
    │
    ├─ 기존 도구로 해결 가능? ──→ 예: 일반 실행 경로로 복귀
    │
    └─ 아니오: Self-Evolution 진입
            │
            ▼
        새 도구 코드 생성 (모델이 직접 작성)
            │
            ▼
        격리된 워크트리에서 테스트 (isolation='worktree')
            │
            ├─ 테스트 실패 → 재생성 (최대 3회)
            │
            └─ 테스트 통과 → Tool Hub 핫 로드
                        │
                        ▼
                    Skills 레지스트리 업데이트
                        │
                        ▼
                    advisor-tool로 사용자에게 알림
```

---

### L6 — Handover 메모리 지속성

**소스:** `docs/ZIGZAC/Handover.md`, `compact-2026-01-12`, `context-management-2025-06-27`

#### 5.6.1 아키텍처

```
컨텍스트 창 사용률 모니터링
        │
        ├─ < 75%: 정상 실행
        │
        └─ ≥ 75%: Compaction 트리거
                │
                ├─ 전체 압축 (aWK): /compact 명령 또는 자동
                │   → 대화 전체를 구조화된 요약으로 대체
                │
                └─ 부분 압축 (gWK + N3Y):
                    → 최근 메시지만 요약, 이전 메시지 보존
                │
                ▼
        continuation summary 생성
        (vH7 프롬프트: Task Overview · Current State ·
         Discoveries · Next Steps · Context to Preserve)
                │
                ▼
        P18() — 다음 세션 시스템 프롬프트에 주입
                │
                ▼
        새 컨텍스트 창에서 즉시 작업 재개
```

#### 5.6.2 활성화 베타 헤더

| 헤더 | 목적 |
|------|------|
| `compact-2026-01-12` | 대화 컨텍스트 압축 |
| `context-management-2025-06-27` | API 레벨 자동 압축/트런케이션 |
| `context-1m-2025-08-07` | 1,000,000 토큰 컨텍스트 창 |
| `prompt-caching-scope-2026-01-05` | 프롬프트 캐싱으로 비용 절감 |
| `task-budgets-2026-03-13` | 태스크별 토큰 예산 상한 |

---

### L7 — 자율 오케스트레이터

**소스:** `docs/ZIGZAC/other.json` (KAIROS · UltraPlan · AFK 플래그)

#### 5.7.1 실행 모드

| 모드 | 피처 플래그 | 베타 헤더 | 설명 |
|------|-----------|---------|------|
| **크론 스케줄** | `tengu_kairos_cron` | — | 지정 시간에 자동 실행 |
| **내구성 크론** | `tengu_kairos_cron_durable` | — | 장애 후 재실행 보장 |
| **UltraPlan** | `tengu_ultraplan_*` | — | 복잡 계획 → 사용자 승인 → 자율 수행 |
| **AFK 모드** | — | `afk-mode-2026-01-31` | 장시간 무감독 자율 실행 |
| **Fast 모드** | `tengu_marble_sandcastle` | `fast-mode-2026-02-01` | Opus 4.6 빠른 응답 |
| **BYOC** | `tengu_ccr_bundle_seed_enabled` | `ccr-byoc-2025-07-29` | 사용자 자체 클라우드 실행 |
| **UltraThink** | `tengu_ultrathink` | `effort-2025-11-24` | ultrathink 키워드 → effort=high |

---

## 6. 데이터 흐름

```
사용자 입력
    │
    ▼
[L7] 오케스트레이터: 실행 모드 결정
    │
    ▼
[L6] 메모리 시스템: 이전 컨텍스트 로드 (있을 경우)
    │
    ▼
[L1] 모델 라우터: 복잡도 평가 → 모델 선택 → 베타 헤더 조립
    │
    ▼
[L2] 프로토콜 레이어: API 요청 → 스트리밍 응답 파싱
    │
    ├─ 일반 텍스트 → 사용자에게 반환
    │
    └─ <tool_call> 감지
            │
            ▼
        [L4] Skills 레지스트리: 스킬 매핑 여부 확인
            │
            ├─ 스킬 있음 → 스킬 실행 계획 생성
            │
            └─ 스킬 없음
                    │
                    ▼
                [L3] Tool Hub: 도구 검색 및 실행
                    │
                    ├─ 성공 → 결과 반환
                    │
                    └─ 실패
                            │
                            ▼
                        [L5] Self-Evolution: 새 도구 생성
    │
    ▼
[L6] 메모리 시스템: 컨텍스트 사용률 확인 → 필요 시 압축
    │
    ▼
다음 턴
```

---

## 7. 모델 선택 전략

```
작업 유형               권장 모델              활성화 베타 헤더
─────────────────────────────────────────────────────────────────
빠른 질의응답           haiku-4-5              fast-mode
단순 코드 작성          sonnet-4-5             advanced-tool-use
복잡 코딩 작업          sonnet-4-5             advanced-tool-use
                                              structured-outputs
대용량 문서 처리        sonnet-4-5             context-1m
장기 자율 작업          opus-4-1               afk-mode
                                              interleaved-thinking
                                              context-management
깊은 추론 (ultrathink)  opus-4-1               interleaved-thinking
                                              effort (high)
                                              redact-thinking
실시간 빠른 응답        opus-4-6 (Sonnet-46)   fast-mode
웹 검색 포함            sonnet-4-5             web-search-2025-03-05
```

---

## 8. 베타 헤더 주입 전략

ZIGZAC는 `Brain.md`에서 추출한 27개 베타 헤더를 조건에 따라 동적으로 조합합니다.

```typescript
// 의사 코드
function buildBetaHeaders(context: AgentContext): string[] {
  const headers: string[] = [];

  // 사고 계열
  if (context.model.supportsThinking && !context.disableThinking) {
    headers.push("interleaved-thinking-2025-05-14");
    if (!context.showThinkingSummaries) {
      headers.push("redact-thinking-2026-02-12");
    }
  }

  // 컨텍스트 계열
  if (context.inputTokens > 500_000) {
    headers.push("context-1m-2025-08-07");
  }
  if (context.useApiContextManagement) {
    headers.push("context-management-2025-06-27");
  }

  // 도구 계열
  if (context.isFirstParty || context.isFoundry) {
    headers.push("advanced-tool-use-2025-11-20");
  } else {
    headers.push("tool-search-tool-2025-10-19");
  }

  // 실행 모드 계열
  if (context.isAgentic) {
    headers.push("prompt-caching-scope-2026-01-05");
  }
  if (context.afkMode && !context.fastMode) {
    headers.push("afk-mode-2026-01-31");
  }
  if (context.fastMode && context.isFastModeSupported) {
    headers.push("fast-mode-2026-02-01");
  }

  return headers;
}
```

---

## 9. 기술 스택

| 구성요소 | 선택 | 이유 |
|---------|------|------|
| **언어** | TypeScript | GEMINI_TOOLS_MANIFEST가 TS 타입 기반, cli.js 역분석 자산과 동일 생태계 |
| **런타임** | Node.js 22+ | Anthropic SDK 네이티브 지원 |
| **AI SDK** | @anthropic-ai/sdk | Brain.md 모델 레지스트리 + 27개 베타 헤더 직접 활용 |
| **도구 확장** | MCP (Model Context Protocol) | mcp-client/servers 베타 헤더 확보, 생태계 최대 |
| **프로젝트 구조** | pi-mono 모노레포 | 코어/스킬/어댑터/허브 단일 저장소 관리 |
| **테스트** | Vitest | 빠른 실행, TypeScript 네이티브 |
| **패키지 관리** | pnpm workspaces | 모노레포 의존성 최적화 |

---

## 10. 보안 고려사항

- **권한 분류기:** `agent_dna_schema.json`의 `turnClassifierDurationMs` 필드가 추적하는 분류기를 구현하여 도구 실행 전 허용/거부 판단
- **도구 허용 목록:** OS 제어 도구(`control.md`)는 `request_access` 선행 호출 필수 — 세션 시작 시 사용자 승인
- **Self-Evolution 격리:** 새 도구 생성 시 `isolation='worktree'` 옵션으로 격리된 git worktree에서 테스트
- **BYOC 격리:** `ccr-byoc-2025-07-29` 모드에서는 사용자 클라우드 자격증명 외부 유출 차단
- **AST 보안 검사:** `tengu_birch_trellis` 플래그 — Bash 명령 인젝션 AST 레벨 검사 활성화
- **베타 헤더 범위 제어:** `prompt-caching-scope-2026-01-05`로 캐시 범위를 명시적으로 제한

---

## 11. 구현 로드맵

### Phase 1 — 코어 엔진 (L1 + L2)
- [ ] `Brain.md` → TypeScript 모델 레지스트리 타입 변환
- [ ] 베타 헤더 동적 주입 엔진 구현
- [ ] `agent_dna_schema.json` → G8 전역 상태 싱글톤 구현
- [ ] hermes XML Function-Calling 파서 구현
- [ ] 기본 API 요청/응답 스트리밍 파이프라인

### Phase 2 — Tool Hub (L3)
- [ ] openclaw 플러그인 인터페이스 정의
- [ ] `GEMINI_TOOLS_MANIFEST.json` 21개 도구 플러그인 변환
- [ ] `control.md` 42개 도구 플러그인 변환
- [ ] 도구 자동 스캔·등록 시스템
- [ ] tool-search-tool 베타 헤더 연동

### Phase 3 — Skills 레지스트리 (L4)
- [ ] pi-skills `skill.json` 스키마 TypeScript 타입 정의
- [ ] 의존성 그래프 해석기 구현
- [ ] 내장 스킬 5종 구현 (web-research, code-review, computer-automation, file-refactor, data-pipeline)
- [ ] `skills-2025-10-02` 베타 헤더 → Anthropic Skills API 연동

### Phase 4 — 메모리 지속성 (L6)
- [ ] `Handover.md` vH7 프롬프트 → continuation summary 생성기 구현
- [ ] 컨텍스트 사용률 모니터링 (75% 임계값)
- [ ] 전체 압축(aWK) / 부분 압축(gWK) 구현
- [ ] `context-management-2025-06-27` API 레벨 압축 연동
- [ ] 다음 세션 시스템 프롬프트 주입(P18) 구현

### Phase 5 — 자율화 (L5 + L7)
- [ ] hermes self-evolution 실패 감지 루프 구현
- [ ] 새 도구 코드 생성 → 격리 테스트 → 핫 로드 파이프라인
- [ ] KAIROS 크론 스케줄러 (`tengu_kairos_cron`) 구현
- [ ] UltraPlan 사용자 승인 흐름 구현
- [ ] AFK 모드 / BYOC 모드 연동

# Claude 내부 모델 식별자 & 베타 헤더 전체 목록

> **출처:** `cli.js` v2.1.88 — `b_5`, `n66`(모델 레지스트리), `x38`(최대 토큰 테이블), `anthropic-beta` 헤더 상수군  
> **분해 깊이:** 1단계  
> **추출 일시:** 2026-04-15

---

## 1. `b_5` — Vertex AI 리전 오버라이드 맵

> 각 모델 prefix → 해당 Vertex 리전을 덮어쓸 **환경변수명** 매핑.  
> 환경변수를 설정하면 해당 모델 요청이 지정 리전으로 라우팅됨.

```json
{
  "source": "b_5 (VertexRegionOverrides)",
  "description": "모델 prefix → Vertex AI 리전 env var 이름 매핑. 환경변수를 설정하면 해당 모델 요청이 특정 리전으로 라우팅됨.",
  "entries": [
    { "model_prefix": "claude-haiku-4-5",  "env_var": "VERTEX_REGION_CLAUDE_HAIKU_4_5" },
    { "model_prefix": "claude-3-5-haiku",  "env_var": "VERTEX_REGION_CLAUDE_3_5_HAIKU" },
    { "model_prefix": "claude-3-5-sonnet", "env_var": "VERTEX_REGION_CLAUDE_3_5_SONNET" },
    { "model_prefix": "claude-3-7-sonnet", "env_var": "VERTEX_REGION_CLAUDE_3_7_SONNET" },
    { "model_prefix": "claude-opus-4-1",   "env_var": "VERTEX_REGION_CLAUDE_4_1_OPUS" },
    { "model_prefix": "claude-opus-4",     "env_var": "VERTEX_REGION_CLAUDE_4_0_OPUS" },
    { "model_prefix": "claude-sonnet-4-6", "env_var": "VERTEX_REGION_CLAUDE_4_6_SONNET" },
    { "model_prefix": "claude-sonnet-4-5", "env_var": "VERTEX_REGION_CLAUDE_4_5_SONNET" },
    { "model_prefix": "claude-sonnet-4",   "env_var": "VERTEX_REGION_CLAUDE_4_0_SONNET" }
  ]
}
```

---

## 2. 모델 레지스트리 (`n66`) — 크로스 프로바이더 ID 전체

> 내부 변수 `n66`에 정의된 공식·비공개 모델 ID.  
> 각 모델은 `firstParty` / `bedrock` / `vertex` / `foundry` 4개 형식으로 표현됨.

```json
{
  "source": "n66 (ModelRegistry), TF6 initializer",
  "description": "Claude Code 내부 모델 레지스트리. 4개 프로바이더별 정확한 모델 식별자.",
  "models": [
    {
      "key": "haiku35",
      "firstParty": "claude-3-5-haiku-20241022",
      "bedrock":    "us.anthropic.claude-3-5-haiku-20241022-v1:0",
      "vertex":     "claude-3-5-haiku@20241022",
      "foundry":    "claude-3-5-haiku"
    },
    {
      "key": "haiku45",
      "firstParty": "claude-haiku-4-5-20251001",
      "bedrock":    "us.anthropic.claude-haiku-4-5-20251001-v1:0",
      "vertex":     "claude-haiku-4-5@20251001",
      "foundry":    "claude-haiku-4-5"
    },
    {
      "key": "sonnet35",
      "firstParty": "claude-3-5-sonnet-20241022",
      "bedrock":    "anthropic.claude-3-5-sonnet-20241022-v2:0",
      "vertex":     "claude-3-5-sonnet-v2@20241022",
      "foundry":    "claude-3-5-sonnet"
    },
    {
      "key": "sonnet37",
      "firstParty": "claude-3-7-sonnet-20250219",
      "bedrock":    "us.anthropic.claude-3-7-sonnet-20250219-v1:0",
      "vertex":     "claude-3-7-sonnet@20250219",
      "foundry":    "claude-3-7-sonnet"
    },
    {
      "key": "sonnet40",
      "firstParty": "claude-sonnet-4-20250514",
      "bedrock":    "us.anthropic.claude-sonnet-4-20250514-v1:0",
      "vertex":     "claude-sonnet-4@20250514",
      "foundry":    "claude-sonnet-4"
    },
    {
      "key": "sonnet45",
      "firstParty": "claude-sonnet-4-5-20250929",
      "bedrock":    "us.anthropic.claude-sonnet-4-5-20250929-v1:0",
      "vertex":     "claude-sonnet-4-5@20250929",
      "foundry":    "claude-sonnet-4-5"
    },
    {
      "key": "sonnet46",
      "firstParty": "claude-sonnet-4-6",
      "bedrock":    "us.anthropic.claude-sonnet-4-6",
      "vertex":     "claude-sonnet-4-6",
      "foundry":    "claude-sonnet-4-6",
      "note":       "날짜 suffix 없음 — 최신 고정 핀"
    },
    {
      "key": "opus40",
      "firstParty": "claude-opus-4-20250514",
      "bedrock":    "us.anthropic.claude-opus-4-20250514-v1:0",
      "vertex":     "claude-opus-4@20250514",
      "foundry":    "claude-opus-4"
    },
    {
      "key": "opus41",
      "firstParty": "claude-opus-4-1-20250805",
      "bedrock":    "us.anthropic.claude-opus-4-1-20250805-v1:0",
      "vertex":     "claude-opus-4-1@20250805",
      "foundry":    "claude-opus-4-1"
    },
    {
      "key": "opus45",
      "firstParty": "claude-opus-4-5-20251101",
      "bedrock":    "us.anthropic.claude-opus-4-5-20251101-v1:0",
      "vertex":     "claude-opus-4-5@20251101",
      "foundry":    "claude-opus-4-5"
    },
    {
      "key": "opus46",
      "firstParty": "claude-opus-4-6",
      "bedrock":    "us.anthropic.claude-opus-4-6-v1",
      "vertex":     "claude-opus-4-6",
      "foundry":    "claude-opus-4-6",
      "note":       "날짜 suffix 없음 — 최신 고정 핀"
    }
  ]
}
```

---

## 3. 최대 출력 토큰 테이블 (`x38`) — Opus 4 계열 특별 제한

> `x38` 테이블: 특정 모델에 적용되는 **8192 토큰 출력 상한** 오버라이드.  
> 비공개 alias `claude-4-opus-20250514` 포함 (역순 버전 표기).

```json
{
  "source": "x38 (MaxOutputTokensOverride), hr8 initializer",
  "description": "일부 Opus 4 모델에 적용되는 최대 출력 토큰 제한(8192). 프로바이더별 모든 형식 alias 포함.",
  "max_output_tokens": 8192,
  "affected_model_aliases": [
    { "id": "claude-opus-4-20250514",              "provider": "firstParty" },
    { "id": "claude-opus-4-0",                     "provider": "firstParty (alias)" },
    { "id": "claude-4-opus-20250514",              "provider": "INTERNAL ALIAS (역순 표기)", "note": "비공개 내부 식별자" },
    { "id": "anthropic.claude-opus-4-20250514-v1:0","provider": "Bedrock" },
    { "id": "claude-opus-4@20250514",              "provider": "Vertex AI" },
    { "id": "claude-opus-4-1-20250805",            "provider": "firstParty" },
    { "id": "anthropic.claude-opus-4-1-20250805-v1:0","provider": "Bedrock" },
    { "id": "claude-opus-4-1@20250805",            "provider": "Vertex AI" }
  ]
}
```

---

## 4. 기타 내부 모델 식별자

> 레지스트리 외부에서 하드코딩된 특수 목적 식별자들.

```json
{
  "source": "scattered constants in cli.js",
  "identifiers": [
    {
      "id": "claude-code-20250219",
      "var": "mJ8",
      "purpose": "에이전틱(agentic) 쿼리 시 anthropic-beta 헤더에 자동 추가되는 Claude Code 전용 식별자. API가 Code 클라이언트임을 인식하게 함."
    },
    {
      "id": "claude-4-opus-20250514",
      "var": "x38 key",
      "purpose": "claude-opus-4-20250514의 역순 표기 비공개 alias. 최대 출력 토큰 테이블에서만 사용."
    },
    {
      "id": "claude-opus-4-6",
      "var": "G06.firstParty",
      "purpose": "날짜 미고정 최신 Opus 4.6 핀. bedrock: claude-opus-4-6-v1 (v1 접미사)"
    },
    {
      "id": "claude-sonnet-4-6",
      "var": "kJ1.firstParty",
      "purpose": "날짜 미고정 최신 Sonnet 4.6 핀."
    }
  ]
}
```

---

## 5. `anthropic-beta` 베타 헤더 전체 목록

> 총 **27개**. 내부 변수명, 활성화 조건, 사용처 포함.

```json
{
  "source": "anthropic-beta header constants",
  "description": "API 요청 시 anthropic-beta 헤더에 주입되는 베타 기능 식별자 전체 목록.",
  "count": 27,
  "headers": [
    {
      "beta": "interleaved-thinking-2025-05-14",
      "var": "j2q",
      "activation": "DISABLE_INTERLEAVED_THINKING 환경변수 미설정 + 대상 모델이 thinking 지원 시 자동 활성화",
      "purpose": "확장 사고(Extended Thinking)를 응답 스트림에 인터리브"
    },
    {
      "beta": "context-1m-2025-08-07",
      "var": "s66",
      "activation": "모델이 1M 컨텍스트 지원 시 + 입력 토큰이 충분히 클 때",
      "purpose": "최대 입력 컨텍스트를 1,000,000 토큰으로 확장"
    },
    {
      "beta": "context-management-2025-06-27",
      "var": "pJ8",
      "activation": "USE_API_CONTEXT_MANAGEMENT 환경변수 또는 내부 플래그 활성화 + firstParty/foundry 프로바이더",
      "purpose": "API 레벨 컨텍스트 관리 (자동 압축/트런케이션)"
    },
    {
      "beta": "structured-outputs-2025-12-15",
      "var": "t66",
      "activation": "에이전틱 쿼리 + tengu_tool_pear 피처 플래그 + 지원 모델",
      "purpose": "구조화된 JSON 출력 (structured outputs) 스키마 강제"
    },
    {
      "beta": "structured-outputs-2025-11-13",
      "var": "(inline)",
      "activation": "Java SDK 예제에서 직접 지정",
      "purpose": "structured-outputs 이전 버전 (11-13)"
    },
    {
      "beta": "web-search-2025-03-05",
      "var": "yX1",
      "activation": "vertex 프로바이더 + 지원 모델, 또는 foundry 프로바이더",
      "purpose": "웹 검색 도구 활성화"
    },
    {
      "beta": "advanced-tool-use-2025-11-20",
      "var": "H2q",
      "activation": "firstParty/foundry 프로바이더에서 tool use 요청 시 (vertex/bedrock는 J2q 사용)",
      "purpose": "고급 도구 사용 기능 (tool_choice, parallel tool calls 등)"
    },
    {
      "beta": "tool-search-tool-2025-10-19",
      "var": "J2q",
      "activation": "vertex 또는 bedrock 프로바이더에서 tool use 요청 시",
      "purpose": "tool-search-tool: 도구 탐색 도구 (advanced-tool-use의 vertex/bedrock 대응 버전)"
    },
    {
      "beta": "effort-2025-11-24",
      "var": "EX1",
      "activation": "effort 파라미터 사용 시 (사고 깊이 조절)",
      "purpose": "모델 추론 노력(effort) 수준 제어 파라미터 활성화"
    },
    {
      "beta": "task-budgets-2026-03-13",
      "var": "LX1",
      "activation": "task_budget 파라미터(토큰 예산) 설정 시 자동 추가",
      "purpose": "태스크별 토큰 예산 제어"
    },
    {
      "beta": "prompt-caching-scope-2026-01-05",
      "var": "pF6",
      "activation": "에이전틱 쿼리 시 기본 활성화",
      "purpose": "프롬프트 캐싱 범위 제어"
    },
    {
      "beta": "fast-mode-2026-02-01",
      "var": "RX1",
      "activation": "fastMode 옵션 + BK() + oJ() + jJ(model) 조건 모두 충족 시",
      "purpose": "Fast Mode (Opus 4.6 빠른 응답 모드) 활성화"
    },
    {
      "beta": "redact-thinking-2026-02-12",
      "var": "BJ8",
      "activation": "에이전틱 쿼리 + thinking 지원 + showThinkingSummaries !== true",
      "purpose": "사고 과정(thinking) 내용을 응답에서 제거(redact)"
    },
    {
      "beta": "afk-mode-2026-01-31",
      "var": "x06",
      "activation": "fastMode가 아닌 경우 + firstParty/foundry 프로바이더 + 조건 충족 시",
      "purpose": "AFK 모드 — 장시간 자율 실행 세션"
    },
    {
      "beta": "advisor-tool-2026-03-01",
      "var": "hX1",
      "activation": "advisor 기능 활성화 시",
      "purpose": "어드바이저 도구 (사용자 조언 생성 전용 툴)"
    },
    {
      "beta": "oauth-2025-04-20",
      "var": "rM",
      "activation": "OAuth 인증 흐름 (claude.ai 로그인) 시 기본 추가",
      "purpose": "OAuth 2.0 인증 API 활성화"
    },
    {
      "beta": "environments-2025-11-01",
      "var": "LVY",
      "activation": "environments API 엔드포인트 사용 시",
      "purpose": "환경(environment) 관리 API (샌드박스/실행 환경 제어)"
    },
    {
      "beta": "ccr-byoc-2025-07-29",
      "var": "(inline)",
      "activation": "CCR(Claude Code Remote) BYOC(Bring Your Own Cloud) 세션 요청 시",
      "purpose": "CCR BYOC 모드 — 사용자 클라우드 인프라에서 에이전트 실행"
    },
    {
      "beta": "ccr-triggers-2026-01-30",
      "var": "VqY",
      "activation": "RemoteTrigger 도구 사용 시 anthropic-beta 헤더에 자동 추가",
      "purpose": "CCR 트리거 — 원격 에이전트 이벤트 트리거 제어"
    },
    {
      "beta": "mcp-servers-2025-12-04",
      "var": "ROz",
      "activation": "MCP 서버 목록 API 호출 시",
      "purpose": "MCP(Model Context Protocol) 서버 관리 API"
    },
    {
      "beta": "mcp-client-2025-11-20",
      "var": "(SDK example)",
      "activation": "MCP 클라이언트로 모델 호출 시",
      "purpose": "MCP 클라이언트 프로토콜 베타"
    },
    {
      "beta": "skills-2025-10-02",
      "var": "(inline)",
      "activation": "Skills API (버전 생성 등) 호출 시",
      "purpose": "Skills(재사용 가능한 에이전트 능력 단위) API"
    },
    {
      "beta": "files-api-2025-04-14",
      "var": "(inline)",
      "activation": "Files API 목록 조회 시 자동 추가",
      "purpose": "파일 업로드/관리 API 활성화"
    },
    {
      "beta": "message-batches-2024-09-24",
      "var": "(inline)",
      "activation": "메시지 배치 생성 API 호출 시",
      "purpose": "비동기 메시지 배치 처리 API"
    },
    {
      "beta": "token-counting-2024-11-01",
      "var": "(inline)",
      "activation": "토큰 카운팅 API(/v1/messages/count_tokens) 호출 시",
      "purpose": "요청 전 토큰 수 사전 계산"
    },
    {
      "beta": "compact-2026-01-12",
      "var": "(SDK example)",
      "activation": "컨텍스트 압축(compact) 사용 시",
      "purpose": "컨텍스트 압축 — 긴 대화를 압축하여 컨텍스트 절약"
    },
    {
      "beta": "bedrock-2023-05-31",
      "var": "(inline)",
      "activation": "Bedrock 프로바이더 사용 시",
      "purpose": "AWS Bedrock API 버전 헤더"
    },
    {
      "beta": "vertex-2023-10-16",
      "var": "(inline)",
      "activation": "Vertex AI 프로바이더 사용 시",
      "purpose": "Google Vertex AI API 버전 헤더"
    }
  ]
}
```

---

## 6. 베타 헤더 빠른 참조 테이블

| 베타 식별자 | 내부 변수 | 카테고리 | 주요 목적 |
|------------|----------|---------|----------|
| `interleaved-thinking-2025-05-14` | `j2q` | 사고 | Extended Thinking 스트림 인터리브 |
| `redact-thinking-2026-02-12` | `BJ8` | 사고 | 사고 내용 응답에서 제거 |
| `effort-2025-11-24` | `EX1` | 사고 | 추론 노력 수준 제어 |
| `structured-outputs-2025-12-15` | `t66` | 출력 | JSON 스키마 강제 출력 |
| `structured-outputs-2025-11-13` | (inline) | 출력 | 구 버전 structured output |
| `context-1m-2025-08-07` | `s66` | 컨텍스트 | 1M 토큰 컨텍스트 창 |
| `context-management-2025-06-27` | `pJ8` | 컨텍스트 | API 레벨 자동 컨텍스트 관리 |
| `prompt-caching-scope-2026-01-05` | `pF6` | 컨텍스트 | 프롬프트 캐싱 범위 |
| `compact-2026-01-12` | (inline) | 컨텍스트 | 대화 컨텍스트 압축 |
| `task-budgets-2026-03-13` | `LX1` | 실행 | 태스크별 토큰 예산 |
| `fast-mode-2026-02-01` | `RX1` | 실행 | Fast Mode (빠른 응답) |
| `afk-mode-2026-01-31` | `x06` | 실행 | AFK 자율 실행 모드 |
| `web-search-2025-03-05` | `yX1` | 도구 | 웹 검색 도구 |
| `advanced-tool-use-2025-11-20` | `H2q` | 도구 | 고급 도구 사용 (firstParty) |
| `tool-search-tool-2025-10-19` | `J2q` | 도구 | 도구 탐색 (Vertex/Bedrock) |
| `advisor-tool-2026-03-01` | `hX1` | 도구 | 어드바이저 도구 |
| `mcp-client-2025-11-20` | (inline) | MCP | MCP 클라이언트 프로토콜 |
| `mcp-servers-2025-12-04` | `ROz` | MCP | MCP 서버 관리 API |
| `skills-2025-10-02` | (inline) | API | Skills API |
| `files-api-2025-04-14` | (inline) | API | Files API |
| `message-batches-2024-09-24` | (inline) | API | 메시지 배치 처리 |
| `token-counting-2024-11-01` | (inline) | API | 사전 토큰 카운팅 |
| `oauth-2025-04-20` | `rM` | 인증 | OAuth 2.0 |
| `environments-2025-11-01` | `LVY` | 인프라 | 실행 환경 관리 |
| `ccr-byoc-2025-07-29` | (inline) | CCR | BYOC 원격 실행 |
| `ccr-triggers-2026-01-30` | `VqY` | CCR | 원격 에이전트 트리거 |
| `bedrock-2023-05-31` | (inline) | 프로바이더 | AWS Bedrock API 버전 |
| `vertex-2023-10-16` | (inline) | 프로바이더 | GCP Vertex AI API 버전 |

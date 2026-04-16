# Handover — Claude Code 컨텍스트 연속성 시스템 전체 문서

> **출처:** `cli.js` v2.1.88  
> **관련 변수:** `vH7`, `N3Y`, `rWK`, 함수 `aWK()`, `gWK()`, `P18()`, `y3Y()`, `WWK()`  
> **목적:** 컨텍스트 창이 가득 찼을 때 Claude가 작업을 다음 세션에 인계하는 메커니즘 전체

---

## 아키텍처 개요

```
컨텍스트 창 초과 감지
        │
        ▼
  [Compaction 트리거]
        │
   ┌────┴────────────────────────────────┐
   │                                     │
   ▼                                     ▼
전체 압축 (aWK)                    부분 압축 (gWK + N3Y)
(대화 전체를 요약)                 (최근 메시지만 요약, 이전은 보존)
        │                                     │
        └──────────────┬──────────────────────┘
                       ▼
              [요약 텍스트 생성]
              <analysis>...</analysis>
              <summary>...</summary>
                       │
                       ▼
                 y3Y() 후처리
                 (태그 제거 및 정규화)
                       │
                       ▼
                 P18() — 다음 세션
                 시스템 프롬프트 조립
```

---

## 1. `vH7` — Continuation Summary 프롬프트 (전문)

> **위치:** `cli.js:88098`  
> **용도:** 작업 미완료 시 인계 메모 작성 지시 — 다음 인스턴스가 즉시 재개할 수 있도록 구조화된 요약 요청

```
You have been working on the task described above but have not yet completed it.
Write a continuation summary that will allow you (or another instance of yourself)
to resume work efficiently in a future context window where the conversation history
will be replaced with this summary. Your summary should be structured, concise,
and actionable. Include:

1. Task Overview
   The user's core request and success criteria
   Any clarifications or constraints they specified

2. Current State
   What has been completed so far
   Files created, modified, or analyzed (with paths if relevant)
   Key outputs or artifacts produced

3. Important Discoveries
   Technical constraints or requirements uncovered
   Decisions made and their rationale
   Errors encountered and how they were resolved
   What approaches were tried that didn't work (and why)

4. Next Steps
   Specific actions needed to complete the task
   Any blockers or open questions to resolve
   Priority order if multiple steps remain

5. Context to Preserve
   User preferences or style requirements
   Domain-specific details that aren't obvious
   Any promises made to the user

Be concise but complete—err on the side of including information that would
prevent duplicate work or repeated mistakes. Write in a way that enables
immediate resumption of the task.

Wrap your summary in <summary></summary> tags.
```

**특이사항:** `vH7`는 작업 중단/미완성 시나리오 전용. 대화 전체 압축(`compact`)과 다르게, **진행 중인 단일 태스크**의 인계에 특화됨.

---

## 2. 전체 압축 프롬프트 — `aWK()` 함수 생성 (전문)

> **위치:** `cli.js:9474xxx`  
> **용도:** `/compact` 명령 또는 자동 컨텍스트 압축 시, 대화 전체를 하나의 구조화된 요약으로 대체

### 선두 강제 지시 (Critical Preamble)

```
CRITICAL: Respond with TEXT ONLY. Do NOT call any tools.

- Do NOT use Read, Bash, Grep, Glob, Edit, Write, or ANY other tool.
- You already have all the context you need in the conversation above.
- Tool calls will be REJECTED and will waste your only turn — you will fail the task.
- Your entire response must be plain text: an <analysis> block followed by a <summary> block.
```

### 본문 지시 (전문)

```
Your task is to create a detailed summary of the conversation so far, paying close
attention to the user's explicit requests and your previous actions.
This summary should be thorough in capturing technical details, code patterns, and
architectural decisions that would be essential for continuing development work
without losing context.

Before providing your final summary, wrap your analysis in <analysis> tags to organize
your thoughts and ensure you've covered all necessary points. In your analysis process:

1. Chronologically analyze each message and section of the conversation.
   For each section thoroughly identify:
   - The user's explicit requests and intents
   - Your approach to addressing the user's requests
   - Key decisions, technical concepts and code patterns
   - Specific details like:
     - file names
     - full code snippets
     - function signatures
     - file edits
   - Errors that you ran into and how you fixed them
   - Pay special attention to specific user feedback that you received,
     especially if the user told you to do something differently.
2. Double-check for technical accuracy and completeness, addressing each required
   element thoroughly.

Your summary should include the following sections:

1. Primary Request and Intent:
   Capture all of the user's explicit requests and intents in detail

2. Key Technical Concepts:
   List all important technical concepts, technologies, and frameworks discussed.

3. Files and Code Sections:
   Enumerate specific files and code sections examined, modified, or created.
   Pay special attention to the most recent messages and include full code snippets
   where applicable and include a summary of why this file read or edit is important.

4. Errors and fixes:
   List all errors that you ran into, and how you fixed them. Pay special attention
   to specific user feedback that you received, especially if the user told you to
   do something differently.

5. Problem Solving:
   Document problems solved and any ongoing troubleshooting efforts.

6. All user messages:
   List ALL user messages that are not tool results. These are critical for
   understanding the users' feedback and changing intent.

7. Pending Tasks:
   Outline any pending tasks that you have explicitly been asked to work on.

8. Current Work:
   Describe in detail precisely what was being worked on immediately before this
   summary request, paying special attention to the most recent messages from both
   user and assistant. Include file names and code snippets where applicable.

9. Optional Next Step:
   List the next step that you will take that is related to the most recent work
   you were doing. IMPORTANT: ensure that this step is DIRECTLY in line with the
   user's most recent explicit requests, and the task you were working on immediately
   before this summary request. If your last task was concluded, then only list next
   steps if they are explicitly in line with the users request. Do not start on
   tangential requests or really old requests that were already completed without
   confirming with the user first.

   If there is a next step, include direct quotes from the most recent conversation
   showing exactly what task you were working on and where you left off. This should
   be verbatim to ensure there's no drift in task interpretation.
```

### 응답 형식 예시 (Example Block)

```xml
<example>
<analysis>
[Your thought process, ensuring all points are covered thoroughly and accurately]
</analysis>

<summary>
1. Primary Request and Intent:
   [Detailed description]

2. Key Technical Concepts:
   - [Concept 1]
   - [Concept 2]
   - [...]

3. Files and Code Sections:
   - [File Name 1]
      - [Summary of why this file is important]
      - [Summary of the changes made to this file, if any]
      - [Important Code Snippet]
   - [File Name 2]
      - [Important Code Snippet]
   - [...]

4. Errors and fixes:
    - [Detailed description of error 1]:
      - [How you fixed the error]
      - [User feedback on the error if any]
    - [...]

5. Problem Solving:
   [Description of solved problems and ongoing troubleshooting]

6. All user messages:
    - [Detailed non tool use user message]
    - [...]

7. Pending Tasks:
   - [Task 1]
   - [Task 2]
   - [...]

8. Current Work:
   [Precise description of current work]

9. Optional Next Step:
   [Optional Next step to take]

</summary>
</example>
```

### 후미 지시 (compact instructions 사용자 커스터마이징)

```
Please provide your summary based on the conversation so far, following this structure
and ensuring precision and thoroughness in your response.

There may be additional summarization instructions provided in the included context.
If so, remember to follow these instructions when creating the above summary.
Examples of instructions include:

<example>
## Compact Instructions
When summarizing the conversation focus on typescript code changes and also remember
the mistakes you made and how you fixed them.
</example>

<example>
# Summary instructions
When you are using compact - please focus on test output and code changes.
Include file reads verbatim.
</example>
```

### 후미 공통 리마인더 (`rWK`)

```


REMINDER: Do NOT call any tools. Respond with plain text only —
```

---

## 3. 부분 압축 프롬프트 — `N3Y` 변수 (전문)

> **위치:** `cli.js:9470xxx`  
> **용도:** SM(Session Memory) Compact — 이전 컨텍스트는 보존하고 **최근 메시지만** 요약할 때 사용

```
Your task is to create a detailed summary of the RECENT portion of the conversation
— the messages that follow earlier retained context. The earlier messages are being
kept intact and do NOT need to be summarized. Focus your summary on what was
discussed, learned, and accomplished in the recent messages only.

Before providing your final summary, wrap your analysis in <analysis> tags to organize
your thoughts and ensure you've covered all necessary points. In your analysis process:

1. Analyze the recent messages chronologically. For each section thoroughly identify:
   - The user's explicit requests and intents
   - Your approach to addressing the user's requests
   - Key decisions, technical concepts and code patterns
   - Specific details like:
     - file names
     - full code snippets
     - function signatures
     - file edits
   - Errors that you ran into and how you fixed them
   - Pay special attention to specific user feedback that you received,
     especially if the user told you to do something differently.
2. Double-check for technical accuracy and completeness, addressing each required
   element thoroughly.

Your summary should include the following sections:

1. Primary Request and Intent:
   Capture the user's explicit requests and intents from the recent messages

2. Key Technical Concepts:
   List important technical concepts, technologies, and frameworks discussed recently.

3. Files and Code Sections:
   Enumerate specific files and code sections examined, modified, or created.
   Include full code snippets where applicable and include a summary of why this
   file read or edit is important.

4. Errors and fixes:
   List errors encountered and how they were fixed.

5. Problem Solving:
   Document problems solved and any ongoing troubleshooting efforts.

6. All user messages:
   List ALL user messages from the recent portion that are not tool results.

7. Pending Tasks:
   Outline any pending tasks from the recent messages.

8. Current Work:
   Describe precisely what was being worked on immediately before this summary request.

9. Optional Next Step:
   List the next step related to the most recent work.
   Include direct quotes from the most recent conversation.

[응답 예시 구조는 aWK()와 동일]

Please provide your summary based on the RECENT messages only (after the retained
earlier context), following this structure and ensuring precision and thoroughness
in your response.
```

**전체 압축(`aWK`)과의 차이:**

| 항목 | `aWK` (전체 압축) | `N3Y` (부분 압축) |
|------|-----------------|-----------------|
| 대상 범위 | 대화 전체 | 최근 메시지만 |
| 이전 컨텍스트 | 요약으로 대체 | 보존 (건드리지 않음) |
| 사용 시점 | `/compact` 또는 자동 | SM Compact (세션 메모리) |
| 섹션 8 | "Current Work" | "Current Work" (동일) |
| 섹션 9 | "Work Completed" → 부재 | "Optional Next Step" |

---

## 4. 세션 인계 헤더 — `P18()` 함수 (전문)

> **위치:** `cli.js:9480xxx`  
> **용도:** 압축된 요약을 다음 세션의 **시스템 프롬프트 첫머리**로 조립. 새 컨텍스트 창에서 Claude가 가장 먼저 읽는 텍스트.

```
함수 시그니처: P18(summaryText, autoResume, transcriptPath, hasRecentMessages)

파라미터:
  q = summaryText      — y3Y()로 후처리된 요약 텍스트
  K = autoResume       — true면 "계속 진행" 지시 자동 추가
  _ = transcriptPath   — 전체 트랜스크립트 파일 경로 (있을 경우)
  z = hasRecentMessages — 최근 메시지 보존 여부
```

### 생성되는 시스템 프롬프트 구조

```
This session is being continued from a previous conversation that ran out of context.
The summary below covers the earlier portion of the conversation.

[y3Y(요약) 정규화된 텍스트]

[transcriptPath가 있을 경우 추가:]
If you need specific details from before compaction (like exact code snippets,
error messages, or content you generated), read the full transcript at: {path}

[hasRecentMessages가 true일 경우 추가:]
Recent messages are preserved verbatim.

[autoResume이 true일 경우 추가:]
Continue the conversation from where it left off without asking the user any
further questions. Resume directly — do not acknowledge the summary, do not recap
what was happening, do not preface with "I'll continue" or similar. Pick up the
last task as if the break never happened.
```

---

## 5. 요약 후처리 — `y3Y()` 함수

> 모델이 생성한 raw 출력에서 태그를 제거하고 정규화.

```javascript
// 동작:
// 1. <analysis>...</analysis> 블록 전체 제거
// 2. <summary>...</summary> 내용 추출 → "Summary:\n{내용}" 형식으로 재조립
// 3. 연속 빈 줄(\n\n+) → 최대 \n\n으로 정규화
// 4. 앞뒤 공백 trim
```

---

## 6. 시스템 전체 흐름 요약

```
[컨텍스트 임박]
      │
      ├─ 자동 감지 (tengu_auto_compact_succeeded 이벤트)
      │         또는
      └─ 수동 /compact 명령
            │
     [압축 모드 결정]
            │
    ┌───────┴───────────────────┐
    │ pp8() → SM Compact 여부   │
    │ (tengu_sm_compact 플래그) │
    └───────────────────────────┘
            │
    ┌───────┴────────────────────────────────────┐
    │                                            │
    ▼                                            ▼
 전체 압축                                  부분 압축
 aWK(userCompactInstructions)         gWK(N3Y, userCompactInstructions)
 → 대화 전체 요약                      → 최근 메시지만 요약
    │                                            │
    └──────────────┬─────────────────────────────┘
                   ▼
            Claude API 호출
            (도구 사용 금지 강제)
                   │
                   ▼
            <analysis> + <summary> 수신
                   │
                   ▼
              y3Y() 후처리
                   │
                   ▼
         P18() → 다음 세션 시스템 프롬프트
                   │
         ┌─────────┴──────────────┐
         │ autoResume=true        │ autoResume=false
         ▼                        ▼
   "Pick up the last         요약만 표시
    task as if the            (사용자 확인 후 재개)
    break never happened."
```

---

## 7. 컨텍스트 인계 관련 환경변수 & 피처 플래그

| 변수/플래그 | 의미 |
|------------|------|
| `ENABLE_CLAUDE_CODE_SM_COMPACT` | 세션 메모리 부분 압축 강제 활성화 |
| `DISABLE_CLAUDE_CODE_SM_COMPACT` | 세션 메모리 부분 압축 강제 비활성화 |
| `tengu_sm_compact` | SM Compact GrowthBook 피처 플래그 |
| `tengu_session_memory` | 세션 메모리 전체 기능 게이팅 |
| `tengu_sm_compact_config` | `{minTokens, maxTokens, minTextBlockMessages}` 설정 |
| `tengu_auto_compact_succeeded` | 자동 압축 성공 텔레메트리 이벤트 |
| `tengu_compact` | 수동 compact 텔레메트리 이벤트 |
| `tengu_partial_compact` | 부분 압축 텔레메트리 이벤트 |

---

## 8. Auto-Dream — 메모리 통합 시스템 (전문)

> **함수:** `WWK()` (Dream 프롬프트 빌더), 텔레메트리: `tengu_auto_dream_*`  
> **위치:** `cli.js:9429719` (WWK), `cli.js:11350053` (상수)  
> **목적:** 유휴 시간에 백그라운드에서 자동으로 메모리 파일을 정리·통합하는 반사적(reflective) 패스

### 8-1. Dream 트리거 조건

| 항목 | 값 |
|------|-----|
| 최소 경과 시간 | `minHours: 24` (24시간 이상) |
| 최소 세션 수 | `minSessions: 5` (5세션 이상 누적) |
| 잠금 방식 | `.consolidate-lock` 파일 (PID 기반, `t8Y=3600000ms` 타임아웃) |
| 실행 방식 | `querySource:"auto_dream"` — 포크 에이전트로 격리 실행 |

### 8-2. Dream 시스템 프롬프트 전문 (`WWK()` 생성)

```
# Dream: Memory Consolidation

You are performing a dream — a reflective pass over your memory files. Synthesize
what you've learned recently into durable, well-organized memories so that future
sessions can orient quickly.

Memory directory: `{memoryDir}`
{D87: "This directory already exists — write to it directly with the Write tool
       (do not run mkdir or check for its existence)."}

Session transcripts: `{transcriptDir}` (large JSONL files — grep narrowly,
                                          don't read whole files)

---

## Phase 1 — Orient

- `ls` the memory directory to see what already exists
- Read `MEMORY.md` to understand the current index
- Skim existing topic files so you improve them rather than creating duplicates
- If `logs/` or `sessions/` subdirectories exist (assistant-mode layout),
  review recent entries there

## Phase 2 — Gather recent signal

Look for new information worth persisting. Sources in rough priority order:

1. **Daily logs** (`logs/YYYY/MM/YYYY-MM-DD.md`) if present
   — these are the append-only stream
2. **Existing memories that drifted** — facts that contradict something you
   see in the codebase now
3. **Transcript search** — if you need specific context (e.g., "what was the
   error message from yesterday's build failure?"), grep the JSONL transcripts
   for narrow terms:
   `grep -rn "<narrow term>" {transcriptDir}/ --include="*.jsonl" | tail -50`

Don't exhaustively read transcripts. Look only for things you already suspect matter.

## Phase 3 — Consolidate

For each thing worth remembering, write or update a memory file at the top level
of the memory directory. Use the memory file format and type conventions from
your system prompt's auto-memory section — it's the source of truth for what to
save, how to structure it, and what NOT to save.

Focus on:
- Merging new signal into existing topic files rather than creating near-duplicates
- Converting relative dates ("yesterday", "last week") to absolute dates so they
  remain interpretable after time passes
- Deleting contradicted facts — if today's investigation disproves an old memory,
  fix it at the source

## Phase 4 — Prune and index

Update `MEMORY.md` so it stays under 200 lines AND under ~25KB. It's an **index**,
not a dump — each entry should be one line under ~150 characters:
`- [Title](file.md) — one-line hook`. Never write memory content directly into it.

- Remove pointers to memories that are now stale, wrong, or superseded
- Demote verbose entries: if an index line is over ~200 chars, it's carrying
  content that belongs in the topic file — shorten the line, move the detail
- Add pointers to newly important memories
- Resolve contradictions — if two files disagree, fix the wrong one

---

Return a brief summary of what you consolidated, updated, or pruned.
If nothing changed (memories are already tight), say so.

[선택적 추가 컨텍스트 블록]:
## Additional context
{additionalContext}
```

### 8-3. Dream 실행 제약사항

```
**Tool constraints for this run:**
Bash is restricted to read-only commands (`ls`, `find`, `grep`, `cat`, `stat`,
`wc`, `head`, `tail`, and similar).
Anything that writes, redirects to a file, or modifies state will be denied.
Plan your exploration with this in mind — no need to probe.

Sessions since last consolidation (N):
- {sessionId_1}
- {sessionId_2}
- ...
```

### 8-4. Dream 관련 상수 & 설정값

| 상수/변수 | 값 | 의미 |
|-----------|-----|------|
| `oP` | `"MEMORY.md"` | 메모리 인덱스 파일명 |
| `nK6` | `200` | MEMORY.md 최대 줄 수 |
| `gq8` | `25000` | MEMORY.md 최대 바이트 (25KB) |
| `fWK.minHours` | `24` | Dream 최소 간격 (시간) |
| `fWK.minSessions` | `5` | Dream 최소 누적 세션 수 |
| `t8Y` | `3600000ms` | 잠금 파일 타임아웃 (1시간) |
| `s8Y` | `".consolidate-lock"` | 잠금 파일명 |
| `$hY` | `"auto memory"` | 기능 표시명 |
| `K3Y` | `600000ms` | Dream 에이전트 타임아웃 (10분) |

### 8-5. Dream 텔레메트리 이벤트

| 이벤트 | 페이로드 | 의미 |
|--------|---------|------|
| `tengu_auto_dream_fired` | `{hours_since, sessions_since}` | Dream 실행 시작 |
| `tengu_auto_dream_completed` | `{cache_read, cache_created, output, sessions_reviewed}` | 성공 완료 |
| `tengu_auto_dream_failed` | `{}` | 포크 에이전트 실패 |
| `tengu_auto_dream_toggled` | `{enabled}` | 사용자 ON/OFF 전환 |

### 8-6. autoDreamEnabled 설정

```
// settings.json 스키마 (Zod):
autoDreamEnabled: z.boolean().optional()
  .describe("Enable background memory consolidation (auto-dream).
             When set, overrides the server-side default.")
```

### 8-7. Dream 전체 흐름

```
[유휴 감지 / 세션 종료]
        │
        ▼
  경과 시간 >= 24h AND 세션 수 >= 5?
        │
        ▼ YES
  .consolidate-lock 획득 시도
        │
        ▼ 성공
  세션 transcripts 목록 수집
  (mtime > lastDreamMtime 필터링)
        │
        ▼
  WWK() → Dream 시스템 프롬프트 생성
        │
        ▼
  hG() API 호출
  querySource="auto_dream"
  skipTranscript=true
  Bash = 읽기 전용 제한
        │
        ├── Phase 1: Orient (ls + MEMORY.md 읽기)
        ├── Phase 2: Gather (로그/트랜스크립트 grep)
        ├── Phase 3: Consolidate (메모리 파일 업데이트)
        └── Phase 4: Prune (MEMORY.md 정리, 200줄/25KB 제한)
        │
        ▼
  완료 → 잠금 해제 + tengu_auto_dream_completed
  실패 → 잠금 롤백(Om8) + tengu_auto_dream_failed
        │
        ▼
  filesTouched가 있으면 "Improved" verb로 UI 표시
```

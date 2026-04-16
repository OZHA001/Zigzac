# ZIGZAC — 최고 수준의 AI 에이전트 통합 프레임워크

> Claude Code 역분석 자산(Brain · Control · Handover · DNA · Flags) + **hermes** + **pi-mono/skills** + **openclaw** 를 7개 레이어로 통합한 자율 AI 에이전트 플랫폼

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 목차

- [프로젝트 개요](#프로젝트-개요)
- [핵심 특징](#핵심-특징)
- [아키텍처 한눈에 보기](#아키텍처-한눈에-보기)
- [통합 소스 구성](#통합-소스-구성)
- [빠른 시작](#빠른-시작)
- [디렉토리 구조](#디렉토리-구조)
- [로드맵](#로드맵)
- [문서](#문서)

---

## 프로젝트 개요

ZIGZAC는 Anthropic Claude Code CLI(`cli.js v2.1.88`)를 직접 역분석하여 확보한 **비공개 내부 자산**을 기반으로, 세 개의 오픈소스 에이전트 프레임워크를 통합한 고성능 AI 에이전트 플랫폼입니다.

| 확보 자산 | 내용 |
|----------|------|
| `Brain.md` | 11개 모델 ID × 4 프로바이더, 27개 베타 헤더 전체 |
| `control.md` | 브라우저 제어 18종 + macOS 제어 24종 = **42개 실행 도구** |
| `GEMINI_TOOLS_MANIFEST.json` | Agent/Bash/FileEdit/WebFetch 등 **21개 SDK 도구 스키마** |
| `agent_dna_schema.json` | 전역 에이전트 상태 **89개 필드** 완전 명세 |
| `Handover.md` | 컨텍스트 압축·인계 프롬프트 전문 (무한 실행 루프 기반) |
| `other.json` | tengu/KAIROS/UltraPlan/ultrathink 등 **420개 피처 플래그** |

---

## 핵심 특징

- 🧠 **스마트 모델 라우팅** — 작업 복잡도에 따라 Haiku/Sonnet/Opus 자동 선택 + 27개 베타 헤더 동적 주입
- ♾️ **무한 컨텍스트** — Handover 인계 시스템으로 토큰 한계 없이 장기 작업 지속
- 🔌 **플러그인 도구 허브** — openclaw 패턴으로 누구나 새 도구를 `.tool.ts` 파일 하나로 기여
- 📦 **Skills 레지스트리** — pi-skills 스키마 기반으로 능력 단위를 버전 관리
- 🔄 **자기 진화** — hermes self-evolution 루프로 반복 실패 작업에서 새 도구를 스스로 생성
- 🤖 **자율 오케스트레이션** — KAIROS/UltraPlan 플래그 기반 크론 스케줄 + AFK 무감독 실행

---

## 아키텍처 한눈에 보기

```
┌─────────────────────────────────────────────────────────────┐
│                      ZIGZAC AGENT                           │
│                                                             │
│  [L7] KAIROS / UltraPlan 자율 오케스트레이터                 │
│       └─ 크론 스케줄 · AFK 모드 · BYOC 원격 실행            │
│  [L6] Handover 메모리 지속성                                 │
│       └─ 컨텍스트 압축 · continuation 인계 · 1M 창          │
│  [L5] hermes Self-Evolution 루프                             │
│       └─ 실패 감지 → 도구 자동 생성 → 핫 로드               │
│  [L4] pi-skills Skills 레지스트리                            │
│       └─ 버전 관리 능력 단위 · 의존성 그래프                 │
│  [L3] openclaw Tool Hub                                      │
│       └─ 42개 제어 도구 + 21개 SDK 도구 플러그인             │
│  [L2] hermes Function-Calling 프로토콜                       │
│       └─ 구조화 도구 호출 · 병렬 실행 · JSON 스키마 강제     │
│  [L1] 모델 라우터 (Brain.md)                                 │
│       └─ 11모델 × 4 프로바이더 · 베타 헤더 주입              │
└─────────────────────────────────────────────────────────────┘
```

상세 아키텍처는 [ARCHITECTURE.md](./ARCHITECTURE.md)를 참조하세요.

---

## 통합 소스 구성

| 소스 | 역할 | 적용 레이어 |
|------|------|------------|
| **ZIGZAC** (역분석 자산) | 모델 ID · 베타 헤더 · 도구 스키마 · 상태 기계 · 메모리 | L1, L3, L6, L7 |
| **hermes-agent** (NousResearch) | XML 기반 Function-Calling 프로토콜 | L2 |
| **hermes-agent-self-evolution** | 에이전트 자기 진화 루프 | L5 |
| **pi-mono / pi-skills** (badlogic) | 모노레포 구조 · Skills 스키마 | L4 |
| **openclaw / clawhub** | 플러그인 도구 허브 아키텍처 | L3 |

---

## 빠른 시작

```bash
# 1. 저장소 클론
git clone https://github.com/OZHA001/Zigzac.git
cd Zigzac

# 2. 의존성 설치 (Phase 1 이후)
npm install

# 3. 환경변수 설정
cp .env.example .env
# ANTHROPIC_API_KEY=sk-ant-...

# 4. 에이전트 실행
npm run agent
```

> ⚠️ 현재 Phase 1 구현 진행 중입니다. 위 명령은 Phase 1 완료 후 사용 가능합니다.

---

## 디렉토리 구조

```
Zigzac/
├── docs/
│   ├── ZIGZAC/            # 역분석 자산 (Brain · Control · Handover · DNA · Flags)
│   └── ref/               # 통합 대상 외부 저장소 참조
├── src/                   # (Phase 1~) 소스 코드
│   ├── core/              # L1 모델 라우터 + L2 프로토콜
│   ├── tools/             # L3 Tool Hub 플러그인
│   ├── skills/            # L4 Skills 레지스트리
│   ├── memory/            # L6 Handover 메모리 시스템
│   └── orchestrator/      # L7 자율 오케스트레이터
├── README.md
└── ARCHITECTURE.md
```

---

## 로드맵

| Phase | 범위 | 상태 |
|-------|------|------|
| **Phase 1** | 모델 라우터 + 베타 헤더 엔진 + Hermes 도구 호출 파서 | 🚧 진행 예정 |
| **Phase 2** | 42 + 21개 도구 openclaw 플러그인 등록 | ⏳ 대기 |
| **Phase 3** | pi-skills 기반 Skills 레지스트리 | ⏳ 대기 |
| **Phase 4** | Handover 컨텍스트 압축·인계 시스템 | ⏳ 대기 |
| **Phase 5** | Self-Evolution + KAIROS 자율 오케스트레이터 | ⏳ 대기 |

---

## 문서

- [ARCHITECTURE.md](./ARCHITECTURE.md) — 7레이어 상세 설계, 통합 방법, 기술 스택
- [docs/ZIGZAC/Brain.md](./docs/ZIGZAC/Brain.md) — 모델 레지스트리 & 베타 헤더 전체
- [docs/ZIGZAC/control.md](./docs/ZIGZAC/control.md) — 42개 에이전트 제어 도구
- [docs/ZIGZAC/Handover.md](./docs/ZIGZAC/Handover.md) — 컨텍스트 인계 시스템
- [docs/ZIGZAC/agent_dna_schema.json](./docs/ZIGZAC/agent_dna_schema.json) — 에이전트 상태 스키마
- [docs/ZIGZAC/GEMINI_TOOLS_MANIFEST.json](./docs/ZIGZAC/GEMINI_TOOLS_MANIFEST.json) — SDK 도구 매니페스트
- [docs/ZIGZAC/other.json](./docs/ZIGZAC/other.json) — 피처 플래그 전체
# Claude Code × Codex 크로스 에이전트 워크플로우

| 항목 | 내용 |
|------|------|
| 생성일 | 2026-04-03 |
| 변경일 | 2026-04-03 |
| 목적 | 워크플로우 설계 고민 — Claude Code와 Codex 간 연계 패턴 정리 |
| 참조 | [Toby님의 AI 개발 루틴](https://www.notion.so/325841d618e181468060c17e7d3a746c) |

---

## 목차

1. [배경 — Toby님 워크플로우 분석](#1-배경--toby님-워크플로우-분석)
2. [핵심 질문](#2-핵심-질문)
3. [연계 패턴 4가지](#3-연계-패턴-4가지)
   - 3.1 셸 파이프 오케스트레이션
   - 3.2 루프 스크립트 (다중 라운드)
   - 3.3 Claude Code Skill로 Codex 호출
   - 3.4 MCP 서버 브릿지
4. [현실적 제약 사항](#4-현실적-제약-사항)
5. [추천 접근 — 단계별 도입](#5-추천-접근--단계별-도입)
6. [판단 기준 — Codex를 추가할 가치가 있는가?](#6-판단-기준--codex를-추가할-가치가-있는가)

---

## 1. 배경 — Toby님 워크플로우 분석

Toby(이일민)님의 AI 개발 루틴 핵심 흐름:

```
프로젝트 선정 → Claude(웹)로 PRD 작성 → Codex로 PRD 검토
    → Claude Code로 개선 → /prd 고도화 → /ralph로 prd.json 생성 → Ralph Loop 자동 개발
```

### 각 단계 역할

| 단계 | 도구 | 역할 |
|------|------|------|
| 1단계 | 사람 | 오픈소스 레퍼런스 + 내 프로젝트 선정 |
| 2단계 | Claude (웹) | 프로젝트 분석 → PRD 초안 작성 |
| 3단계 | Codex → Claude Code | PRD 교차 검토 → 개선 |
| 4단계 | Claude Code (`/prd`) | 불명확한 부분 질의 → PRD 고도화 |
| 5단계 | Claude Code (`/ralph`) | User Story를 task 단위로 분해 → `prd.json` 생성 |
| 6단계 | Ralph Loop | `prd.json` 기반 자동 구현 루프 |

### 핵심 패턴: AI 오케스트레이션

한 AI의 **출력**을 다른 AI의 **입력**으로 넘기는 구조.
목적은 **second opinion** — 단일 AI의 편향을 다른 AI가 보완.

---

## 2. 핵심 질문

> Claude Code를 메인으로 Codex와 연계하여, 서로 대화를 주고받게 할 수 있는가?

**결론: 가능하다.** 두 도구 모두 CLI 헤드리스 모드(`-p` 플래그)를 지원하므로, 셸 스크립트로 상호 연결할 수 있다.

---

## 3. 연계 패턴 4가지

### 3.1 셸 파이프 오케스트레이션 (가장 단순)

가장 적은 코드로 시작할 수 있는 패턴.

```bash
# Step 1: Claude Code가 PRD 작성 → 파일 저장
claude -p "이 프로젝트를 분석해서 PRD를 작성해줘" > prd.md

# Step 2: Codex가 PRD 검토 → 피드백 저장
codex -p "$(cat prd.md) — 이 PRD를 검토하고 개선점을 제시해줘" > review.md

# Step 3: Claude Code가 피드백 반영 → 최종 PRD
claude -p "원본 PRD: $(cat prd.md) / 검토 의견: $(cat review.md) — 개선된 최종 PRD를 작성해줘" > prd-final.md
```

**장점**: 스크립트 3줄, 즉시 실행 가능
**단점**: 단방향 1회 — 대화가 아닌 릴레이

---

### 3.2 루프 스크립트 (다중 라운드 대화)

여러 라운드에 걸쳐 두 AI가 반복적으로 검토·개선하는 패턴.

```bash
#!/bin/bash
# cross-agent-loop.sh

ROUNDS=3
PROJECT_DIR=$(pwd)
WORK_DIR="$PROJECT_DIR/.ai-review"
mkdir -p "$WORK_DIR"

# 초기 입력
INPUT="이 프로젝트의 코드를 분석해서 PRD를 작성해줘"

for i in $(seq 1 $ROUNDS); do
  echo "=== Round $i / $ROUNDS ==="

  # Claude Code 턴
  claude -p "$INPUT" > "$WORK_DIR/round_${i}_claude.md"
  echo "  ✅ Claude Code 완료"

  # Codex 턴 — Claude 결과를 검토
  CLAUDE_OUTPUT=$(cat "$WORK_DIR/round_${i}_claude.md")
  codex -p "다음 내용을 검토하고 구체적인 개선점을 제시해줘:

$CLAUDE_OUTPUT" > "$WORK_DIR/round_${i}_codex.md"
  echo "  ✅ Codex 검토 완료"

  # 다음 라운드 입력 = Codex 피드백 반영 요청
  CODEX_FEEDBACK=$(cat "$WORK_DIR/round_${i}_codex.md")
  INPUT="이전 피드백을 모두 반영해서 PRD를 개선해줘:

$CODEX_FEEDBACK"
done

# 최종 결과
cp "$WORK_DIR/round_${ROUNDS}_claude.md" "$PROJECT_DIR/prd-final.md"
echo "✅ 최종 PRD: prd-final.md"
```

**장점**: 라운드 수 조절 가능, 점진적 개선
**단점**: 라운드마다 컨텍스트가 리셋됨 (이전 대화 누적 안 됨)

---

### 3.3 Claude Code Skill로 Codex 호출 (하네스 통합)

Claude Code 안에서 Codex를 **도구처럼** 사용하는 패턴.

#### Skill 정의 (`SKILL.md`)

```markdown
---
name: codex-review
description: Codex에게 현재 작업물 검토를 요청
---

다음 단계를 수행해:

1. 현재 작업 중인 파일 또는 PRD를 `/tmp/codex-input.md`로 저장
2. 아래 명령을 실행:
   ```bash
   codex -p "$(cat /tmp/codex-input.md) — 검토하고 개선점을 제시해줘" > /tmp/codex-review.md
   ```
3. `/tmp/codex-review.md`를 읽어서 핵심 피드백을 요약
4. 피드백을 반영하여 작업물을 개선
```

#### 사용법

```
claude> /codex-review
```

**장점**: Claude Code 세션 내에서 자연스럽게 호출, 컨텍스트 유지
**단점**: Skill 실행 시 외부 CLI 호출 의존

---

### 3.4 MCP 서버 브릿지 (완전 자동화)

가장 고급 패턴 — Codex를 MCP 서버로 래핑하면 Claude Code가 네이티브 도구처럼 호출 가능.

#### 아키텍처

```
┌─────────────┐     MCP Protocol      ┌─────────────────┐
│ Claude Code │ ◄──────────────────► │ Codex MCP Server │
│  (메인 AI)  │   tool call/response  │  (래핑된 Codex)  │
└─────────────┘                       └─────────────────┘
       │                                      │
       ▼                                      ▼
   프로젝트 파일                          Codex CLI (-p)
```

#### MCP 서버 구현 예시 (Node.js 스켈레톤)

```javascript
// codex-mcp-server/index.js
import { McpServer } from "@anthropic-ai/sdk/mcp";
import { execSync } from "child_process";

const server = new McpServer({ name: "codex-bridge" });

server.tool("codex_review", { prompt: "string" }, async ({ prompt }) => {
  const result = execSync(`codex -p "${prompt}"`, {
    encoding: "utf-8",
    timeout: 120000,
  });
  return { content: [{ type: "text", text: result }] };
});

server.start();
```

#### Claude Code 설정 (`settings.json`)

```json
{
  "mcpServers": {
    "codex-bridge": {
      "command": "node",
      "args": ["./codex-mcp-server/index.js"]
    }
  }
}
```

**장점**: Claude Code가 Codex를 네이티브 도구로 인식, 가장 깊은 통합
**단점**: MCP 서버 개발·유지 필요, 복잡도 높음

---

## 4. 현실적 제약 사항

| 항목 | 설명 | 영향도 |
|------|------|--------|
| **토큰 비용** | 두 AI 모두 API 과금 → 라운드마다 비용 2배 | 높음 |
| **컨텍스트 유실** | 파일 기반 전달 시 대화 맥락이 끊김 | 중간 |
| **Codex CLI 기능** | OpenAI Codex CLI는 Claude Code 대비 하네스 기능이 제한적 | 중간 |
| **디버깅 난이도** | AI 간 오해 발생 시 원인 추적 복잡 | 중간 |
| **속도** | 각 AI 호출마다 수십 초 대기 → 라운드 3회면 수 분 소요 | 낮음 |
| **출력 일관성** | 두 AI의 응답 형식이 다를 수 있음 → 파싱 필요 | 낮음 |

---

## 5. 추천 접근 — 단계별 도입

| 단계 | 패턴 | 대상 | 설명 |
|------|------|------|------|
| **입문** | 3.1 셸 파이프 | 개인 실험 | 스크립트 3줄로 시작, 효과 체감 |
| **실용** | CLAUDE.md 커스텀 명령 | 일상 개발 | Toby님처럼 `/prd`, `/ralph` 정의 → Claude Code 단독으로 충분 |
| **중급** | 3.2 루프 스크립트 | PRD 품질이 중요한 프로젝트 | 다중 라운드 교차 검토 |
| **고급** | 3.4 MCP 브릿지 | 팀 자동화 파이프라인 | 완전 자동화, CI/CD 연동 가능 |

---

## 6. 판단 기준 — Codex를 추가할 가치가 있는가?

### 추가하면 좋은 경우

- PRD 품질이 프로젝트 성패를 좌우할 때 (second opinion 필요)
- 두 AI의 **관점 차이**를 의도적으로 활용하고 싶을 때
- 특정 도메인에서 Codex가 Claude보다 강점이 있을 때

### 불필요한 경우

- 개인 프로젝트나 프로토타입 — Claude Code 단독 루프로 충분
- 비용 민감한 환경 — 라운드당 2배 과금
- 빠른 반복이 중요한 경우 — AI 간 핸드오프가 병목

### 대안: Claude Code 자체 교차 검토

```bash
# Claude Code에게 "비평가 역할"을 부여하여 자체 검토
claude -p "PRD를 작성해줘" > prd-v1.md
claude -p "너는 시니어 PM이야. 이 PRD의 허점을 찾아줘: $(cat prd-v1.md)" > critique.md
claude -p "비평을 반영해서 개선해줘: $(cat critique.md)" > prd-v2.md
```

> **핵심**: 크로스 에이전트의 본질은 "다른 관점"이다. 같은 AI라도 **역할(persona)을 바꾸면** 유사한 효과를 낼 수 있다.

---

## 참고 자료

- [Toby님의 AI 개발 루틴 — 초보자 가이드](https://www.notion.so/325841d618e181468060c17e7d3a746c)
- [Claude Code 하네스 추천구성](claude-code-harness-추천구성.md) — §3.8 멀티 프로젝트 패턴
- [Claude Code 초보자 튜토리얼](claude-code-초보자-튜토리얼.md) — Skills, Hooks 기초
- [Codex 공식 플러그인 완전 가이드](../guides/claude-code-codex-플러그인-완전-가이드.md) — 수동 CLI 대신 4줄 설치로 사용하는 공식 플러그인

# Claude Code 개인 설정 가이드 (2026)

| 항목 | 날짜 |
|------|------|
| 생성일 | 2026-03-11 |
| 변경일 | 2026-04-01 |

> AI 분야 저명한 개발자들의 GitHub, 블로그, 공식 문서를 기반으로 정리한 **개인용 Claude Code 최적 구성법**

### 관련 문서
- [Harness 추천 구성](claude-code-harness-추천구성.md) — 설정 적용 순서 전략
- [CLAUDE.md 실전 작성법](claude-code-CLAUDE-md-실전-작성법.md) — CLAUDE.md 심화 패턴
- [팀 IDE 통합 가이드](claude-code-팀-IDE-통합-가이드.md) — 개인 설정을 팀으로 확장

---

## 목차

1. [설치 및 기본 설정](#1-설치-및-기본-설정)
2. [CLAUDE.md 작성법](#2-claudemd-작성법)
3. [settings.json 권한 설정](#3-settingsjson-권한-설정)
4. [Hooks 자동화](#4-hooks-자동화)
5. [Skills 구성](#5-skills-구성)
6. [커스텀 Subagent](#6-커스텀-subagent)
7. [MCP 서버 연결](#7-mcp-서버-연결)
8. [커스텀 Slash Commands](#8-커스텀-slash-commands)
9. [메모리 및 컨텍스트 관리](#9-메모리-및-컨텍스트-관리)
10. [상태 표시줄(Status Line) 설정](#10-상태-표시줄status-line-설정)
11. [일상 워크플로우 패턴](#11-일상-워크플로우-패턴)
12. [흔한 실수와 해결법](#12-흔한-실수와-해결법)

---

## 1. 설치 및 기본 설정

```bash
# 네이티브 바이너리 설치 (권장, Node.js 불필요)
curl -fsSL https://claude.ai/install | sh

# 또는 npm 설치
npm install -g @anthropic-ai/claude-code
```

### 초기 프로젝트 세팅

```bash
cd your-project
claude          # REPL 모드 진입
/init           # 프로젝트 분석 후 CLAUDE.md 자동 생성
```

`/init` 명령은 빌드 시스템, 테스트 프레임워크, 코드 패턴을 자동 감지하여 CLAUDE.md 초안을 생성한다.

---

## 2. CLAUDE.md 작성법

CLAUDE.md는 **매 세션 시작 시 자동 로드**되는 핵심 설정 파일이다.

### 파일 위치별 적용 범위

| 위치 | 적용 범위 | 용도 |
|------|----------|------|
| `~/.claude/CLAUDE.md` | 모든 프로젝트 | 개인 글로벌 규칙 |
| `./CLAUDE.md` | 현재 프로젝트 | 팀 공용 (git 커밋) |
| `.claude/rules/*.md` | 현재 프로젝트 | 규칙 파일 분할 |
| `./sub-dir/CLAUDE.md` | 하위 디렉토리 | 모듈별 지침 |

### 핵심 원칙: **200줄 이하**로 유지

> "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!" — Anthropic 공식 문서

**각 줄마다 자문:** "이 줄을 삭제하면 Claude가 실수할까?" → 아니라면 삭제.

### 글로벌 CLAUDE.md 예시 (`~/.claude/CLAUDE.md`)

```markdown
# 개발 철학
- 간결하고 읽기 쉬운 코드를 우선시한다
- 과도한 추상화보다 명확한 구현을 선호한다

# 코드 스타일
- 한국어 주석 사용
- 타입 안전성을 항상 고려

# 워크플로우
- 코드 변경 후 반드시 타입 체크 실행
- 전체 테스트 스위트 대신 관련 단일 테스트 실행
- GitHub 작업에는 항상 `gh` CLI 사용

# 커밋
- 커밋 메시지는 한국어로 작성
- 컨벤셔널 커밋 형식 사용 (feat:, fix:, refactor: 등)
```

### 프로젝트 CLAUDE.md 예시 (`./CLAUDE.md`)

```markdown
# 프로젝트 개요
Spring Boot 3.x + Java 21 기반 REST API 서버

# 빌드 & 테스트
- 빌드: `./gradlew build`
- 테스트: `./gradlew test`
- 단일 테스트: `./gradlew test --tests "com.example.SomeTest"`

# 아키텍처
- Controller → Service → Repository 레이어드 구조
- Entity를 Controller에서 직접 반환 금지, DTO 사용 필수
- DI는 생성자 주입(@RequiredArgsConstructor) 필수

# 컨벤션
- REST 경로: kebab-case
- JSON 필드: camelCase
- 패키지 구조: @docs/architecture.md 참조
```

### 포함해야 할 것 vs 제외할 것

| 포함 | 제외 |
|------|------|
| Claude가 추론할 수 없는 빌드 명령어 | 코드만 봐도 알 수 있는 것 |
| 기본값과 다른 코드 스타일 규칙 | 표준 언어 컨벤션 |
| 테스트 방법 및 테스트 러너 | 상세 API 문서 (링크로 대체) |
| 프로젝트 고유 아키텍처 결정 | 자주 변하는 정보 |
| 환경 변수 등 개발 환경 quirks | 코드 스니펫 (금방 outdated됨) |

### `@` Import 문법으로 외부 파일 참조

```markdown
# CLAUDE.md
자세한 내용은 아래 참조:
- 프로젝트 개요: @README.md
- Git 워크플로우: @docs/git-instructions.md
- 개인 오버라이드: @~/.claude/my-overrides.md
```

---

## 3. settings.json 권한 설정

### 파일 위치

| 파일 | 용도 |
|------|------|
| `~/.claude/settings.json` | 글로벌 사용자 설정 |
| `.claude/settings.json` | 프로젝트 설정 (git 커밋 가능) |
| `.claude/settings.local.json` | 로컬 오버라이드 (gitignore) |

### 권장 글로벌 설정 (`~/.claude/settings.json`)

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Edit",
      "Write",
      "Bash(git *)",
      "Bash(gh *)",
      "Bash(npm *)",
      "Bash(npx *)",
      "Bash(gradle *)",
      "Bash(./gradlew *)",
      "Bash(ls *)",
      "Bash(cat *)",
      "Bash(mkdir *)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Write(./.env)",
      "Write(./.env.*)",
      "Bash(rm -rf *)"
    ]
  },
  "thinking": "always"
}
```

> **Tip:** `"thinking": "always"` 설정은 복잡한 작업에서 눈에 띄게 더 나은 결과를 만든다. — 여러 개발자 공통 추천

### 권한 문법

| 패턴 | 의미 |
|------|------|
| `Edit` | 모든 파일 편집 허용 |
| `Bash(git *)` | git으로 시작하는 모든 명령 허용 |
| `Bash(npm install)` | 특정 명령만 허용 |
| `Read(./.env)` | .env 파일 읽기 (deny에 사용) |
| `Write(src/*)` | src 폴더 내 쓰기만 허용 |

---

## 4. Hooks 자동화

Hooks는 CLAUDE.md 지침과 달리 **100% 결정적으로 실행**된다. 반드시 매번 실행되어야 하는 작업에 사용.

### Hook 이벤트 종류

| 이벤트 | 시점 |
|--------|------|
| `PreToolUse` | 도구 실행 전 |
| `PostToolUse` | 도구 실행 후 |
| `Notification` | 알림 발생 시 |
| `Stop` | Claude 응답 완료 시 |

### 설정 방법

**방법 1:** `/hooks` 명령으로 인터랙티브 설정

**방법 2:** `.claude/settings.json`에 직접 작성

### 실용 예시

#### 파일 저장 후 자동 포맷팅

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ]
  }
}
```

#### 민감 파일 쓰기 차단

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_FILE_PATHS\" | grep -qE '\\.(env|key|pem)$' && echo 'BLOCKED: 민감 파일 수정 차단' && exit 1 || exit 0"
          }
        ]
      }
    ]
  }
}
```

#### 파일 편집 후 ESLint/타입체크 자동 실행

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix \"$CLAUDE_FILE_PATHS\" 2>/dev/null; npx tsc --noEmit 2>&1 | head -20"
          }
        ]
      }
    ]
  }
}
```

---

## 5. Skills 구성

Skills는 **필요할 때만 로드**되는 도메인 지식/워크플로우 파일이다. CLAUDE.md에 넣기엔 특정 상황에만 필요한 내용에 적합.

### 디렉토리 구조

```
.claude/skills/
├── api-conventions/
│   └── SKILL.md
├── fix-issue/
│   └── SKILL.md
└── db-migration/
    └── SKILL.md
```

### Skills 작성 예시

#### 자동 호출 Skill (Claude가 필요 시 자동 적용)

```markdown
# .claude/skills/api-conventions/SKILL.md
---
name: api-conventions
description: REST API 설계 컨벤션
---
# API 설계 규칙
- URL 경로: kebab-case 사용
- JSON 속성: camelCase 사용
- 목록 API에는 반드시 페이지네이션 포함
- API 버전은 URL 경로에 포함 (/v1/, /v2/)
- 에러 응답 형식: { "error": { "code": "...", "message": "..." } }
```

#### 수동 호출 Skill (`/fix-issue 1234`로 실행)

```markdown
# .claude/skills/fix-issue/SKILL.md
---
name: fix-issue
description: GitHub 이슈 분석 및 수정
disable-model-invocation: true
---
GitHub 이슈를 분석하고 수정: $ARGUMENTS

1. `gh issue view $ARGUMENTS`로 이슈 상세 확인
2. 코드베이스에서 관련 파일 검색
3. 수정 구현
4. 테스트 작성 및 실행
5. 린트 및 타입 체크 통과 확인
6. 서술적 커밋 메시지 작성
7. PR 생성
```

### 글로벌 Skills (`~/.claude/skills/`)

모든 프로젝트에서 사용할 개인 Skill을 글로벌 위치에 둘 수 있다.

```markdown
# ~/.claude/skills/code-review/SKILL.md
---
name: code-review
description: 코드 리뷰 체크리스트
---
코드 리뷰 시 확인 사항:
1. OWASP Top 10 보안 취약점
2. N+1 쿼리 문제
3. 에러 핸들링 누락
4. 테스트 커버리지
5. 네이밍 컨벤션 준수
```

---

## 6. 커스텀 Subagent

Subagent는 **별도 컨텍스트 윈도우**에서 실행되어 메인 대화의 컨텍스트를 오염시키지 않는다.

### 디렉토리: `.claude/agents/`

```markdown
# .claude/agents/security-reviewer.md
---
name: security-reviewer
description: 보안 취약점 코드 리뷰
tools: Read, Grep, Glob, Bash
model: opus
---
시니어 보안 엔지니어로서 코드를 리뷰하세요:
- 인젝션 취약점 (SQL, XSS, 커맨드 인젝션)
- 인증/인가 결함
- 코드 내 시크릿/자격증명
- 안전하지 않은 데이터 처리

구체적 라인 참조와 수정 제안을 포함하세요.
```

```markdown
# .claude/agents/debugger.md
---
name: debugger
description: 버그 추적 및 디버깅 전문
tools: Read, Grep, Glob, Bash
model: sonnet
---
디버거로서 버그를 체계적으로 추적하세요:
1. 에러 메시지와 스택 트레이스 분석
2. 관련 코드 경로 추적
3. 재현 조건 파악
4. 근본 원인 식별
5. 수정 방안 제시
```

### 사용법

```
보안 이슈를 서브에이전트로 리뷰해줘
```

---

## 7. MCP 서버 연결

MCP(Model Context Protocol)로 외부 도구와 데이터 소스를 연결한다.

### 설정 위치

| 위치 | 범위 |
|------|------|
| `~/.claude.json` (user scope) | 모든 프로젝트에서 사용 |
| `.mcp.json` (프로젝트 루트) | 해당 프로젝트에서만 사용 |

### MCP 서버 추가 방법

```bash
# CLI로 추가
claude mcp add github-mcp -- npx -y @anthropic-ai/github-mcp-server

# 또는 .mcp.json 직접 편집
```

### 개인 개발자에게 추천하는 MCP 서버

| MCP 서버 | 용도 |
|----------|------|
| **GitHub** | PR, 이슈, 코드 리뷰 |
| **Playwright** | 브라우저 자동화, UI 테스트 |
| **Sentry** | 에러 모니터링, 디버깅 |
| **PostgreSQL/Supabase** | DB 쿼리, 스키마 탐색 |
| **Context7** | 라이브러리 최신 문서 조회 |
| **Figma** | 디자인 → 코드 변환 |

### .mcp.json 예시

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/github-mcp-server"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@context7/mcp-server"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/postgres-mcp-server"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

> **Tip:** 1~2개의 자주 쓰는 서버부터 시작하라. 너무 많으면 도구 선택에 혼란을 줄 수 있다.

---

## 8. 커스텀 Slash Commands

### 디렉토리

| 위치 | 범위 |
|------|------|
| `~/.claude/commands/` | 모든 프로젝트 (개인용) |
| `.claude/commands/` | 현재 프로젝트 |

### 예시

```markdown
# ~/.claude/commands/review.md
현재 브랜치의 변경사항을 리뷰하세요:
1. `git diff main...HEAD`로 전체 변경 확인
2. 보안 취약점 확인
3. 성능 이슈 확인
4. 테스트 커버리지 확인
5. 개선 사항 요약
```

```markdown
# ~/.claude/commands/pr.md
PR을 생성하세요:
1. 변경사항 분석
2. 서술적 PR 제목 작성 (70자 이내)
3. 변경 사항 요약을 PR 본문에 포함
4. `gh pr create`로 PR 생성
```

사용: `/review`, `/pr`

---

## 9. 메모리 및 컨텍스트 관리

### Auto Memory

Claude가 세션 중 학습한 내용을 자동으로 저장하는 기능 (기본 활성화).

- 빌드 명령어, 디버깅 인사이트, 아키텍처 노트 등을 자동 기록
- `/memory` 명령으로 토글 가능
- 저장 위치: `~/.claude/projects/<project-path>/memory/`

### 컨텍스트 관리 전략

컨텍스트 관리는 Claude Code 사용에서 **가장 중요한 요소**다.

| 전략 | 설명 |
|------|------|
| `/clear` | 관련 없는 작업 사이에 컨텍스트 초기화 |
| `/compact <지침>` | 컨텍스트 요약 (예: `/compact API 변경 내용에 집중`) |
| `/btw` | 사이드 질문 (컨텍스트에 남지 않음) |
| `/rewind` 또는 `Esc+Esc` | 체크포인트로 되돌리기 |
| 서브에이전트 사용 | 탐색 작업을 별도 컨텍스트에서 수행 |

### 세션 이어가기

```bash
claude --continue    # 마지막 대화 이어하기
claude --resume      # 최근 세션 목록에서 선택
```

`/rename`으로 세션에 이름 부여 (예: `oauth-migration`).

---

## 10. 상태 표시줄(Status Line) 설정

컨텍스트 사용량을 실시간으로 모니터링하는 커스텀 상태 표시줄 설정.

### 설정 파일: `~/.claude/statusline.sh`

```bash
#!/bin/bash
# 저장소 이름 + 컨텍스트 사용률 표시
REPO=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "no-repo")
CTX_PCT=${CLAUDE_CONTEXT_PERCENT:-0}

if [ "$CTX_PCT" -lt 40 ]; then
  COLOR="green"
elif [ "$CTX_PCT" -lt 60 ]; then
  COLOR="yellow"
else
  COLOR="red"
fi

echo "${REPO} | ctx: ${CTX_PCT}%"
```

> Freek Van der Herten 등 많은 개발자가 컨텍스트 사용률 모니터링을 **필수 설정**으로 추천한다.

---

## 11. 일상 워크플로우 패턴

### 추천 작업 흐름 (Anthropic 공식)

```
1. Explore (Plan Mode) → 코드 읽기, 구조 파악
2. Plan (Plan Mode)    → 구현 계획 수립
3. Implement           → 코드 작성 + 검증
4. Commit              → 커밋 및 PR 생성
```

### 효과적인 프롬프트 패턴

```
# 나쁜 예
"로그인 버그 고쳐줘"

# 좋은 예
"세션 타임아웃 후 로그인 실패 현상이 있어.
src/auth/의 토큰 갱신 로직을 확인해줘.
실패를 재현하는 테스트를 먼저 작성하고, 고친 후 테스트 통과를 확인해."
```

### Writer/Reviewer 패턴 (멀티 세션)

| 세션 A (Writer) | 세션 B (Reviewer) |
|-----------------|-------------------|
| `API rate limiter 구현해줘` | |
| | `src/middleware/rateLimiter.ts 리뷰해줘. 엣지 케이스, 레이스 컨디션 확인` |
| `리뷰 피드백 반영해줘: [세션 B 결과]` | |

### 대규모 마이그레이션 (Fan-out)

```bash
# 파일 목록 생성 후 병렬 처리
for file in $(cat files.txt); do
  claude -p "$file을 React에서 Vue로 마이그레이션해줘" \
    --allowedTools "Edit,Bash(git commit *)" &
done
```

---

## 12. 흔한 실수와 해결법

| 실수 | 증상 | 해결 |
|------|------|------|
| **Kitchen Sink 세션** | 관련 없는 작업을 같은 세션에서 처리 | `/clear`로 작업 간 컨텍스트 초기화 |
| **반복 교정** | 2번 이상 같은 수정 요청 | `/clear` 후 더 나은 초기 프롬프트로 재시작 |
| **비대한 CLAUDE.md** | 지시를 무시하기 시작 | 200줄 이하로 정리. Skills로 분리 |
| **검증 없는 신뢰** | 그럴듯하지만 동작하지 않는 코드 | 테스트/린터/스크린샷으로 항상 검증 |
| **무한 탐색** | 범위 없는 조사로 컨텍스트 소진 | 서브에이전트로 탐색 위임 |

---

## 전체 디렉토리 구조 요약

```
~/.claude/
├── CLAUDE.md                  # 글로벌 지침
├── settings.json              # 글로벌 권한/설정
├── commands/                  # 글로벌 커스텀 슬래시 명령
│   ├── review.md
│   └── pr.md
├── skills/                    # 글로벌 스킬
│   └── code-review/
│       └── SKILL.md
└── statusline.sh              # 상태 표시줄 스크립트

your-project/
├── CLAUDE.md                  # 프로젝트 지침 (git 커밋)
├── .mcp.json                  # MCP 서버 설정
└── .claude/
    ├── settings.json          # 프로젝트 권한 (git 커밋)
    ├── settings.local.json    # 로컬 오버라이드 (gitignore)
    ├── commands/              # 프로젝트 슬래시 명령
    ├── skills/                # 프로젝트 스킬
    │   ├── api-conventions/
    │   │   └── SKILL.md
    │   └── fix-issue/
    │       └── SKILL.md
    ├── agents/                # 커스텀 서브에이전트
    │   ├── security-reviewer.md
    │   └── debugger.md
    └── hooks/                 # Hook 스크립트
```

---

## 직접 확인해보기

- [ ] `claude --version`으로 설치 확인
- [ ] `claude`로 REPL 진입 후 `/init`으로 CLAUDE.md 생성
- [ ] `~/.claude/CLAUDE.md`에 글로벌 지침 작성 (언어, 코딩 스타일)
- [ ] `settings.json`에서 `deny` 패턴 1개 이상 설정 (예: `Read(.env)`)
- [ ] Hook 1개 설정 후 동작 확인 (예: PreToolUse에서 Bash 명령 로깅)
- [ ] `/clear` → 새 프롬프트로 깨끗한 세션 시작 체험

---

## Sources

- [Best Practices for Claude Code - 공식 문서](https://code.claude.com/docs/en/best-practices)
- [Claude Code Hooks Reference](https://code.claude.com/docs/en/hooks)
- [Claude Code Memory](https://code.claude.com/docs/en/memory)
- [Claude Code MCP 연결](https://code.claude.com/docs/en/mcp)
- [Claude Code Permissions](https://code.claude.com/docs/en/permissions)
- [Claude Code Skills](https://code.claude.com/docs/en/skills)
- [Freek Van der Herten - My Claude Code Setup](https://freek.dev/3026-my-claude-code-setup)
- [Okhlopkov - Claude Code Setup Guide 2026](https://okhlopkov.com/claude-code-setup-mcp-hooks-skills-2026/)
- [Trail of Bits - claude-code-config](https://github.com/trailofbits/claude-code-config)
- [shanraisshan - claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)
- [Cranot - claude-code-guide](https://github.com/Cranot/claude-code-guide)
- [zebbern - claude-code-guide](https://github.com/zebbern/claude-code-guide)
- [Builder.io - How I use Claude Code](https://www.builder.io/blog/claude-code)
- [HumanLayer - Writing a Good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [Shipyard - Claude Code CLI Cheatsheet](https://shipyard.build/blog/claude-code-cheat-sheet/)
- [Nick Babich - CLAUDE.md Best Practices (UX Planet)](https://uxplanet.org/claude-md-best-practices-1ef4f861ce7c)
- [ChrisWiles - claude-code-showcase](https://github.com/ChrisWiles/claude-code-showcase)
- [feiskyer - claude-code-settings](https://github.com/feiskyer/claude-code-settings)

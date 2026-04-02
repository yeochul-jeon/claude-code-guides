# Claude Code 팀 공용 · IDE · 내부 앱 연결 가이드 (2026)

| 항목 | 날짜 |
|------|------|
| 생성일 | 2026-03-11 |
| 변경일 | 2026-04-02 |

> 개인 설정을 넘어, **팀 공유 설정, IDE 통합, CI/CD 파이프라인, Agent Teams, Plugin 생태계, SDK 연동**까지 다루는 확장 가이드

### 관련 문서
- [개인 설정 가이드](claude-code-개인설정-가이드.md) — 개인 설정 기초 (이 문서의 선행 학습)
- [Harness 추천 구성](claude-code-harness-추천구성.md) — Phase 4~5에서 팀 설정 도입 전략
- [CLAUDE.md 실전 작성법](claude-code-CLAUDE-md-실전-작성법.md) — 팀 CLAUDE.md 작성 패턴

---

## 목차

1. [팀 공용 설정 체계](#1-팀-공용-설정-체계)
2. [엔터프라이즈 관리 설정 (managed-settings.json)](#2-엔터프라이즈-관리-설정)
3. [IDE 통합](#3-ide-통합)
4. [GitHub Actions CI/CD 연동](#4-github-actions-cicd-연동)
5. [Agent Teams (멀티 에이전트 협업)](#5-agent-teams-멀티-에이전트-협업)
6. [Plugin 생태계](#6-plugin-생태계)
7. [코드 인텔리전스 (LSP)](#7-코드-인텔리전스-lsp)
8. [SDK 및 프로그래밍 방식 연동](#8-sdk-및-프로그래밍-방식-연동)
9. [팀 마켓플레이스 구축](#9-팀-마켓플레이스-구축)
10. [보안 및 거버넌스](#10-보안-및-거버넌스)

---

## 1. 팀 공용 설정 체계

### 설정 파일 우선순위 (높음 → 낮음)

```
1. Managed Settings (최상위, 오버라이드 불가)
   ├── 서버 관리 설정 (Claude.ai admin console)
   ├── MDM/OS 정책 (Jamf, Intune, Group Policy)
   └── managed-settings.json 파일

2. CLI 인자 (세션 임시 오버라이드)

3. Local 프로젝트 설정 (.claude/settings.local.json)

4. 공유 프로젝트 설정 (.claude/settings.json)  ← git 커밋

5. 사용자 설정 (~/.claude/settings.json)
```

> **핵심 규칙:** 상위 스코프가 항상 우선. 사용자 설정에서 `allow`해도 프로젝트 설정에서 `deny`하면 차단됨.

### 팀 공유 프로젝트 설정 (`.claude/settings.json`)

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Bash(./gradlew *)",
      "Read(./src/**)",
      "Edit(./src/**)"
    ],
    "deny": [
      "Bash(curl *)",
      "Bash(wget *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
```

### 개인 로컬 오버라이드 (`.claude/settings.local.json`)

gitignore되며, 개인만 적용되는 설정.

```json
{
  "permissions": {
    "allow": [
      "Read(./docs/**)"
    ]
  },
  "model": "claude-opus-4-6",
  "thinking": "always"
}
```

### 팀 CLAUDE.md 전략

```
프로젝트/
├── CLAUDE.md                    # 팀 공용 (git 커밋)
├── frontend/
│   └── CLAUDE.md                # 프론트엔드 전용 규칙
├── backend/
│   └── CLAUDE.md                # 백엔드 전용 규칙
└── .claude/
    ├── settings.json            # 팀 공용 권한
    ├── settings.local.json      # 개인 오버라이드 (gitignore)
    └── skills/
        └── deploy/SKILL.md      # 팀 공용 스킬
```

팀 CLAUDE.md에는 **모든 팀원에게 공통**인 내용만 포함:
- 빌드/테스트 명령어
- 브랜치 네이밍, PR 컨벤션
- 아키텍처 결정 사항
- 코드 스타일 규칙

---

## 2. 엔터프라이즈 관리 설정

`managed-settings.json`은 IT/DevOps가 배포하며, **개별 사용자가 오버라이드할 수 없는** 정책.

### 배포 경로

| OS | 경로 |
|----|------|
| macOS | `/Library/Application Support/ClaudeCode/managed-settings.json` |
| Linux/WSL | `/etc/claude-code/managed-settings.json` |
| Windows | `C:\Program Files\ClaudeCode\managed-settings.json` |

### MDM 배포 (macOS)

Jamf, Kandji 등을 통해 `com.anthropic.claudecode` managed preferences 도메인으로 배포.

### 엔터프라이즈 managed-settings.json 예시

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(npm run *)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "WebFetch",
      "Read(~/.ssh/**)",
      "Read(~/.aws/**)",
      "Read(~/.kube/**)"
    ]
  },
  "allowManagedPermissionRulesOnly": true,
  "allowManagedHooksOnly": true,
  "allowManagedMcpServersOnly": true,
  "disableBypassPermissionsMode": true,
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverName": "git" }
  ],
  "deniedMcpServers": [
    { "serverName": "filesystem" }
  ],
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "our-org/approved-plugins",
      "ref": "v2.0"
    }
  ],
  "sandbox": {
    "enabled": true,
    "network": {
      "allowedDomains": [
        "github.company.com",
        "registry.npmjs.org"
      ]
    }
  },
  "companyAnnouncements": [
    "코드 리뷰는 모든 PR에 필수입니다",
    "보안 정책 가이드: docs.company.com/security"
  ],
  "forceLoginMethod": "console"
}
```

### Managed-Only 설정 키

| 설정 | 용도 |
|------|------|
| `allowManagedPermissionRulesOnly` | 사용자/프로젝트 권한 규칙 차단 |
| `allowManagedHooksOnly` | 관리자 Hook만 허용 |
| `allowManagedMcpServersOnly` | 허가된 MCP 서버만 사용 |
| `disableBypassPermissionsMode` | `--dangerously-skip-permissions` 차단 |
| `strictKnownMarketplaces` | 허가된 플러그인 마켓플레이스만 허용 |
| `network.allowManagedDomainsOnly` | 네트워크 도메인 제한 |

---

## 3. IDE 통합

### VS Code

VS Code 확장이 가장 완성도 높은 IDE 통합을 제공한다.

**설치:**
- VS Code Marketplace에서 "Claude Code" 검색 후 설치
- 확장에 CLI 바이너리가 포함되어 별도 설치 불필요

**주요 기능:**
- 네이티브 채팅 패널
- 인라인 편집 및 diff 리뷰
- 체크포인트 기반 undo
- `@` 파일 참조
- 병렬 대화 지원

**단축키:**
| 키 | 기능 |
|----|------|
| `Cmd+Esc` / `Ctrl+Esc` | Claude Code 패널 열기 |
| 선택 후 `Cmd+L` | 선택한 코드를 Claude에 전달 |

### JetBrains (IntelliJ, WebStorm 등)

**설치:**
- JetBrains Marketplace → "Claude Code **[Beta]**" 플러그인 설치
- > ⚠️ **Beta 기능**: 안정성이 보장되지 않으며, 향후 변경될 수 있습니다.
- IDE 재시작

**주요 기능:**
- 통합 터미널에서 Claude Code CLI 실행
- 코드 변경을 IDE의 diff 뷰어에 표시
- 현재 선택/탭을 자동으로 Claude에 공유

**단축키:**
| 키 | 기능 |
|----|------|
| `Cmd+Esc` (Mac) / `Ctrl+Esc` (Windows/Linux) | Claude Code 실행 |

### IDE vs 터미널 사용 가이드

| 작업 | 권장 환경 |
|------|----------|
| 빠른 인라인 편집 | IDE (VS Code/JetBrains) |
| 시각적 코드 탐색 | IDE |
| 대규모 리팩토링 | 터미널 Claude Code |
| 멀티 에이전트 작업 | 터미널 Claude Code |
| 외부 시스템 연동 (MCP) | 터미널 Claude Code |
| 자율 워크플로우 | 터미널 Claude Code |

---

## 4. GitHub Actions CI/CD 연동

### 빠른 설정

```bash
# Claude Code 터미널에서 실행
/install-github-app
```

이 명령이 GitHub App 설치와 시크릿 설정을 가이드한다.

### 수동 설정

1. [Claude GitHub App](https://github.com/apps/claude) 설치
2. `ANTHROPIC_API_KEY`를 리포지토리 시크릿에 추가
3. 워크플로우 파일 작성

### 기본 워크플로우 (`.github/workflows/claude.yml`)

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

jobs:
  claude:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### 자동 PR 코드 리뷰

```yaml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: >
            이 PR의 코드 품질, 정확성, 보안을 리뷰하세요.
            diff를 분석하고 리뷰 코멘트로 발견 사항을 게시하세요.
          claude_args: "--max-turns 5"
```

### 이슈 자동 구현

```yaml
name: Auto Implement
on:
  issues:
    types: [opened, assigned]

jobs:
  implement:
    if: contains(github.event.issue.labels.*.name, 'claude-auto')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: >
            이 이슈를 분석하고 구현하세요.
            완료 후 PR을 생성하세요.
          claude_args: "--model claude-opus-4-6 --max-turns 15"
```

### 일일 리포트 생성

```yaml
name: Daily Report
on:
  schedule:
    - cron: "0 9 * * *"

jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "어제의 커밋과 열린 이슈를 요약하세요"
```

### 클라우드 제공자별 설정

| 제공자 | 인증 방식 | 모델 ID 형식 |
|--------|----------|-------------|
| Anthropic API (직접) | `ANTHROPIC_API_KEY` | `claude-sonnet-4-6` |
| AWS Bedrock | OIDC + IAM Role | `us.anthropic.claude-sonnet-4-6` |
| Google Vertex AI | Workload Identity Federation | `claude-sonnet-4@20250514` |

### 비용 최적화 팁

- `--max-turns`로 최대 반복 횟수 제한
- 특정 `@claude` 명령으로 불필요한 API 호출 방지
- 워크플로우 수준 타임아웃 설정
- GitHub concurrency 제어로 병렬 실행 제한

---

## 5. Agent Teams (멀티 에이전트 협업)

> ⚠️ **실험적 기능** (2026.02 출시, `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` 필요). 프로덕션 환경에서는 충분한 테스트 후 사용을 권장합니다. 여러 Claude 인스턴스가 팀으로 협업한다.

### 활성화

```json
// settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 구조

| 구성요소 | 역할 |
|---------|------|
| **Team Lead** | 팀 생성, 작업 분배, 결과 종합 |
| **Teammates** | 독립 컨텍스트에서 작업 수행 |
| **Task List** | 공유 작업 목록 (claim/complete) |
| **Mailbox** | 에이전트 간 직접 메시징 |

### Subagent vs Agent Teams

| | Subagent | Agent Teams |
|-|----------|-------------|
| **컨텍스트** | 결과만 상위로 반환 | 완전 독립 |
| **통신** | 메인 에이전트에게만 보고 | 팀원끼리 직접 메시징 |
| **조율** | 메인이 모든 작업 관리 | 공유 태스크 리스트로 자율 조율 |
| **비용** | 낮음 | 높음 (인스턴스 × N) |
| **적합 용도** | 결과만 중요한 집중 작업 | 토론/협업이 필요한 복잡한 작업 |

### 사용 예시

#### 병렬 코드 리뷰

```
PR #142를 에이전트 팀으로 리뷰해줘. 3명의 리뷰어 생성:
- 보안 관점
- 성능 영향
- 테스트 커버리지 검증
각자 리뷰 후 발견 사항을 보고해줘.
```

#### 경쟁 가설 디버깅

```
앱이 메시지 하나 보낸 후 종료되는 버그가 있어.
5명의 에이전트 팀을 생성해서 각각 다른 가설을 조사해줘.
서로의 이론을 반박하도록 하고 (과학적 토론처럼),
합의된 결과를 문서에 정리해줘.
```

#### 크로스 레이어 기능 개발

```
인증 모듈 리팩토링을 에이전트 팀으로 진행해줘:
- 프론트엔드 담당 1명
- 백엔드 담당 1명
- 테스트 담당 1명
각자 Sonnet 모델 사용. 구현 전 계획 승인 필수.
```

### 디스플레이 모드

| 모드 | 설명 | 요구사항 |
|------|------|---------|
| `in-process` | 메인 터미널에서 통합 실행. `Shift+Down`으로 팀원 전환 | 없음 |
| `tmux` / `iTerm2` | 팀원마다 별도 패널 | tmux 또는 iTerm2 |

```json
{ "teammateMode": "in-process" }
```

### 베스트 프랙티스

- **3~5명**으로 시작 (비용 대비 효율 최적)
- 팀원당 **5~6개 태스크** 배분
- **파일 충돌 방지**: 팀원마다 다른 파일 세트 담당
- 연구/리뷰 작업부터 시작하여 익숙해진 후 구현 작업으로 확장
- 리드가 직접 작업하려 하면 `"팀원들이 완료할 때까지 기다려"` 지시

---

## 6. Plugin 생태계

### 플러그인 매니저

```bash
/plugin                          # 인터랙티브 UI (Discover/Installed/Marketplaces/Errors)
/plugin install name@marketplace  # CLI 설치
/plugin disable name@marketplace  # 비활성화
/plugin enable name@marketplace   # 재활성화
/plugin uninstall name@marketplace # 제거
/reload-plugins                   # 세션 중 변경 적용
```

### 공식 마켓플레이스 주요 플러그인

#### 외부 서비스 통합

| 플러그인 | 서비스 |
|---------|--------|
| `github` | GitHub PR, 이슈, 코드 리뷰 |
| `gitlab` | GitLab 통합 |
| `atlassian` | Jira/Confluence |
| `linear` | Linear 프로젝트 관리 |
| `asana` | Asana 태스크 관리 |
| `notion` | Notion 문서 |
| `figma` | Figma 디자인 |
| `slack` | Slack 메시징 |
| `sentry` | 에러 모니터링 |
| `vercel` | Vercel 배포 |
| `firebase` | Firebase 통합 |
| `supabase` | Supabase 통합 |

#### 개발 워크플로우

| 플러그인 | 용도 |
|---------|------|
| `commit-commands` | Git 커밋/푸시/PR 워크플로우 |
| `pr-review-toolkit` | PR 리뷰 전문 에이전트 |
| `agent-sdk-dev` | Agent SDK 개발 도구 |
| `plugin-dev` | 플러그인 개발 도구 |

#### 출력 스타일

| 플러그인 | 용도 |
|---------|------|
| `explanatory-output-style` | 구현 선택에 대한 교육적 설명 |
| `learning-output-style` | 인터랙티브 학습 모드 |

### 설치 스코프

| 스코프 | 적용 범위 | 설정 파일 |
|--------|----------|----------|
| **User** | 모든 프로젝트 (개인) | `~/.claude/settings.json` |
| **Project** | 리포지토리 전체 (팀 공유) | `.claude/settings.json` |
| **Local** | 이 리포지토리, 나만 | `.claude/settings.local.json` |
| **Managed** | 관리자 배포, 수정 불가 | `managed-settings.json` |

### 마켓플레이스 추가

```bash
# GitHub 리포지토리
/plugin marketplace add anthropics/claude-code

# GitLab/Bitbucket
/plugin marketplace add https://gitlab.com/company/plugins.git

# 특정 브랜치/태그
/plugin marketplace add https://gitlab.com/company/plugins.git#v1.0.0

# 로컬 경로
/plugin marketplace add ./my-marketplace
```

---

## 7. 코드 인텔리전스 (LSP)

LSP 플러그인은 Claude에게 **IDE 수준의 코드 이해력**을 부여한다.

### 지원 언어 및 플러그인

| 언어 | 플러그인 | 필요 바이너리 |
|------|---------|-------------|
| Python | `pyright-lsp` | `pyright-langserver` |
| TypeScript | `typescript-lsp` | `typescript-language-server` |
| Java | `jdtls-lsp` | `jdtls` |
| Kotlin | `kotlin-lsp` | `kotlin-language-server` |
| Go | `gopls-lsp` | `gopls` |
| Rust | `rust-analyzer-lsp` | `rust-analyzer` |
| C/C++ | `clangd-lsp` | `clangd` |
| C# | `csharp-lsp` | `csharp-ls` |
| PHP | `php-lsp` | `intelephense` |
| Swift | `swift-lsp` | `sourcekit-lsp` |
| Ruby | (추가 가능) | — |

### 설치 예시 (Java)

```bash
# 1. LSP 바이너리 설치
brew install jdtls

# 2. 플러그인 설치
/plugin install jdtls-lsp@claude-plugins-official
```

### Claude가 얻는 능력

| 기능 | 설명 |
|------|------|
| **자동 진단** | 파일 편집 후 타입 에러, 누락 import 등 자동 감지 및 수정 |
| **정의로 이동** (goToDefinition) | 심볼의 정의 위치 탐색 |
| **참조 찾기** (findReferences) | 심볼의 모든 사용처 검색 |
| **호버** (hover) | 타입 정보 조회 |
| **심볼 목록** (documentSymbol) | 파일 내 심볼 구조 파악 |

> grep 기반 검색 대비 **약 900배 성능 향상** (50ms vs 45초)

---

## 8. SDK 및 프로그래밍 방식 연동

### Headless 모드 (비대화형)

```bash
# 단순 쿼리
claude -p "이 프로젝트가 무엇을 하는지 설명해줘"

# JSON 출력
claude -p "모든 API 엔드포인트 목록" --output-format json

# 스트리밍 JSON
claude -p "이 로그 파일 분석" --output-format stream-json

# 파이프라인 통합
claude -p "분석 프롬프트" --output-format json | your_command
```

### Claude Agent SDK

프로그래밍 방식으로 Claude Code를 내부 도구에 통합.

```bash
# TypeScript SDK
npm install @anthropic-ai/claude-agent-sdk

# Python SDK
pip install claude-agent-sdk
```

**지원 인증:**
- Anthropic API 키 직접 사용
- Amazon Bedrock (AWS 인프라)
- Google Cloud Vertex AI

#### TypeScript SDK 예시 — PR 자동 코드 리뷰

```typescript
import { ClaudeAgent } from "@anthropic-ai/claude-agent-sdk";

const agent = new ClaudeAgent({
  model: "claude-sonnet-4-6",
  apiKey: process.env.ANTHROPIC_API_KEY,
  allowedTools: ["Read", "Grep", "Glob", "Bash(gh *)"],
  maxTurns: 20,
});

// PR diff를 분석하여 리뷰 코멘트 생성
const result = await agent.run({
  prompt: `
    gh pr diff ${prNumber}의 변경사항을 리뷰하세요.
    보안 취약점, 성능 이슈, 코드 스타일 위반을 중심으로 확인.
    결과를 JSON 형식으로 반환: { "comments": [{ "file", "line", "severity", "message" }] }
  `,
  outputFormat: "json",
});

// 결과를 PR 코멘트로 등록
for (const comment of result.comments) {
  await gh.createReviewComment(prNumber, comment);
}
```

#### Python SDK 예시 — 이슈 자동 트리아지

```python
from claude_agent_sdk import ClaudeAgent

agent = ClaudeAgent(
    model="claude-sonnet-4-6",
    allowed_tools=["Read", "Grep", "Glob", "Bash(gh *)"],
    max_turns=10,
)

# GitHub 이슈를 분석하여 라벨 추천
result = agent.run(
    prompt=f"gh issue view {issue_number}를 분석하고 "
           f"적절한 라벨(bug/feature/docs/chore)과 담당자를 추천하세요.",
    output_format="json",
)

# 라벨 자동 적용
subprocess.run(["gh", "issue", "edit", str(issue_number),
                "--add-label", result["label"]])
```

### 활용 사례

| 용도 | 설명 |
|------|------|
| CI/CD 파이프라인 | 빌드 후 자동 코드 리뷰 |
| 커스텀 개발 도구 | 내부 CLI 도구에 AI 코딩 기능 추가 |
| 자동화 워크플로우 | 이슈 트리아지, 릴리스 노트 생성 |
| 내부 앱 통합 | 사내 도구에 코드 분석 기능 임베딩 |

### 대규모 마이그레이션 (Fan-out)

```bash
# 파일 목록 생성
claude -p "마이그레이션 필요한 2000개 Python 파일 목록을 만들어줘" > files.txt

# 병렬 처리
for file in $(cat files.txt); do
  claude -p "$file을 Python 3.12 문법으로 마이그레이션" \
    --allowedTools "Edit,Bash(git commit *)" &
done
wait
```

---

## 9. 팀 마켓플레이스 구축

팀 전용 플러그인을 중앙에서 관리·배포하는 마켓플레이스.

### 프로젝트 설정에 팀 마켓플레이스 추가

```json
// .claude/settings.json
{
  "extraKnownMarketplaces": {
    "our-team-tools": {
      "source": {
        "source": "github",
        "repo": "our-org/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "formatter@our-team-tools": true,
    "code-standards@our-team-tools": true
  }
}
```

팀원이 리포지토리를 trust하면 자동으로 마켓플레이스 및 플러그인 설치를 안내받는다.

### Managed 마켓플레이스 (엔터프라이즈)

```json
// managed-settings.json
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "our-org/approved-plugins",
      "ref": "v2.0"
    }
  ],
  "blockedMarketplaces": [
    {
      "source": "github",
      "repo": "untrusted-org/plugins"
    }
  ]
}
```

---

## 10. 보안 및 거버넌스

### 보안 체크리스트

| 항목 | 방법 |
|------|------|
| API 키 보호 | GitHub Secrets, 환경 변수 사용. settings에 직접 포함 금지 |
| 민감 파일 접근 차단 | `deny` 규칙으로 `.env`, `.ssh`, `.aws` 등 차단 |
| 네트워크 제한 | `sandbox.network.allowedDomains`로 허용 도메인 제한 |
| 위험 명령 차단 | `deny`로 `rm -rf`, `curl`, `wget` 등 차단 |
| 권한 우회 차단 | `disableBypassPermissionsMode: true` |
| 플러그인 제한 | `strictKnownMarketplaces`로 허가된 소스만 허용 |
| MCP 서버 제한 | `allowManagedMcpServersOnly: true` |
| Hook 제한 | `allowManagedHooksOnly: true` |

### Sandbox 모드

OS 수준 격리로 파일시스템/네트워크 접근을 제한.

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "filesystem": {
      "allowWrite": ["/opt/tools"]
    },
    "network": {
      "allowedDomains": [
        "github.company.com",
        "registry.npmjs.org"
      ]
    }
  }
}
```

### 설정 확인

```bash
# Claude Code 내에서
/status    # 모든 활성 설정 소스와 우선순위 표시
```

---

## 전체 아키텍처 요약

```
┌─────────────────────────────────────────────────────┐
│                  엔터프라이즈 관리                      │
│  managed-settings.json / MDM / Admin Console        │
│  (권한, MCP, Hook, 마켓플레이스 제한)                    │
└──────────────────────┬──────────────────────────────┘
                       │ (최우선)
┌──────────────────────▼──────────────────────────────┐
│                    팀 프로젝트                         │
│  .claude/settings.json  │  CLAUDE.md  │  .mcp.json  │
│  .claude/skills/        │  .claude/agents/           │
│  .github/workflows/claude.yml (CI/CD)               │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                    개인 설정                          │
│  ~/.claude/settings.json  │  ~/.claude/CLAUDE.md    │
│  ~/.claude/commands/      │  ~/.claude/skills/      │
│  ~/.claude/agents/        │  ~/.claude.json (MCP)   │
│  .claude/settings.local.json (로컬 오버라이드)        │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                    IDE 통합                           │
│  VS Code Extension  │  JetBrains Plugin             │
│  LSP Code Intelligence (11개 언어)                   │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                    실행 모드                          │
│  대화형 REPL  │  Headless (-p)  │  Agent Teams      │
│  SDK (TypeScript/Python)  │  GitHub Actions         │
└─────────────────────────────────────────────────────┘
```

---

## Sources

- [Claude Code Settings - 공식 문서](https://code.claude.com/docs/en/settings)
- [Claude Code Permissions](https://code.claude.com/docs/en/permissions)
- [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions)
- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [Claude Code Plugins - Discover](https://code.claude.com/docs/en/discover-plugins)
- [Claude Code MCP](https://code.claude.com/docs/en/mcp)
- [Claude Code Headless Mode](https://code.claude.com/docs/en/headless)
- [Agent SDK Overview](https://platform.claude.com/docs/en/agent-sdk/overview)
## 직접 확인해보기

- [ ] `.claude/settings.json`을 팀 저장소에 커밋하고, 팀원이 동일 설정으로 동작하는지 확인
- [ ] VS Code에서 `Cmd+Esc`로 Claude Code 패널 열기 확인
- [ ] GitHub Actions 워크플로우 1개 설정 후 PR 코멘트 동작 테스트
- [ ] `settings.json` 우선순위 이해: Enterprise > CLI > Local > Shared > User
- [ ] 보안 체크리스트 항목 중 3개 이상 점검 완료

---

## Sources

- [anthropics/claude-code-action (GitHub)](https://github.com/anthropics/claude-code-action)
- [Claude Code JetBrains Integration](https://code.claude.com/docs/en/jetbrains)
- [Piebald-AI/claude-code-lsps (GitHub)](https://github.com/Piebald-AI/claude-code-lsps)
- [managed-settings.json Guide](https://managed-settings.com/)
- [Configure Claude Code for Agent Teams (Medium)](https://medium.com/@haberlah/configure-claude-code-to-power-your-agent-team-90c8d3bca392)
- [Claude Code Agent Teams Guide 2026](https://claudefa.st/blog/guide/agents/agent-teams)
- [Shipyard - Multi-agent Orchestration 2026](https://shipyard.build/blog/claude-code-multi-agent/)
- [eesel.ai - Admin Controls Guide](https://www.eesel.ai/blog/admin-controls-claude-code)
- [eesel.ai - Plugin Ecosystem](https://www.eesel.ai/blog/claude-code-plugin)

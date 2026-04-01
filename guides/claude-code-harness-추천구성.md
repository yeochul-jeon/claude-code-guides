# Claude Code Harness 추천 구성 전략 가이드

| 항목 | 날짜 |
|------|------|
| 생성일 | 2026-03-11 |
| 변경일 | 2026-04-01 |

> 풀스택(Java/Spring + React/Next.js) + 인프라(Terraform/K8s) 개발자를 위한
> Claude Code 구성 전략 로드맵. 어떤 설정을 어떤 순서로, 왜 적용하는지에 대한 가이드.

### 관련 문서
- [개인 설정 가이드](claude-code-개인설정-가이드.md) — 각 설정 항목 상세 설명
- [CLAUDE.md 실전 작성법](claude-code-CLAUDE-md-실전-작성법.md) — CLAUDE.md 작성 패턴
- [팀 IDE 통합 가이드](claude-code-팀-IDE-통합-가이드.md) — Phase 4~5 확장

---

## 목차

1. [구성 전략 개요](#1-구성-전략-개요)
2. [Phase 1: 글로벌 기반 구축](#2-phase-1-글로벌-기반-구축)
3. [Phase 2: 프로젝트별 맞춤 구성](#3-phase-2-프로젝트별-맞춤-구성)
4. [Phase 3: 생산성 확장](#4-phase-3-생산성-확장)
5. [Phase 4: 팀 공유 및 표준화](#5-phase-4-팀-공유-및-표준화)
6. [Phase 5: 자동화 및 CI/CD](#6-phase-5-자동화-및-cicd)
7. [구성 진화 전략](#7-구성-진화-전략)
8. [전체 디렉토리 맵](#8-전체-디렉토리-맵)

---

## 1. 구성 전략 개요

### 핵심 철학

Addy Osmani의 원칙을 따른다:

> **"에이전트에게는 사무실 투어가 아니라 지뢰밭 경고만 제공하라."**

- `/init`으로 자동 생성된 내용은 대부분 중복. 에이전트가 스스로 발견할 수 없는 것만 설정
- CLAUDE.md는 **200줄 이하**, 글로벌은 **50줄 이하**
- 반복 실수가 발생할 때마다 규칙을 추가하고, 불필요하면 제거하는 **진단 도구**로 운영

### 5단계 로드맵

```
Phase 1: 글로벌 기반        ← 1일차에 완료 (즉시 효과)
Phase 2: 프로젝트별 맞춤     ← 프로젝트 시작 시 적용
Phase 3: 생산성 확장        ← 1~2주 사용 후 필요에 따라
Phase 4: 팀 공유            ← 팀 도입 시
Phase 5: CI/CD 자동화       ← 워크플로우 안정화 후
```

### 적용 원칙

| 원칙 | 설명 |
|------|------|
| **점진적 추가** | 처음부터 전부 설정하지 않는다. 문제가 생길 때 규칙을 추가 |
| **최소 설정** | 각 줄에 "이걸 삭제하면 Claude가 실수할까?" 자문 |
| **계층 분리** | 글로벌은 철학, 프로젝트는 기술, Skills는 도메인 지식 |
| **정기 정리** | 월 1회 CLAUDE.md 리뷰. 불필요한 규칙 제거 |

---

## 2. Phase 1: 글로벌 기반 구축

> **목표:** 모든 프로젝트에 공통 적용되는 개인 개발 철학과 금지사항 설정
> **소요:** 30분

### 2.1 글로벌 CLAUDE.md (`~/.claude/CLAUDE.md`)

**원칙:** 50줄 이하. 개발 철학 + 공통 금지사항 + 언어 설정만.

```markdown
# 개발 철학
- 간결하고 읽기 쉬운 코드를 우선시한다
- 기존 코드 스타일과 패턴을 존중한다
- 과도한 추상화보다 명확한 구현을 선호한다
- 작은 점진적 변경을 추구한다
- 시스템을 처음부터 재구현하기 전에 반드시 허락을 구한다

# 언어
- 모든 대화, 코드 설명, 주석은 한국어로 작성
- 커밋 메시지는 한국어 + 컨벤셔널 커밋 형식 (feat:, fix:, refactor:)

# 코드 규칙
- import 순서: 표준 라이브러리 → 프레임워크 → 서드파티 → 로컬
- 불필요한 추상화, 헬퍼, 유틸리티 클래스 생성 금지
- any 타입 사용 금지 (TypeScript)

# 금지사항
- --no-verify 절대 사용 금지
- 관련 없는 변경을 같은 커밋에 포함 금지
- 처음부터 재구현 금지 (반드시 허락 필요)
- 코드에 불필요한 주석/docstring 추가 금지

# 워크플로우
- 코드 변경 후 타입 체크 실행
- 전체 테스트 스위트 대신 관련 테스트만 실행
- 의심스러운 결정은 먼저 질문하고 가정하지 말 것
- GitHub 작업에는 항상 gh CLI 사용
```

**이 파일이 효과적인 이유:**
- 기술 스택 언급 없음 (프로젝트마다 다르므로)
- Claude가 이미 아는 것 없음 (표준 컨벤션 등)
- 반복 실수 방지 규칙만 포함

### 2.2 글로벌 Settings (`~/.claude/settings.json`)

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
      "Bash(ls *)",
      "Bash(cat *)",
      "Bash(mkdir *)",
      "Bash(which *)",
      "Bash(echo *)"
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

**설계 의도:**
- 읽기/검색/편집은 항상 허용 → 승인 피로 제거
- git/gh는 허용하되 push는 프로젝트별로 판단
- `.env` 파일 접근은 글로벌 차단
- `thinking: "always"` → 복잡한 작업에서 눈에 띄게 나은 결과

### 2.3 상태 표시줄 (선택, 강력 권장)

컨텍스트 사용량 모니터링은 **가장 중요한 리소스 관리** 도구.

```bash
# ~/.claude/statusline.sh
#!/bin/bash
REPO=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "no-repo")
echo "${REPO} | ctx: ${CLAUDE_CONTEXT_PERCENT:-0}%"
```

---

## 3. Phase 2: 프로젝트별 맞춤 구성

> **목표:** 각 프로젝트 유형에 맞는 CLAUDE.md + AGENTS.md + 로컬 설정
> **시점:** 프로젝트 시작 시 또는 Claude Code 처음 사용 시

### 3.1 구성 전략: CLAUDE.md vs AGENTS.md 역할 분담

```
AGENTS.md  → 빌드 명령어, 테스트 명령어, 디렉토리 역할 (범용, 다른 AI 도구도 읽음)
CLAUDE.md  → Claude 전용 지시 (코드 스타일 강제, 아키텍처 규칙, skills/hooks 참조)
```

팀에서 Cursor, Copilot 등 다른 도구도 쓴다면 AGENTS.md를 별도로 만들고,
Claude Code만 쓴다면 CLAUDE.md 하나에 통합해도 무방.

### 3.2 백엔드 프로젝트 (Spring Boot)

**핵심 원칙:** JPA 함정, QueryDSL 패턴, 레이어 규칙에 집중. Spring Boot 기본 사항은 생략.

**CLAUDE.md 작성 전략:**

| 포함 (Claude가 발견 불가) | 제외 (Claude가 발견 가능) |
|--------------------------|--------------------------|
| 빌드 명령어 (`./gradlew` 경로) | Spring Boot 기본 구조 |
| Entity에 @Data 금지 규칙 | Controller/Service/Repository 역할 |
| QueryDSL의 BooleanExpression 패턴 | JPA 기본 사용법 |
| API 에러 응답 형식 | REST API 표준 |
| 테스트 슬라이스 전략 (@DataJpaTest 등) | JUnit 기본 사용법 |

**settings.json 추가:**

```json
{
  "permissions": {
    "allow": [
      "Bash(./gradlew *)",
      "Bash(gradle *)"
    ]
  }
}
```

### 3.3 프론트엔드 프로젝트 (Next.js)

**핵심 원칙:** 사용하는 UI 라이브러리(shadcn 등), 상태 관리 선택, 테스트 도구에 집중.

**CLAUDE.md 작성 전략:**

| 포함 | 제외 |
|------|------|
| `pnpm` 사용 (npm 아님) | React 기본 문법 |
| shadcn/ui 우선 사용 규칙 | Tailwind 기본 클래스 |
| React Query 쿼리 키 규칙 | Next.js App Router 구조 |
| import 순서 | TypeScript 기본 |
| MSW 모킹 전략 | Jest 기본 사용법 |

**settings.json 추가:**

```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm *)",
      "Bash(npx *)"
    ]
  }
}
```

### 3.4 인프라 프로젝트 (Terraform/K8s)

**핵심 원칙:** **보안이 최우선.** 파괴적 명령 차단, 시크릿 접근 제한.

**CLAUDE.md 작성 전략:**

| 포함 | 제외 |
|------|------|
| `terraform destroy` 사용자 확인 필수 | Terraform 기본 문법 |
| 상태 파일 원격 백엔드 위치 | HCL 구문 |
| IAM 최소 권한 원칙 강제 | kubectl 기본 명령 |
| 환경별 tfvars 파일 경로 | Helm 기본 사용법 |
| prod 변경은 PR 필수 | K8s 리소스 타입 설명 |

**settings.json 추가 (보안 강화):**

```json
{
  "permissions": {
    "allow": [
      "Bash(terraform plan *)",
      "Bash(terraform init *)",
      "Bash(kubectl get *)",
      "Bash(kubectl describe *)",
      "Bash(helm list *)"
    ],
    "deny": [
      "Bash(terraform destroy *)",
      "Bash(terraform apply *)",
      "Bash(kubectl delete *)",
      "Bash(helm uninstall *)",
      "Read(./*.tfvars)",
      "Read(./secrets/**)"
    ]
  }
}
```

> `terraform apply`와 `destroy`를 deny에 넣는 이유: 인프라 변경은 반드시 수동 확인 후 실행해야 하므로 Claude가 자동으로 실행하는 것을 원천 차단.

### 3.5 모노레포 전략

```
monorepo/
├── CLAUDE.md              # 공통: 커밋 규칙, 크로스 영역 테스트
├── AGENTS.md              # 범용: 빌드 명령어 모음 (선택)
├── backend/CLAUDE.md      # Spring Boot 규칙
├── frontend/CLAUDE.md     # Next.js 규칙
├── infra/CLAUDE.md        # Terraform 규칙 + 보안 강화
└── .claude/settings.json  # 통합 권한
```

루트 CLAUDE.md는 **크로스 영역 규칙만**:

```markdown
# 크로스 영역 규칙
- backend API 변경 시 frontend 타입 동기화 확인
- DB 스키마 변경 시 backend + infra 양쪽 영향도 확인
- 각 영역별 상세 규칙은 하위 CLAUDE.md 참조
```

---

## 4. Phase 3: 생산성 확장

> **목표:** 반복 작업 자동화, 도메인 지식 축적
> **시점:** 1~2주 사용 후, 반복 패턴이 보일 때

### 4.1 Skills: 필요할 때만 로드되는 도메인 지식

**판단 기준:** CLAUDE.md에 넣기엔 특정 상황에서만 필요한 내용 → Skill로 분리

**추천 구성 순서:**

| 우선순위 | Skill | 이유 |
|---------|-------|------|
| 1순위 | `fix-issue` | GitHub 이슈 → PR 워크플로우 자동화 |
| 2순위 | `code-review` | 일관된 리뷰 체크리스트 |
| 3순위 | `db-migration` | DB 변경의 위험성, 절차 표준화 |
| 선택 | `api-design` | API 설계 시에만 필요한 세부 규칙 |
| 선택 | `perf-audit` | 성능 분석 시에만 필요 |

**글로벌 vs 프로젝트 Skills:**

```
~/.claude/skills/           # 모든 프로젝트 공통
├── code-review/SKILL.md    # 범용 코드 리뷰
└── fix-issue/SKILL.md      # GitHub 이슈 처리

.claude/skills/             # 프로젝트 전용
├── api-design/SKILL.md     # 이 프로젝트의 API 규칙
└── db-migration/SKILL.md   # 이 프로젝트의 DB 마이그레이션 절차
```

### 4.2 Subagents: 전문화된 리뷰어

**판단 기준:** 특정 관점에서 별도 컨텍스트로 분석이 필요할 때

**추천 구성:**

```
~/.claude/agents/            # 글로벌 (모든 프로젝트에서 사용)
├── security-reviewer.md     # 보안 리뷰 전문
└── debugger.md              # 디버깅 전문

.claude/agents/              # 프로젝트 전용
├── spring-reviewer.md       # Spring 패턴 리뷰
└── infra-reviewer.md        # IaC 리뷰
```

### 4.3 Hooks: 100% 실행 보장이 필요한 것

**판단 기준:** CLAUDE.md에 써도 가끔 무시되는 규칙 → Hook으로 강제

**추천 순서:**

| 우선순위 | Hook | 효과 |
|---------|------|------|
| 1순위 | 민감 파일 쓰기 차단 | `.env`, `.key`, `.pem` 수정 차단 |
| 2순위 | 자동 포맷팅 | 파일 편집 후 자동 prettier/checkstyle |
| 선택 | 타입 체크 | 편집 후 tsc/javac 자동 실행 |

### 4.4 Slash Commands: 빈번한 반복 작업

```
~/.claude/commands/          # 글로벌
├── review.md                # /review → 현재 브랜치 변경사항 리뷰
├── pr.md                    # /pr → PR 생성
└── standup.md               # /standup → 오늘 작업 요약
```

### 4.5 코드 인텔리전스 (LSP Plugin)

풀스택 + 인프라 환경에서 설치할 LSP:

| 언어 | 플러그인 | 설치 명령 |
|------|---------|----------|
| Java | `jdtls-lsp` | `brew install jdtls` |
| TypeScript | `typescript-lsp` | `npm i -g typescript-language-server` |
| Python | `pyright-lsp` | `pip install pyright` |

```bash
# Claude Code 내에서
/plugin install jdtls-lsp@claude-plugins-official
/plugin install typescript-lsp@claude-plugins-official
```

---

## 5. Phase 4: 팀 공유 및 표준화

> **목표:** 팀원 온보딩 간소화, 공통 규칙 표준화
> **시점:** 팀에 Claude Code 도입 시

### 5.1 Git에 커밋할 파일

```
.claude/
├── settings.json            # 팀 공용 권한 → git 커밋
├── skills/                  # 팀 공용 스킬 → git 커밋
│   └── fix-issue/SKILL.md
├── agents/                  # 팀 공용 에이전트 → git 커밋
│   └── security-reviewer.md
└── settings.local.json      # 개인 오버라이드 → .gitignore

CLAUDE.md                    # 팀 공용 프로젝트 규칙 → git 커밋
AGENTS.md                    # 크로스 에이전트 공통 → git 커밋 (선택)
```

### 5.2 .gitignore에 추가

```gitignore
.claude/settings.local.json
```

### 5.3 팀 마켓플레이스 (선택)

팀 전용 플러그인이 여러 개라면:

```json
// .claude/settings.json
{
  "extraKnownMarketplaces": {
    "our-team": {
      "source": {
        "source": "github",
        "repo": "our-org/claude-plugins"
      }
    }
  }
}
```

---

## 6. Phase 5: 자동화 및 CI/CD

> **목표:** GitHub Actions로 코드 리뷰/이슈 구현 자동화
> **시점:** 팀 워크플로우 안정화 후

### 6.1 도입 순서

```
1단계: @claude 멘션 응답            ← 가장 낮은 위험
2단계: PR 자동 코드 리뷰            ← 리뷰 보조
3단계: 이슈 자동 구현 (라벨 기반)    ← 단순 작업만
4단계: 일일 리포트 / 릴리스 노트     ← 정보 수집
```

### 6.2 1단계 워크플로우

```yaml
# .github/workflows/claude.yml
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

### 6.3 비용 관리 팁

| 전략 | 설명 |
|------|------|
| `--max-turns 5~10` | 반복 횟수 제한 |
| 라벨 기반 트리거 | 모든 이슈가 아닌 특정 라벨만 처리 |
| concurrency 제어 | 동시 실행 제한 |
| 모델 선택 | 단순 작업은 Sonnet, 복잡한 작업만 Opus |

---

## 7. 구성 진화 전략

### 7.1 주기적 점검 체크리스트

**매주:**
- [ ] Claude가 반복하는 실수가 있는가? → CLAUDE.md에 규칙 추가
- [ ] CLAUDE.md에 있지만 Claude가 이미 잘 하는 것은? → 삭제
- [ ] 컨텍스트 사용률이 자주 60%를 넘는가? → `/clear` 습관화

**매월:**
- [ ] CLAUDE.md가 200줄을 넘었는가? → Skills/rules로 분리
- [ ] 사용하지 않는 Skill이 있는가? → 제거
- [ ] Hook이 워크플로우를 방해하는가? → 조정
- [ ] 새로운 팀 컨벤션이 생겼는가? → CLAUDE.md 업데이트

### 7.2 진화 패턴

```
Phase 1 → 2주 사용 → "아, 이것도 필요하네" → Phase 2에 규칙 추가
Phase 2 → 1달 사용 → "이 규칙 Claude가 이미 알아" → 불필요한 규칙 삭제
Phase 3 → 반복 작업 발견 → Skill/Command 생성
Phase 4 → 팀 도입 → 공용 설정 커밋
Phase 5 → 안정화 → CI/CD 자동화
```

### 7.3 Anti-patterns (피해야 할 것)

| Anti-pattern | 문제 | 해결 |
|-------------|------|------|
| CLAUDE.md 500줄 | 규칙 무시됨 | 200줄 이하 + Skills 분리 |
| `/init`으로 생성 후 방치 | 중복 정보, 비용 증가 | 수동 작성, 정기 정리 |
| 모든 것을 Hook으로 | 워크플로우 느려짐 | 진짜 필수인 것만 Hook |
| MCP 서버 10개+ | 도구 선택 혼란 | 1~2개부터 시작 |
| 매번 thinking 모드 | 단순 작업에 불필요한 비용 | 기본 on, 단순 작업은 `/fast` |

---

## 8. 전체 디렉토리 맵

### 개인 글로벌 (`~/.claude/`)

```
~/.claude/
├── CLAUDE.md                    # 글로벌 지침 (50줄 이하)
├── settings.json                # 글로벌 권한 + thinking: always
├── statusline.sh                # 상태 표시줄 (컨텍스트 모니터링)
├── commands/                    # 글로벌 슬래시 명령
│   ├── review.md                # /review
│   ├── pr.md                    # /pr
│   └── standup.md               # /standup
├── skills/                      # 글로벌 스킬
│   ├── code-review/SKILL.md     # 범용 코드 리뷰
│   └── fix-issue/SKILL.md       # GitHub 이슈 처리
└── agents/                      # 글로벌 서브에이전트
    ├── security-reviewer.md     # 보안 리뷰
    └── debugger.md              # 디버깅 전문
```

### 프로젝트 (모노레포 예시)

```
project/
├── CLAUDE.md                    # 프로젝트 규칙 (git 커밋)
├── AGENTS.md                    # 크로스 에이전트 공통 (선택)
├── .mcp.json                    # MCP 서버 (필요 시)
│
├── backend/
│   └── CLAUDE.md                # Spring Boot 규칙
├── frontend/
│   └── CLAUDE.md                # Next.js 규칙
├── infra/
│   └── CLAUDE.md                # Terraform 규칙 (보안 강화)
│
└── .claude/
    ├── settings.json            # 팀 공용 권한 (git 커밋)
    ├── settings.local.json      # 개인 오버라이드 (.gitignore)
    ├── skills/                  # 프로젝트 스킬
    │   ├── api-design/SKILL.md
    │   └── db-migration/SKILL.md
    ├── agents/                  # 프로젝트 에이전트
    │   ├── spring-reviewer.md
    │   └── infra-reviewer.md
    └── hooks/                   # Hook 스크립트 (필요 시)
```

### 설정 우선순위 (적용 순서)

```
[1] managed-settings.json   ← 엔터프라이즈 (Phase 4 이후)
[2] CLI 인자                ← 세션 임시
[3] settings.local.json     ← 개인 로컬 오버라이드
[4] .claude/settings.json   ← 프로젝트 팀 공용
[5] ~/.claude/settings.json ← 글로벌 개인 기본값
```

---

## 관련 문서

| 문서 | 내용 | 파일 |
|------|------|------|
| 개인 설정 가이드 | CLAUDE.md, hooks, skills, MCP 등 상세 | `claude-code-개인설정-가이드.md` |
| 팀/IDE/통합 가이드 | 팀 설정, IDE, CI/CD, SDK, Agent Teams | `claude-code-팀-IDE-통합-가이드.md` |
| CLAUDE.md 실전 작성법 | 글로벌/프로젝트별 예시, AGENTS.md, Osmani 원칙 | `claude-code-CLAUDE-md-실전-작성법.md` |

---

## 직접 확인해보기

- [ ] Phase 1 완료: `~/.claude/CLAUDE.md` + `~/.claude/settings.json` 설정
- [ ] Phase 2 완료: 프로젝트 1개에 `CLAUDE.md` + `.claude/settings.json` 작성
- [ ] Phase 3 완료: Skill 또는 Hook 1개 이상 추가
- [ ] `claude` 실행 후 CLAUDE.md 지침이 반영되는지 테스트 (예: 언어 설정 확인)
- [ ] 불필요한 설정 없이 최소한으로 시작했는지 점검

---

## Sources

- [Addy Osmani - Stop Using /init for AGENTS.md](https://addyosmani.com/blog/agents-md/)
- [Best Practices for Claude Code - 공식 문서](https://code.claude.com/docs/en/best-practices)
- [Claude Code Settings](https://code.claude.com/docs/en/settings)
- [Claude Code Skills](https://code.claude.com/docs/en/skills)
- [Claude Code Hooks](https://code.claude.com/docs/en/hooks)
- [Claude Code Permissions](https://code.claude.com/docs/en/permissions)
- [Joe Cotellese - Global Configuration](https://joecotellese.com/posts/claude-code-global-configuration/)
- [Builder.io - How to Write a Good CLAUDE.md](https://www.builder.io/blog/claude-md-guide)
- [Freek Van der Herten - My Claude Code Setup](https://freek.dev/3026-my-claude-code-setup)

# Claude Code Guides

Claude Code(Anthropic CLI) 활용을 위한 한국어 실전 가이드 모음입니다.

## 대상 독자

- Claude Code를 처음 도입하는 개발자
- 팀 표준 구성을 설계하는 테크 리드
- 엔터프라이즈 거버넌스를 담당하는 DevOps/IT 관리자

## 문서 목록

아래 순서대로 읽으면 개인 설정부터 팀 통합까지 단계적으로 학습할 수 있습니다.

| 순서 | 문서 | 설명 |
|:---:|------|------|
| 0 | [Quick Start](guides/claude-code-quick-start.md) | 5분 안에 Claude Code 설치부터 첫 사용까지 |
| 1 | [개인 설정 가이드](guides/claude-code-개인설정-가이드.md) | CLAUDE.md, settings.json, Hooks, Skills, MCP 서버 등 개인 워크스페이스 구성 |
| 2 | [Harness 추천 구성](guides/claude-code-harness-추천구성.md) | 5단계 구성 전략 로드맵 — 어떤 설정을 어떤 순서로, 왜 적용하는지 |
| 3 | [CLAUDE.md 실전 작성법](guides/claude-code-CLAUDE-md-실전-작성법.md) | 글로벌/프로젝트별 CLAUDE.md 패턴, AGENTS.md 표준, 백엔드/프론트엔드/인프라 템플릿 |
| 4 | [팀 IDE 통합 가이드](guides/claude-code-팀-IDE-통합-가이드.md) | 팀 공유 설정, IDE 통합, CI/CD, Agent Teams, Plugin, SDK, 보안 거버넌스 |
| 5 | [Codex 공식 플러그인 완전 가이드](guides/claude-code-codex-플러그인-완전-가이드.md) | OpenAI Codex 플러그인 설치, 코드 리뷰, 작업 위임 — 초보자 완전 가이드 |
| - | [FAQ & 트러블슈팅](guides/claude-code-FAQ.md) | 자주 묻는 질문과 오류 해결 통합 가이드 |

### 팀 공유 템플릿

| 파일 | 설명 |
|------|------|
| [team-settings.json](templates/team-settings.json) | 팀 공유 settings.json (권한, Hooks, 알림) |
| [managed-settings.json](templates/managed-settings.json) | 엔터프라이즈 관리자용 강제 설정 |
| [protect-files.sh](templates/protect-files.sh) | 민감 파일 수정 차단 Hook 스크립트 |
| [팀 온보딩 체크리스트](templates/team-onboarding-checklist.md) | 신규 팀원 30분 설정 완료 가이드 |
| [CLAUDE.md.backend](templates/CLAUDE.md.backend) | 백엔드 프로젝트 CLAUDE.md (Java/Spring) |
| [CLAUDE.md.frontend](templates/CLAUDE.md.frontend) | 프론트엔드 프로젝트 CLAUDE.md (React/Next.js) |
| [CLAUDE.md.infra](templates/CLAUDE.md.infra) | 인프라/DevOps CLAUDE.md (Terraform/K8s) |
| [CLAUDE.md.docs](templates/CLAUDE.md.docs) | 문서 프로젝트 CLAUDE.md |

### 도구 연동 (CJ 내부)

| 문서 | 설명 |
|------|------|
| [Confluence CLI 설정](internal/confluence-cli-설정-가이드.md) | CJ Confluence Server/DC 연결 및 CLI 활용법 |

## 핵심 철학

> "에이전트에게 사무실 투어가 아니라 지뢰밭 경고를 제공하라" — Addy Osmani

모든 가이드는 이 원칙을 따릅니다:
- CLAUDE.md는 50~200줄 이내로, Claude가 스스로 발견할 수 없는 것만 포함
- 설정은 점진적으로 추가 (한 번에 모든 것을 구성하지 않음)
- 실전 예시와 안티패턴을 함께 제공

## 기여 방법

1. 이 저장소를 clone합니다
2. 문서를 수정하거나 새 가이드를 추가합니다
3. PR을 생성합니다

문서 작성 시 기존 문서의 형식(메타데이터 테이블, 목차, 코드 블록 스타일)을 따라주세요.

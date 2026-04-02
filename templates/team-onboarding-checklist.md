# Claude Code 팀 온보딩 체크리스트

> 신규 팀원이 **30분 내**에 Claude Code 팀 설정을 완료하기 위한 체크리스트

## 사전 요구사항

- [ ] Node.js 18+ 설치 확인
- [ ] Claude Code 최신 버전 설치: `npm install -g @anthropic-ai/claude-code`
- [ ] Anthropic 계정 또는 조직 로그인 완료: `claude login`

## Step 1: 프로젝트 Clone & 설정 확인 (5분)

- [ ] 프로젝트 clone: `git clone <repo-url> && cd <project>`
- [ ] `.claude/settings.json` 존재 확인 (팀 공유 설정)
- [ ] `.mcp.json` 존재 확인 (팀 MCP 서버)
- [ ] 환경변수 설정 (`.env.local` 또는 시스템 환경변수)

## Step 2: 개인 설정 (10분)

- [ ] 글로벌 CLAUDE.md 확인/생성: `~/.claude/CLAUDE.md`
- [ ] 개인 settings.json 확인: `~/.claude/settings.json`
- [ ] 로컬 오버라이드 필요 시: `.claude/settings.local.json` 생성

## Step 3: MCP 서버 연결 확인 (5분)

- [ ] `claude mcp list`로 팀 MCP 서버 목록 확인
- [ ] 인증이 필요한 MCP 서버 토큰 설정 (환경변수)
- [ ] Claude Code에서 MCP 도구 호출 테스트

## Step 4: 동작 확인 (10분)

- [ ] `claude` 실행 후 "이 프로젝트의 규칙을 설명해줘"로 CLAUDE.md 인식 확인
- [ ] 파일 편집 후 자동 포맷팅 Hook 동작 확인
- [ ] 보호 파일(.env 등) 수정 시도 → 차단 확인
- [ ] 팀 Slash Command 동작 확인 (있는 경우)
- [ ] `/simplify`로 내장 Skill 동작 확인

## 문제 해결

| 증상 | 해결 |
|------|------|
| CLAUDE.md가 인식되지 않음 | 파일 위치 확인 (프로젝트 루트 또는 `.claude/`) |
| Hook이 실행되지 않음 | `jq` 설치 확인, 스크립트 실행 권한 (`chmod +x`) |
| MCP 서버 연결 실패 | 환경변수 확인, `claude mcp list`로 상태 확인 |
| 권한 거부 | `.claude/settings.local.json`에 개인 allow 규칙 추가 |

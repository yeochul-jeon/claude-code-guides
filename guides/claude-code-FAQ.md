# Claude Code FAQ & 트러블슈팅

| 항목 | 날짜 |
|------|------|
| 생성일 | 2026-04-01 |
| 변경일 | 2026-04-02 |

> 각 가이드에 분산된 트러블슈팅 내용을 통합 정리한 문서

### 관련 문서
- [개인 설정 가이드](claude-code-개인설정-가이드.md) — §12 흔한 실수와 해결법
- [Confluence CLI 설정](../internal/confluence-cli-설정-가이드.md) — §6 트러블슈팅

---

## 목차

1. [일반 사용 문제](#1-일반-사용-문제)
2. [CLAUDE.md 관련](#2-claudemd-관련)
3. [권한 및 설정](#3-권한-및-설정)
4. [성능 및 컨텍스트](#4-성능-및-컨텍스트)
5. [도구 연동](#5-도구-연동)
6. [플랫폼별 참고사항](#6-플랫폼별-참고사항)

---

## 1. 일반 사용 문제

### Q: Claude가 지시를 무시하거나 엉뚱한 응답을 합니다

**원인**: 컨텍스트에 관련 없는 대화가 쌓여 혼란 발생 (Kitchen Sink 세션)

**해결**:
```
/clear     # 컨텍스트 초기화 후 새로운 프롬프트로 재시작
```

**예방**: 작업 단위로 `/clear`를 사용하여 컨텍스트를 분리합니다.

### Q: 같은 수정을 2번 이상 요청해도 고쳐지지 않습니다

**원인**: 초기 프롬프트가 불명확하여 Claude가 의도를 잘못 파악

**해결**:
1. `/clear`로 대화 초기화
2. 더 구체적인 프롬프트로 재시작 (파일 경로, 예상 동작, 현재 오류를 명시)

### Q: 그럴듯하지만 동작하지 않는 코드를 생성합니다

**해결**: 항상 검증 단계를 포함합니다.
- 테스트: `./gradlew test` 또는 `npm test` 실행 요청
- 린터: 코드 스타일 검사 실행
- 스크린샷: UI 변경 시 결과 확인 요청

---

## 2. CLAUDE.md 관련

### Q: CLAUDE.md가 너무 길어지면 어떻게 해야 하나요?

**기준**: 200줄 이하를 유지합니다.

**해결**: 도메인 지식, 반복 패턴은 **Skills**로 분리합니다.
```
.claude/skills/
├── spring-boot/SKILL.md    # Spring 관련 규칙
├── react/SKILL.md          # React 관련 규칙
└── testing/SKILL.md        # 테스트 패턴
```

자세한 내용: [CLAUDE.md 실전 작성법 §8](claude-code-CLAUDE-md-실전-작성법.md)

### Q: CLAUDE.md를 어디에 두어야 하나요?

| 위치 | 적용 범위 |
|------|----------|
| `~/.claude/CLAUDE.md` | 모든 프로젝트 (글로벌) |
| `프로젝트루트/CLAUDE.md` | 해당 프로젝트만 |
| `프로젝트루트/.claude/CLAUDE.md` | 해당 프로젝트만 (숨김) |

여러 파일이 존재하면 **모두 병합**되어 적용됩니다.

---

## 3. 권한 및 설정

### Q: 특정 명령만 허용하고 싶습니다

`settings.json`에서 패턴으로 제어합니다:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(./gradlew *)",
      "Edit(src/**)"
    ],
    "deny": [
      "Bash(rm *)",
      "Read(.env)"
    ]
  }
}
```

### Q: settings.json 우선순위가 헷갈립니다

높음 → 낮음 순서:

1. **Enterprise** (`managed-settings.json`) — MDM 배포, 사용자 변경 불가
2. **CLI 플래그** — `--allowedTools` 등 실행 시 지정
3. **Local** (`.claude/settings.local.json`) — 개인용, gitignore 대상
4. **Shared** (`.claude/settings.json`) — 팀 공유, 커밋 대상
5. **User** (`~/.claude/settings.json`) — 글로벌 개인 설정

---

## 4. 성능 및 컨텍스트

### Q: 컨텍스트가 부족하다는 경고가 나옵니다

**해결**:
- `/compact`로 대화를 요약하여 컨텍스트 확보
- `/clear`로 새 세션 시작
- 범위가 넓은 탐색은 서브에이전트로 위임

### Q: 대규모 마이그레이션 작업이 너무 오래 걸립니다

**해결**: Fan-out 패턴으로 병렬 처리합니다.

```bash
# 파일 목록 생성
claude -p "마이그레이션 필요한 파일 목록 생성" > files.txt

# 병렬 처리 (macOS/Linux)
for file in $(cat files.txt); do
  claude -p "$file 마이그레이션" \
    --allowedTools "Edit,Bash(git commit *)" &
done
wait
```

> **Windows(WSL)**: 위 스크립트는 bash 전용입니다. Windows에서는 WSL2 터미널에서 실행하세요.

---

## 5. 도구 연동

### Q: MCP 서버 연결이 실패합니다

**확인 순서**:
1. MCP 서버 프로세스가 실행 중인지 확인
2. `settings.json`의 MCP 설정에서 경로와 포트가 올바른지 확인
3. 방화벽/프록시가 연결을 차단하지 않는지 확인

### Q: Confluence CLI에서 `ETIMEDOUT` 에러가 발생합니다

**원인**: VPN 미연결 (사내 Confluence는 VPN 필수)

```bash
# 네트워크 확인
ping cjics.cj.net
```

VPN 연결 후 재시도합니다.

### Q: Confluence CLI에서 `401 Unauthorized` 에러가 발생합니다

**원인**: PAT 토큰 만료 또는 오류

**해결**:
1. Confluence 웹 → 개인 설정 → Personal Access Tokens에서 토큰 상태 확인
2. 만료 시 재발급 후 `~/.zshrc`의 `CONFLUENCE_API_TOKEN` 교체
3. `source ~/.zshrc` 실행

**독립 검증**:
```bash
curl -s -H "Authorization: Bearer $CONFLUENCE_API_TOKEN" \
  "https://cjics.cj.net/confluence/rest/api/space?limit=5" | python3 -m json.tool
```

### Q: `No configuration found` 에러가 발생합니다

환경변수가 로드되지 않은 경우:
```bash
echo $CONFLUENCE_DOMAIN       # 값이 출력되는지 확인
source ~/.zshrc               # 환경변수 재로드
```

### Q: MCP 서버가 도구 목록에 나타나지 않습니다

**확인 순서**:
1. `claude mcp list`로 등록된 서버 확인
2. `.mcp.json` 또는 `~/.claude.json`에 서버 설정이 올바른지 확인
3. stdio 타입: 해당 명령어(`npx`, `python` 등)가 PATH에 있는지 확인
4. http 타입: URL이 접근 가능한지 `curl`로 테스트
5. MCP 설정 변경 후에는 `claude --continue`로 세션 재시작

```bash
# 등록 확인
claude mcp list

# http 서버 연결 테스트
curl -s https://api.githubcopilot.com/mcp/ | head -5
```

### Q: Skills가 `/` 명령 목록에 보이지 않습니다

**확인 순서**:
1. SKILL.md 파일 위치 확인:
   - 개인: `~/.claude/skills/<name>/SKILL.md`
   - 프로젝트: `.claude/skills/<name>/SKILL.md`
2. YAML frontmatter가 올바른지 확인 (`---`로 시작/끝)
3. `name` 필드가 소문자+하이픈인지 확인 (대문자, 공백 불가)
4. `disable-model-invocation: true`인 경우 자동 호출 안됨 — `/skill-name`으로만 사용

```yaml
# 올바른 예시
---
name: my-skill
description: Does something useful
---
```

### Q: GitHub CLI(`gh`) 인증이 안 됩니다

```bash
# 인증 상태 확인
gh auth status

# 재인증 (브라우저 열림)
gh auth login

# 토큰으로 인증 (CI/CD 환경)
echo $GITHUB_TOKEN | gh auth login --with-token
```

> `gh` CLI는 Claude Code에서 GitHub 작업(PR, 이슈 등)에 필수. `Bash(gh *)`를 settings.json에 allow 해야 사용 가능.

### Q: Headless 모드(`claude -p`)에서 제한사항이 있나요?

| 항목 | 대화형 모드 | Headless (`-p`) |
|------|:---------:|:--------------:|
| 사용자 확인 프롬프트 | O | X (자동 거부) |
| 권한 요청 | O (팝업) | X (`--allowedTools`로 사전 지정) |
| Plan Mode | O | X |
| 스트리밍 출력 | O | `--output-format stream-json` |
| 멀티턴 대화 | O | X (단일 프롬프트) |

```bash
# Headless에서 권한을 사전 허용하려면
claude -p "코드 리뷰" --allowedTools "Read,Grep,Glob"

# JSON 출력
claude -p "API 목록" --output-format json

# 파이프라인 통합
cat error.log | claude -p "이 에러 분석해줘" --output-format json
```

---

## 6. 플랫폼별 참고사항

### Windows 사용자

| 항목 | macOS/Linux | Windows |
|------|-------------|---------|
| 설치 | `curl -fsSL https://claude.ai/install \| sh` | WSL2에서 동일 명령 또는 `npm install -g @anthropic-ai/claude-code` |
| 쉘 스크립트 | bash 직접 실행 | WSL2 터미널에서 실행 |
| 패키지 관리자 | `brew install jdtls` | `choco install jdtls` 또는 `scoop install jdtls` |
| 환경변수 | `~/.zshrc` 또는 `~/.bashrc` | WSL: `~/.bashrc` / PowerShell: `$PROFILE` |
| 병렬 처리 | bash `for ... &` + `wait` | WSL2에서 bash 스크립트 실행 |

### macOS 사용자

- Homebrew가 설치되어 있지 않으면: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
- Apple Silicon(M1+)에서는 네이티브 바이너리 설치를 권장합니다

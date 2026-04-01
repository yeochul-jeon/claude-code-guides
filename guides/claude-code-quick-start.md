# Claude Code Quick Start — 5분 시작 가이드

| 항목 | 날짜 |
|------|------|
| 생성일 | 2026-04-01 |
| 변경일 | 2026-04-01 |

> 최소한의 설정으로 Claude Code를 바로 사용할 수 있는 빠른 시작 가이드

### 관련 문서
- [개인 설정 가이드](claude-code-개인설정-가이드.md) — 각 설정 항목 심화 설명
- [CLAUDE.md 실전 작성법](claude-code-CLAUDE-md-실전-작성법.md) — 프로젝트별 CLAUDE.md 패턴

---

## 1. 설치 (1분)

```bash
# macOS / Linux
curl -fsSL https://claude.ai/install | sh
```

> **Windows 사용자**: WSL2 환경에서 위 명령을 실행하거나, `npm install -g @anthropic-ai/claude-code`를 사용하세요.

설치 확인:

```bash
claude --version
```

---

## 2. 프로젝트 초기화 (2분)

```bash
cd your-project
claude          # REPL 모드 진입
/init           # 프로젝트 분석 후 CLAUDE.md 자동 생성
```

`/init`이 생성한 `CLAUDE.md`를 열어 프로젝트에 맞게 수정합니다.

### 최소 CLAUDE.md 예시

```markdown
# 프로젝트 지침

## 기술 스택
- Java 17, Spring Boot 3.x, Gradle

## 빌드 및 테스트
- 빌드: `./gradlew build`
- 테스트: `./gradlew test`

## 규칙
- 한국어로 응답
- 생성자 주입 사용 (@RequiredArgsConstructor)
- Entity에 @Data 금지
```

---

## 3. 핵심 명령어 (2분)

| 명령어 | 설명 |
|--------|------|
| `claude` | REPL 모드 진입 |
| `claude -p "질문"` | 단발 질문 (Headless 모드) |
| `/clear` | 컨텍스트 초기화 |
| `/compact` | 대화 요약으로 컨텍스트 절약 |
| `/init` | CLAUDE.md 자동 생성 |
| `Esc` | 실행 중인 작업 중단 |

---

## 4. 바로 써보기

```
claude
> 이 프로젝트의 구조를 설명해줘
> src/main의 패키지 구조를 분석해줘
> UserService에 단위 테스트를 작성해줘
```

---

## 5. 다음 단계 체크리스트

- [ ] `CLAUDE.md`를 50~200줄 이내로 정리했는가?
- [ ] `settings.json`에서 불필요한 권한을 제한했는가? → [개인 설정 가이드 §3](claude-code-개인설정-가이드.md)
- [ ] 자주 쓰는 명령을 Hooks로 자동화했는가? → [개인 설정 가이드 §4](claude-code-개인설정-가이드.md)
- [ ] 팀 공유가 필요하다면 `.claude/settings.json`을 커밋했는가? → [팀 IDE 통합 가이드](claude-code-팀-IDE-통합-가이드.md)

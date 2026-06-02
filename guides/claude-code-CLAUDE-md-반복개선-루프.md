# CLAUDE.md 반복 개선 루프: 증거 기반 유지·보수 가이드 (2026)

| 항목 | 날짜 |
|------|------|
| 생성일 | 2026-06-02 |
| 변경일 | 2026-06-02 |

> CLAUDE.md를 처음 **작성**하는 방법은 [실전 작성법](claude-code-CLAUDE-md-실전-작성법.md)을 참조.
> 이 문서는 이미 존재하는 CLAUDE.md를 **증거 기반으로 반복 개선**하는 방법을 다룬다.

### 관련 문서
- [실전 작성법](claude-code-CLAUDE-md-실전-작성법.md) — 최초 작성·템플릿·진단 체크리스트
- [하네스 엔지니어링 방법론](claude-code-하네스-엔지니어링-방법론.md) — 월 1회 리뷰 루틴, If-Then 패턴
- [비용효율 측정 프레임워크](claude-code-비용효율-측정-프레임워크.md) — 메트릭 기반 피드백 루프

---

## 목차

1. [왜 반복 개선이 필요한가](#1-왜-반복-개선이-필요한가)
2. [개선 루프 3단계](#2-개선-루프-3단계)
3. [`claude-md-improver` 스킬 사용법](#3-claude-md-improver-스킬-사용법)
4. [`#` 단축키 — 세션 중 즉시 반영](#4--단축키--세션-중-즉시-반영)
5. [Staleness 감지 기준](#5-staleness-감지-기준)
6. [Progressive Disclosure 전략](#6-progressive-disclosure-전략)

---

## 1. 왜 반복 개선이 필요한가

CLAUDE.md는 한 번 작성하면 끝이 아니다. 코드베이스가 변하면 CLAUDE.md의 규칙도 낡아진다.

**CLAUDE.md 부패(Rot) 3가지 메커니즘**:

| 메커니즘 | 증상 | 예시 |
|---------|------|------|
| **Stale Commands** | Claude가 실행하면 오류가 나는 명령어를 사용 | `npm run build` → `pnpm build`로 바뀐 경우 |
| **Outdated Architecture** | Claude가 삭제된 디렉토리를 참조 | `src/utils/` → `lib/utils/`로 이동한 경우 |
| **Redundant Rules** | Claude가 이미 기본적으로 따르는 규칙이 여전히 존재 | "TypeScript를 사용하세요" (이미 파일 확장자가 `.ts`) |

**부패의 결과**: 규칙 밀도가 높아질수록 Claude가 전체 지시를 무시하기 시작한다. 이를 Context Rot라고 한다.

> **핵심**: 규칙 추가는 쉽고, 삭제는 어렵다. 개선 루프는 **삭제를 강제하는 구조**다.

---

## 2. 개선 루프 3단계

```
Observe → Improve → Verify
   ↑________________________↓
```

### Observe: 오류 패턴 수집

세션이 끝날 때마다 다음을 기록한다:

- Claude가 같은 실수를 반복한 횟수
- 내가 같은 교정을 반복한 횟수
- Claude가 지시를 무시한 횟수

**기록 기준**: 같은 문제가 **2번 이상** 발생하면 CLAUDE.md 후보. 1번은 일회성 가능성.

### Improve: Targeted Diff 적용

문제를 확인하면 **전체 재작성이 아니라 수술적 수정**만 한다:

```diff
- ## 테스트
- 테스트를 작성하세요.

+ ## 테스트
+ 단위 테스트: `pnpm test`
+ E2E 테스트: `pnpm test:e2e` (포트 3000 필요)
+ 테스트 없이 커밋 불가 — pre-commit hook이 차단함
```

**추가 기준**:
- Claude가 발견할 수 없는 정보인가? (파일에서 자동 추론 불가)
- 모든 세션에 필요한가? (일회성이면 대화로 해결)

**삭제 기준**:
- Claude가 이미 자연스럽게 따르는가?
- 코드를 보면 자명한 정보인가?

### Verify: 재실행으로 검증

수정 후 동일한 시나리오를 다시 실행해 규칙이 적용되는지 확인한다. Claude에게 "이 프로젝트의 테스트 방법을 설명해줘"처럼 직접 물어보는 것도 유효하다.

---

## 3. `claude-md-improver` 스킬 사용법

`claude-md-improver`는 Anthropic 공식 플러그인으로, 레포 내 모든 CLAUDE.md를 자동으로 감사·채점·개선한다.

### 설치

```bash
/plugin install anthropics/claude-plugins-official
```

설치 후 실행:

```
/claude-md-improver
```

### 5단계 워크플로

| 단계 | 설명 |
|------|------|
| **Discovery** | `find . -name "CLAUDE.md"` — 레포 내 모든 CLAUDE.md 발견 |
| **Quality Assessment** | 6개 기준으로 채점 (100점 만점) |
| **Quality Report** | 파일별 점수·등급·문제점 리포트 출력 |
| **Targeted Updates** | 사용자 승인 후 diff 형식으로 개선안 제시 |
| **Apply Updates** | Edit 도구로 실제 파일 수정 |

> **중요**: 스킬은 Quality Report를 먼저 출력하고 사용자 승인을 받은 후에만 파일을 수정한다.

### 채점 루브릭

| 기준 | 배점 | 체크 항목 |
|------|:----:|---------|
| Commands / Workflows | 20 | 빌드·테스트·배포 명령어가 정확히 있는가? |
| Architecture Clarity | 20 | 코드베이스 구조를 Claude가 이해할 수 있는가? |
| Non-obvious Patterns | 15 | 함정·quirk·비자명한 규칙이 문서화됐는가? |
| Conciseness | 15 | 자명한 정보나 장황한 설명이 없는가? |
| Currency | 15 | 현재 코드베이스 상태를 반영하는가? |
| Actionability | 15 | 모든 명령어가 복사-실행 가능한가? |
| **합계** | **100** | |

### 등급 해석

| 등급 | 점수 | 의미 |
|------|:----:|------|
| A | 90–100 | 포괄적·최신·실행 가능 |
| B | 70–89 | 양호, 사소한 누락 |
| C | 50–69 | 기본 정보만 있음, 주요 섹션 누락 |
| D | 30–49 | 희박하거나 낡음 |
| F | 0–29 | 없거나 심각하게 낡음 |

**목표**: B(70) 이상 유지. A는 과도한 최적화일 수 있다 — 너무 촘촘하면 유지 비용이 높아진다.

### 스킬이 감지하는 공통 문제

- 더 이상 작동하지 않는 빌드 명령
- 언급되지 않은 필수 의존성
- 변경된 파일 구조를 반영하지 않은 아키텍처 설명
- 누락된 환경 변수 설정
- 바뀐 테스트 명령

---

## 4. `#` 단축키 — 세션 중 즉시 반영

Claude Code 세션 중 `#`을 입력하면 Claude가 **현재 세션에서 학습한 내용을 CLAUDE.md에 즉시 반영**한다.

**사용 시나리오**:

```
user: # (방금 알게 된 내용 기록해줘)
claude: 이번 세션에서 발견한 내용을 CLAUDE.md에 추가하겠습니다:
        - pnpm 사용 (npm 아님)
        - 테스트 실행 전 Docker daemon 필요
```

**언제 쓰는가**:
- 세션 중 Claude가 처음에 몰랐지만 이후 학습한 정보
- 다음 세션에도 알아야 할 비자명한 정보
- Observe 단계에서 포착한 반복 실수의 근본 원인

**제한**: `#`은 즉각적이지만 검토 없이 추가된다. 추가 후 CLAUDE.md를 열어 불필요한 내용이 섞이지 않았는지 확인하는 것을 권장한다.

---

## 5. Staleness 감지 기준

규칙을 **언제 삭제**하는가:

| 기준 | 판단 방법 |
|------|---------|
| Claude가 이미 자연스럽게 따른다 | 규칙 없이 새 세션에서 테스트 |
| 코드에서 자동으로 알 수 있다 | 파일 확장자·패키지 구성으로 자명 |
| 해당 라이브러리/패턴이 제거됐다 | `git log` 또는 `grep`으로 참조 여부 확인 |
| 3개월 이상 Claude가 어긴 적이 없다 | 세션 로그 확인 |

**월 1회 리뷰 루틴** (출처: [하네스 엔지니어링 방법론](claude-code-하네스-엔지니어링-방법론.md)):

```bash
# 줄 수 확인
wc -l CLAUDE.md

# 내용 중 삭제 후보 탐색
grep -n "항상\|반드시\|절대" CLAUDE.md
```

200줄 초과 → 즉시 분리 검토. 규칙이 많다고 좋은 게 아니다.

---

## 6. Progressive Disclosure 전략

CLAUDE.md에 모든 정보를 넣으면 컨텍스트가 낭비된다. **필요할 때만 로드**하는 전략:

### 분리 시점 결정 기준

| 조건 | 처리 방법 |
|------|---------|
| 특정 파일 타입 작업 시에만 필요 | `.claude/rules/` 하위에 별도 파일 |
| 특정 도메인 지식 | `@import` 로 별도 파일 분리 |
| 특정 작업 흐름에서만 필요 | Skills로 분리 (`/my-skill` 호출 시에만 로드) |
| 항상 필요한 프로젝트 핵심 | CLAUDE.md 본문에 유지 |

### @import 분리 예시

```markdown
# CLAUDE.md (루트 — 항상 로드)

## 빌드
`pnpm build`

## 테스트
`pnpm test`

@import .claude/rules/database.md    # DB 작업 시 참조
@import .claude/rules/api-design.md  # API 설계 시 참조
```

```markdown
# .claude/rules/database.md
## DB 규칙
- ORM: Prisma (raw SQL 금지)
- Migration: `pnpm prisma migrate dev`
- Index: 외래키에 항상 인덱스 추가
```

**판단 기준**: 세션의 50% 미만에서만 필요한 정보 → 분리 후보.

### Skills로 분리하는 경우

```
/deploy-checklist  → 배포 전 체크리스트 (배포 시에만 필요)
/db-migration      → 마이그레이션 절차 (DB 작업 시에만 필요)
```

CLAUDE.md에 항상 있을 필요가 없는 절차 문서는 Skills로 분리하면 컨텍스트 절약 효과가 크다.

---

## 직접 확인해보기

- [ ] `wc -l CLAUDE.md` — 200줄 초과 여부 확인
- [ ] `/claude-md-improver` 실행 후 등급 확인 (B 미만이면 개선 필요)
- [ ] 마지막으로 규칙을 **삭제**한 것이 1개월 이내인지 확인
- [ ] `#` 단축키로 이번 세션 학습 내용을 기록했는지 확인
- [ ] 200줄 초과 시 `.claude/rules/` 분리 여부 검토

---

## Sources

- [claude-md-improver SKILL.md](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/claude-md-management/skills/claude-md-improver/SKILL.md) — 공식 스킬 원본
- [Claude Code Best Practices](https://code.claude.com/docs/ko/best-practices) — 공식 권장 사항
- [실전 작성법](claude-code-CLAUDE-md-실전-작성법.md) — 최초 작성 방법

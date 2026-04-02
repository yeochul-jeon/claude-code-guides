# CLAUDE.md 실전 작성법: 글로벌 · 프로젝트별 · AGENTS.md (2026)

| 항목 | 날짜 |
|------|------|
| 생성일 | 2026-03-11 |
| 변경일 | 2026-04-02 |

> 글로벌(`~/.claude/CLAUDE.md`) 설정 현황, 프로젝트 유형별(백엔드/프론트엔드/인프라) CLAUDE.md 작성 패턴,
> 그리고 크로스 에이전트 표준인 AGENTS.md까지 정리

### 관련 문서
- [개인 설정 가이드](claude-code-개인설정-가이드.md) — CLAUDE.md 외 전체 설정 항목
- [Harness 추천 구성](claude-code-harness-추천구성.md) — CLAUDE.md 적용 순서 전략

---

## 목차

1. [글로벌 CLAUDE.md 설정 현황](#1-글로벌-claudemd-설정-현황)
   - [v3 최적화 사례](#v3-최적화-사례-7가지-기법)
   - [토큰 관리 전략](#토큰-관리-전략)
2. [AGENTS.md란?](#2-agentsmd란)
3. [백엔드 프로젝트 (Java/Spring)](#3-백엔드-프로젝트-javaspring)
4. [프론트엔드 프로젝트 (React/Next.js)](#4-프론트엔드-프로젝트-reactnextjs)
5. [인프라/DevOps 프로젝트](#5-인프라devops-프로젝트)
6. [모노레포 전략](#6-모노레포-전략)
7. [Addy Osmani의 핵심 주장](#7-addy-osmani의-핵심-주장)
8. [CLAUDE.md 작성 원칙 종합](#8-claudemd-작성-원칙-종합)

---

## 1. 글로벌 CLAUDE.md 설정 현황

`~/.claude/CLAUDE.md`는 **모든 프로젝트에 자동 적용**되는 개인 글로벌 지침이다.

### 개발자들이 공통적으로 포함하는 내용

| 카테고리 | 포함 내용 |
|---------|----------|
| **개발 철학** | 간결함 우선, TDD, 점진적 변경, 기존 패턴 존중 |
| **코드 스타일** | 네이밍 규칙, import 순서, 주석 언어 |
| **커밋 규칙** | 컨벤셔널 커밋, `--no-verify` 금지, 메시지 형식 |
| **도구 선호** | 패키지 매니저 (pnpm/uv/bun), CLI 도구 (gh/glab) |
| **언어 설정** | 응답/주석 언어 (한국어, 일본어 등) |
| **금지 사항** | 처음부터 재구현 금지, 불필요한 추상화 금지 |

### 실전 예시: 일반 풀스택 개발자

```markdown
# ~/.claude/CLAUDE.md

# 개발 철학
- 간결하고 읽기 쉬운 코드를 우선시한다
- 기존 코드 스타일과 패턴을 존중한다
- 시스템을 처음부터 재구현하기 전에 반드시 허락을 구한다
- 불필요한 추상화보다 명확한 구현을 선호한다
- 작은 점진적 변경을 추구한다

# 코드 규칙
- 한국어로 주석 작성
- 모든 파일 첫 줄에 파일 목적을 한 줄 주석으로 기재
- import 순서: 표준 라이브러리 → 프레임워크 → 서드파티 → 로컬
- `any` 타입 사용 금지 (TypeScript)

# 커밋
- 커밋 메시지는 한국어 + 컨벤셔널 커밋 형식
- `--no-verify` 절대 사용 금지
- 관련 없는 변경은 별도 커밋으로 분리

# 도구
- GitHub 작업에는 항상 `gh` CLI 사용
- 의존성 확인 후 적절한 패키지 매니저 사용

# 워크플로우
- 코드 변경 후 타입 체크 실행
- 전체 테스트 스위트 대신 관련 테스트만 실행
- 의심스러운 결정은 먼저 질문하고 가정하지 말 것

# 테스트
- 가능하면 TDD 방식
- 단위 테스트 → 통합 테스트 순서
- 테스트를 건너뛰려면 명시적 허가 필요
```

### 실전 예시: Joe Cotellese 스타일 (시니어 엔지니어)

```markdown
# ~/.claude/CLAUDE.md

# 관계
- 동료 파트너십으로 협업
- 동의하지 않을 때는 기술적 근거와 함께 반론 제시 필수
- 가정하지 말고, 질문할 것

# 핵심 원칙
- Simple over clever
- TDD always
- Ask, don't assume
- Small iterative changes
- Respect existing code patterns

# 코드
- 유지보수성과 가독성 > 영리한 코드
- 기존 코드의 스타일과 포맷을 따를 것
- 주석은 명백히 틀리지 않는 한 보존
- 시간적 이름 금지 ("new", "improved", "v2" 등)

# 커밋
- 커밋 메시지에 Claude를 저자로 표기하지 말 것
- --no-verify 금지
```

### 실전 예시: Freek Van der Herten 스타일 (PHP/Laravel)

```markdown
# ~/.claude/CLAUDE.md

# 태도
- 비판적이고 솔직하게. 아첨하지 말 것
- Spatie PHP 가이드라인 준수
- GitHub 작업에는 항상 gh CLI 사용
```

> **공통 패턴:** 가장 효과적인 글로벌 CLAUDE.md는 **20~50줄** 수준으로 매우 짧고, 프로젝트 공통 철학과 금지사항에만 집중한다.

### v3 최적화 사례: 7가지 기법

실제 글로벌 CLAUDE.md를 **109줄 → 90줄 (17% 감소)**로 최적화한 사례. 줄 수 절감보다 **토큰 효율과 지시 준수율 향상**이 핵심이다.

| # | 기법 | Before | After | 효과 |
|---|------|--------|-------|------|
| 1 | 이모지 제거 | 🔧, 📝 등 9개 | 텍스트만 | ~20 토큰/세션 절약 |
| 2 | 자명한 규칙 삭제 | "Favor readability", "Follow existing code style" | 삭제 | 에이전트가 이미 아는 정보 제거 |
| 3 | 부정형→긍정형 전환 | "No `any` type" | "Use `unknown` and narrow types" | 위반율 ~50% 감소 |
| 4 | 소규모 섹션 통합 | Code Rules + Tools (각 3줄) | "Code & Tools" 1개 섹션 | 헤딩 오버헤드 절약 |
| 5 | Primacy/Recency 앵커링 | 중요 규칙이 중간에 배치 | 문서 최상단+최하단에 IMPORTANT 배치 | 핵심 규칙 준수율 향상 |
| 6 | 응답 길이 제어 | "Be concise" (모호) | "Keep non-code responses under 20 lines" | 구체적 기준으로 일관성 확보 |
| 7 | Skills로 on-demand 분리 | Serena MCP 지침 인라인 (15줄) | `~/.claude/skills/serena/SKILL.md`로 분리 | 미사용 프로젝트에서 토큰 절약 |

> **원칙**: 시스템 프롬프트에 이미 포함된 규칙(예: `--no-verify` 금지)은 CLAUDE.md에서 제거한다.

### 토큰 관리 전략

한국어는 영어 대비 **2~3배 토큰**을 소모한다. 세션 전체 기준으로는 코드가 50~70%이므로 실질 차이는 30~50%이지만, 컨텍스트 한도에 직접 영향을 준다.

**줄 수 관리**

- 글로벌 CLAUDE.md: **200줄 이하** 유지 (공식 권장)
- 프로젝트 CLAUDE.md: 200줄 이하, 넘으면 아래 방법으로 분리

**`@path` import로 분리**

```markdown
# CLAUDE.md
@docs/api-conventions.md
@docs/testing-guide.md
@~/.claude/my-preferences.md
```

- 상대 경로는 해당 파일 기준으로 resolve
- 최대 **5 depth** 재귀 import 지원
- AGENTS.md 호환: `@AGENTS.md`로 import 후 Claude 전용 규칙 추가

**`.claude/rules/`로 경로별 규칙 분리**

특정 파일 패턴에만 적용되는 규칙은 `.claude/rules/`에 분리:

```markdown
# .claude/rules/api-design.md
---
paths:
  - "src/api/**/*.ts"
---

# API 개발 규칙
- 모든 엔드포인트에 입력 검증 포함
- 표준 에러 응답 포맷 사용
- OpenAPI 문서 주석 포함
```

패턴 예시: `**/*.ts`, `src/**/*`, `*.md`, `src/components/*.tsx`, `src/**/*.{ts,tsx}`

**Auto Memory 시스템**

Claude Code는 `~/.claude/projects/<project>/memory/`에 대화에서 학습한 내용을 자동 저장한다. `MEMORY.md` 첫 200줄이 세션 시작 시 로딩된다.

- 기본 활성화 (비활성화: `autoMemoryEnabled: false`)
- `/memory` 명령으로 조회/편집 가능
- CLAUDE.md와 보완적: CLAUDE.md는 명시적 규칙, Memory는 학습된 선호

---

## 2. AGENTS.md란?

### 개요

**AGENTS.md**는 AI 코딩 에이전트를 위한 **벤더 중립적 오픈 표준**이다. CLAUDE.md가 Claude Code 전용이라면, AGENTS.md는 **여러 AI 도구(Cursor, Windsurf, Codex, Kilo Code 등)**에서 공통으로 읽을 수 있는 표준 형식이다.

| | CLAUDE.md | AGENTS.md |
|-|-----------|-----------|
| **대상 도구** | Claude Code 전용 | 다중 AI 코딩 에이전트 공통 |
| **표준화** | Anthropic 자체 형식 | 오픈 표준 (agents.md) |
| **위치** | 프로젝트 루트 또는 `~/.claude/` | 프로젝트 루트 |
| **지원 도구** | Claude Code | Cursor, Windsurf, Codex, Kilo Code 등 |
| **특징** | Claude 전용 기능 (skills, hooks 연동) | 벤더 중립, 상호 운용성 |

### AGENTS.md 핵심 구조

```markdown
# AGENTS.md

## Build & Test
- `npm run build` — 빌드
- `npm run test` — 전체 테스트
- `npm run test -- --filter=auth` — 인증 모듈 테스트만

## Code Conventions
- TypeScript strict mode 사용
- import 순서: React → Next → 서드파티 → 로컬

## Architecture
- src/api/ — API 라우트
- src/lib/ — 공유 유틸리티
- 새 API 엔드포인트는 반드시 src/api/ 하위에 생성

## Gotchas
- `legacy-auth` 모듈은 deprecated지만 아직 import됨. 제거 금지
- `.env.local`은 테스트 환경에서 무시됨
```

### 함께 사용하는 전략

```
프로젝트/
├── AGENTS.md          # 모든 AI 도구 공통 (빌드, 테스트, 아키텍처)
├── CLAUDE.md          # Claude Code 전용 (skills 연동, hooks 참조)
└── .cursorrules       # Cursor 전용 (있다면)
```

Claude Code는 **CLAUDE.md와 AGENTS.md를 모두 읽는다.** 중복을 피하려면:
- `AGENTS.md`: 빌드 명령어, 아키텍처, 코드 컨벤션 (범용)
- `CLAUDE.md`: Claude 전용 지시사항, skills/hooks 참조, Claude 특화 워크플로우

### 실전 예시: Full-Stack 웹앱의 AGENTS.md

```markdown
# AGENTS.md

## Build & Test
- `pnpm dev` — 개발 서버 (Turbopack)
- `pnpm build` — 프로덕션 빌드
- `pnpm test` — 전체 테스트
- `pnpm test -- --filter=auth` — 인증 모듈만

## Code Conventions
- TypeScript strict mode
- import 순서: React → Next → 서드파티 → 로컬
- 들여쓰기 2칸, 세미콜론 없음

## Architecture
- src/app/ — App Router 페이지/레이아웃
- src/lib/ — 공유 유틸리티, API 래퍼
- src/components/ — UI 컴포넌트 (shadcn/ui 기반)

## Gotchas
- `legacy-auth` 모듈은 deprecated이지만 3곳에서 import. 제거 금지
- `.env.local`은 테스트 환경에서 무시됨 — 테스트용 환경변수는 `.env.test`에
- `pnpm build` 전에 `pnpm generate:types` 필수 (GraphQL codegen)
```

### 실전 예시: 같은 프로젝트의 CLAUDE.md (AGENTS.md와 병행)

```markdown
# CLAUDE.md

# 범용 규칙은 AGENTS.md 참조
@AGENTS.md

# Claude 전용 설정
## 언어
- 설명과 주석: 한국어
- 커밋 메시지: 한국어 + Conventional Commits

## Skills 연동
- /simplify — 코드 변경 후 품질 리뷰
- /batch — 대규모 리팩토링 시 병렬 처리

## 워크플로우
- 코드 변경 전 Plan Mode로 계획 수립
- 파일 편집 후 자동 포맷팅 (PostToolUse Hook)
```

> **핵심**: AGENTS.md에 빌드/아키텍처/컨벤션을, CLAUDE.md에 Claude 전용 기능을 분리하면 **다중 AI 도구 환경**에서도 일관성을 유지할 수 있다 (Addy Osmani).

---

## 3. 백엔드 프로젝트 (Java/Spring)

### 프로젝트 CLAUDE.md 예시

```markdown
# CLAUDE.md — Spring Boot API 서버

# 프로젝트 개요
Spring Boot 3.x + Java 21 기반 REST API 서버
QueryDSL + JPA 사용, PostgreSQL

# 빌드 & 테스트
- 빌드: `./gradlew build`
- 테스트: `./gradlew test`
- 단일 테스트: `./gradlew test --tests "com.example.SomeTest"`
- 린트: `./gradlew checkstyleMain`

# 아키텍처 (레이어드)
- Controller → Service → Repository 흐름 엄수
- Entity를 Controller에서 직접 반환 금지, 반드시 DTO 변환
- DI는 생성자 주입(@RequiredArgsConstructor) 필수
- 필드 주입(@Autowired) 금지

# JPA 규칙
- Entity에 @Data 금지 → @Getter + @NoArgsConstructor(access = PROTECTED)
- N+1 방지: Fetch Join, @EntityGraph, BatchSize 고려
- 수정 시 save() 대신 Dirty Checking 활용

# QueryDSL
- 동적 쿼리는 QueryDSL 최우선 사용
- BooleanBuilder 대신 BooleanExpression 반환 메서드 추출
- RepositoryCustom + RepositoryImpl 패턴 준수

# API 컨벤션
- URL: kebab-case (/api/v1/user-profiles)
- JSON: camelCase
- 에러 응답: { "code": "...", "message": "..." }
- 목록 API는 페이지네이션 필수

# 예외 처리
- @RestControllerAdvice로 전역 처리
- try-catch 남용 금지

# 테스트
- JUnit 5 + AssertJ 기본
- Service: Mockito 단위 테스트
- Repository: @DataJpaTest 슬라이스 테스트
- Controller: @WebMvcTest

# Java 스타일
- record 적극 활용 (DTO, 불변 데이터)
- switch expression 사용
- Optional<T> 선호, null 반환 지양
- var는 지역 변수에 한해 허용
```

### 모듈별 CLAUDE.md (하위 디렉토리)

```markdown
# backend/auth/CLAUDE.md

# 인증 모듈
- JWT + Spring Security 6 기반
- 토큰 저장: httpOnly 쿠키
- Refresh Token Rotation 적용
- 테스트 시 @WithMockUser 활용
- SecurityConfig.java 수정 시 반드시 통합 테스트 실행
```

---

## 4. 프론트엔드 프로젝트 (React/Next.js)

### 프로젝트 CLAUDE.md 예시

```markdown
# CLAUDE.md — Next.js 웹 앱

# 프로젝트 개요
Next.js 15 (App Router) + TypeScript + Tailwind CSS + shadcn/ui

# 빌드 & 테스트
- `pnpm dev` — 개발 서버
- `pnpm build` — 프로덕션 빌드
- `pnpm lint` — ESLint
- `pnpm test` — Jest + React Testing Library

# 디렉토리 구조
- app/          — App Router 페이지/레이아웃
- components/   — UI 컴포넌트 (shadcn 기반)
- hooks/        — 커스텀 React hooks
- lib/          — API 래퍼, 유틸리티
- styles/       — Tailwind 커스터마이징

# 컴포넌트 규칙
- shadcn/ui를 기본 UI 라이브러리로 사용
- Tailwind 유틸리티 클래스로 스타일링
- 컴포넌트당 단일 책임
- 상속보다 합성 선호
- 재사용 로직은 커스텀 hook으로 추출

# TypeScript
- strict mode 필수
- any 타입 금지, unknown 또는 제네릭 사용
- 반환 타입 명시
- props는 항상 구조 분해

# React Query (TanStack)
- QueryClient는 app/layout.tsx에서 초기화
- API 로직은 lib/api/에 정리, hooks로 호출
- 쿼리 키: 도메인 접두사 ['user', id]

# import 순서
React → Next → 서드파티 → 로컬 모듈

# 테스트
- @testing-library/react 사용
- MSW(Mock Service Worker)로 API 모킹
- 컴포넌트 테스트는 컴포넌트와 같은 디렉토리에 배치

# 보안
- API 라우트에서 서버사이드 입력 검증
- DOMPurify로 XSS 방지
- 환경 변수는 NEXT_PUBLIC_ 접두사로 공개 범위 구분

# 성능
- React.memo, useMemo, useCallback 적절히 사용
- 이미지: next/image 사용 필수
- 코드 스플리팅: dynamic import 활용
```

---

## 5. 인프라/DevOps 프로젝트

### 프로젝트 CLAUDE.md 예시

```markdown
# CLAUDE.md — 인프라 프로비저닝

# 프로젝트 개요
AWS 기반 쿠버네티스 클러스터 자동 프로비저닝
Terraform + Helm + ArgoCD + GitHub Actions

# 도구 & 명령어
- `terraform init` — 초기화
- `terraform plan -var-file=env/dev.tfvars` — 변경 계획
- `terraform apply` — 적용 (반드시 plan 확인 후)
- `helm upgrade --install <release> <chart>` — Helm 배포
- `kubectl get pods -n <namespace>` — 상태 확인

# 디렉토리 구조
- modules/         — 재사용 Terraform 모듈
- env/             — 환경별 변수 파일 (dev, staging, prod)
- charts/          — Helm 차트
- manifests/       — Kubernetes YAML
- scripts/         — 자동화 스크립트
- .github/workflows/ — CI/CD 파이프라인

# Terraform 규칙
- 모듈 단위로 리소스 구성
- 변수는 variables.tf, 출력은 outputs.tf에 정의
- 상태 파일은 원격 백엔드(S3) 사용
- `terraform destroy`는 절대 사용자 확인 없이 실행 금지
- 리소스 이름: kebab-case + 환경 접두사 (dev-api-server)

# 보안 (매우 중요)
- 시크릿은 AWS Secrets Manager 또는 Vault 사용
- .tfvars에 시크릿 포함 금지
- IAM 정책: 최소 권한 원칙
- 보안 그룹: 0.0.0.0/0 인바운드 금지

# 네이밍
- Terraform 리소스: snake_case
- Kubernetes 리소스: kebab-case
- 환경 변수: UPPER_SNAKE_CASE

# Gotchas
- prod 환경 변경은 반드시 PR + 리뷰 필수
- EKS 노드 그룹 변경 시 rolling update 확인
- RDS 파라미터 그룹 변경은 재부팅 필요할 수 있음
```

---

## 6. 모노레포 전략

### 디렉토리 구조

```
monorepo/
├── CLAUDE.md                    # 공통 규칙 (빌드, 커밋, 전체 아키텍처)
├── AGENTS.md                    # 크로스 에이전트 공통 (선택)
├── backend/
│   ├── CLAUDE.md                # Spring Boot 규칙
│   └── auth/
│       └── CLAUDE.md            # 인증 모듈 전용
├── frontend/
│   ├── CLAUDE.md                # Next.js 규칙
│   └── components/
│       └── CLAUDE.md            # 컴포넌트 컨벤션
├── infra/
│   └── CLAUDE.md                # Terraform/K8s 규칙
└── .claude/
    ├── settings.json            # 팀 공용 권한
    ├── skills/
    │   ├── api-design/SKILL.md
    │   └── db-migration/SKILL.md
    └── agents/
        ├── backend-reviewer.md
        └── frontend-reviewer.md
```

### 로딩 규칙

| 작업 위치 | 로드되는 CLAUDE.md |
|----------|-------------------|
| `backend/auth/UserService.java` | 루트 + `backend/` + `backend/auth/` |
| `frontend/components/Button.tsx` | 루트 + `frontend/` + `frontend/components/` |
| `infra/modules/eks/main.tf` | 루트 + `infra/` |

> 상위 디렉토리는 **항상 자동 로드**, 하위 디렉토리는 **해당 파일 작업 시에만** 로드.

### 루트 CLAUDE.md (모노레포 공통)

```markdown
# CLAUDE.md — 모노레포 루트

# 구조
- backend/ — Spring Boot 3.x API 서버 (Java 21)
- frontend/ — Next.js 15 웹 앱 (TypeScript)
- infra/ — Terraform + Kubernetes 인프라
- shared/ — 공유 타입, 유틸리티

# 공통 규칙
- 커밋: 컨벤셔널 커밋 (feat:, fix:, infra:, docs:)
- PR: 각 영역(backend/frontend/infra) 별도 PR 권장
- 크로스 영역 변경 시 관련 모든 테스트 실행

# 빌드
- backend: `./gradlew build` (backend/ 디렉토리에서)
- frontend: `pnpm build` (frontend/ 디렉토리에서)
- infra: `terraform plan` (infra/ 디렉토리에서)

# 모듈별 상세 규칙은 각 디렉토리의 CLAUDE.md 참조
```

### @import로 모듈별 규칙 연결

루트 CLAUDE.md에서 각 모듈의 규칙을 import하면, 해당 모듈 작업 시 자동으로 로딩된다:

```markdown
# CLAUDE.md (모노레포 루트)
@backend/CLAUDE.md
@frontend/CLAUDE.md
@infra/CLAUDE.md
@shared/CLAUDE.md
```

### .claude/rules/로 파일 패턴별 규칙

모노레포에서 특히 유용하다. 디렉토리별 CLAUDE.md 대신 **파일 패턴 기반** 규칙을 적용:

```markdown
# .claude/rules/backend-api.md
---
paths:
  - "backend/src/api/**/*.java"
---
- Controller → Service → Repository 흐름 엄수
- Entity를 Controller에서 직접 반환 금지
```

```markdown
# .claude/rules/frontend-components.md
---
paths:
  - "frontend/src/components/**/*.tsx"
---
- shadcn/ui를 기본 UI 라이브러리로 사용
- 컴포넌트당 단일 책임
- Props는 항상 구조 분해
```

### 도메인 지식 명시 패턴

에이전트가 코드만으로는 알 수 없는 **비즈니스 도메인 용어와 규칙**을 명시한다:

```markdown
# backend/CLAUDE.md

## 도메인 용어
- SKU (Stock Keeping Unit): 재고 관리 단위
- PDP (Product Detail Page): 상품 상세 페이지
- CAC (Customer Acquisition Cost): 고객 획득 비용

## 비즈니스 규칙
- 재고 5개 이하 → "품절 임박" 표시
- 신규 회원 첫 구매 시 10% 할인 자동 적용
- 50,000원 이상 구매 시 무료 배송
```

> 이 패턴은 Addy Osmani의 "에이전트가 발견할 수 없는 정보만 포함" 원칙에 정확히 부합한다.

---

## 7. Addy Osmani의 핵심 주장

Addy Osmani (Google Chrome 팀 엔지니어링 매니저)의 2026년 2월 글 **"Stop Using /init for AGENTS.md"**의 핵심:

### 연구 결과 (2026)

| 연구 | 결과 |
|------|------|
| **Lulla et al. (ICSE JAWs)** | *수동 작성* AGENTS.md → 런타임 28.64% 감소, 토큰 16.58% 절약 |
| **ETH Zurich** | *자동 생성* 컨텍스트 파일 → 성공률 2~3% 감소, 비용 20%+ 증가 |

### 핵심 메시지

> **"에이전트에게는 사무실 투어가 아니라 지뢰밭 경고가 필요하다."**

- `/init`이 자동 생성하는 내용은 대부분 **에이전트가 스스로 발견할 수 있는 정보**
- 자동 생성 CLAUDE.md는 **중복**이지, 쓸모없는 게 아님. 문제는 **비용과 컨텍스트 희석**
- 에이전트가 스스로 알 수 없는 것만 넣어야 함

### 포함해야 할 것 (MUST)

| 항목 | 예시 |
|------|------|
| 패키지 매니저 지정 | `uv`를 사용하라, `pip` 아님 |
| 테스트의 함정 | `npm test` 전에 반드시 `npm run build` 필요 |
| Deprecated 모듈 | `legacy-auth`는 deprecated이지만 아직 import됨. 제거 금지 |
| 비표준 패턴 | 이 프로젝트의 에러 핸들링은 X 패턴을 따름 |
| 빌드 quirks | macOS에서는 `XYZ=1` 환경변수 필요 |

### 제외해야 할 것 (MUST NOT)

| 항목 | 이유 |
|------|------|
| 디렉토리 구조 설명 | 코드 탐색으로 발견 가능 |
| 기술 스택 개요 | package.json/build.gradle에서 추론 가능 |
| 표준 컨벤션 | 에이전트가 이미 알고 있음 |
| 코드 스니펫 | 금방 outdated됨 |

### 구조적 개선안

```
Layer 1: 최소한의 라우팅 문서 (발견 불가능한 사실만)
Layer 2: 태스크 유형별 페르소나/스킬 파일 (선택적 로드)
Layer 3: 문서 유지보수 서브에이전트 (자동 갱신)
```

→ Claude Code의 **Skills 시스템**이 정확히 이 Layer 2에 해당한다.

---

## 8. CLAUDE.md 작성 원칙 종합

### 진단 도구로서의 CLAUDE.md

> Claude가 반복적으로 실수하는 것 → CLAUDE.md에 규칙 추가
> Claude가 이미 잘 하는 것 → CLAUDE.md에서 삭제
> Claude가 지시를 무시하기 시작 → CLAUDE.md가 너무 길다는 신호

### 체크리스트

| 질문 | 행동 |
|------|------|
| 이 줄을 삭제하면 Claude가 실수할까? | 아니면 삭제 |
| 코드를 보면 알 수 있는 정보인가? | 그렇다면 삭제 |
| 모든 세션에 필요한 정보인가? | 아니면 Skills로 이동 |
| 매번 100% 실행되어야 하는가? | 그렇다면 Hooks로 이동 |
| 다른 AI 도구에서도 필요한가? | 그렇다면 AGENTS.md에 작성 |
| 200줄을 넘는가? | Skills, rules/, @import로 분리 |

### CLAUDE.md vs Hook: 어디에 넣을 것인가?

> **"CLAUDE.md는 advisory (80% 준수), Hook은 deterministic (100% 실행)"** — Builder.io

| 요구사항 | 배치 | 이유 |
|---------|------|------|
| 코드 스타일 가이드라인 | CLAUDE.md | 안내 수준, 상황에 따라 유연 |
| 파일 편집 후 자동 포맷팅 | Hook (PostToolUse) | 매번 100% 실행 필수 |
| 특정 파일(.env 등) 수정 차단 | Hook (PreToolUse) | 실패 시 피해 큼, 100% 차단 필요 |
| 파괴적 명령(rm -rf) 차단 | Hook (PreToolUse) | 보안상 100% 차단 |
| 에이전트 합리화 방지 | Hook (Stop) | "pre-existing issue", "out of scope" 등 변명 차단 |
| 커밋 메시지 형식 | CLAUDE.md + Hook | CLAUDE.md로 안내, Hook으로 검증 |
| 도메인 용어 | CLAUDE.md | 지식 전달 목적, 강제 불필요 |
| Bash 명령 감사 로그 | Hook (PostToolUse) | 모든 명령을 기록해야 하므로 Hook |

### 최종 공식

```
CLAUDE.md = 에이전트가 발견할 수 없는 것 + 반복 실수를 방지하는 규칙
          = 지뢰밭 경고 (NOT 사무실 투어)

Hook     = 반드시 100% 실행해야 하는 자동화 규칙
          = 가드레일 (NOT 가이드라인)
```

---

## 직접 확인해보기

- [ ] 현재 프로젝트의 CLAUDE.md가 200줄 이하인지 확인 (`wc -l CLAUDE.md`)
- [ ] "지뢰밭 경고"만 포함되어 있는지 검토 (코드에서 자동 발견 가능한 내용 제거)
- [ ] 빌드/테스트 명령이 정확히 기재되어 있는지 실제 실행하여 검증
- [ ] 모노레포라면 서브디렉토리별 CLAUDE.md가 분리되어 있는지 확인
- [ ] `claude`로 진입 후 "이 프로젝트의 규칙을 설명해줘"로 CLAUDE.md 인식 확인

---

## Sources

### 공식 문서
- [Claude Code — Memory (CLAUDE.md)](https://code.claude.com/docs/en/memory)
- [Claude Code — Skills](https://code.claude.com/docs/en/skills)
- [Claude Code — Hooks](https://code.claude.com/docs/en/hooks)
- [Agent Skills 오픈 표준](https://agentskills.io)

### CLAUDE.md 작성법
- [Addy Osmani - Stop Using /init for AGENTS.md](https://addyosmani.com/blog/agents-md/)
- [Builder.io - How to Write a Good CLAUDE.md](https://www.builder.io/blog/claude-md-guide)
- [HumanLayer - Writing a Good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [Dometrain - Creating the Perfect CLAUDE.md](https://dometrain.com/blog/creating-the-perfect-claudemd-for-claude-code/)
- [Nick Babich - CLAUDE.md Best Practices (UX Planet)](https://uxplanet.org/claude-md-best-practices-1ef4f861ce7c)

### AGENTS.md / 크로스 에이전트
- [AGENTS.md GitHub 리포지토리](https://github.com/agentsmd/agents.md)
- [OpenAI Codex - Custom Instructions with AGENTS.md](https://developers.openai.com/codex/guides/agents-md/)

### 템플릿 & 예시
- [Spring AI Examples CLAUDE.md](https://github.com/spring-projects/spring-ai-examples/blob/main/CLAUDE.md)
- [Spring Boot CLAUDE.md Template (ruvnet)](https://github.com/ruvnet/ruflo/wiki/CLAUDE-MD-Java)
- [React CLAUDE.md Template (ruvnet)](https://github.com/ruvnet/ruflo/wiki/CLAUDE-MD-React)
- [DevOps CLAUDE.md Template](https://github.com/ruvnet/claude-flow/wiki/CLAUDE-MD-DevOps)
- [Next.js + TypeScript + Tailwind CLAUDE.md (Gist)](https://gist.github.com/gregsantos/2fc7d7551631b809efa18a0bc4debd2a)
- [Trail of Bits — CLAUDE.md 템플릿](https://github.com/trailofbits/claude-code-config)
- [황민호 — claude-code-mastering](https://github.com/revfactory/claude-code-mastering)

### 개인 설정 사례
- [Joe Cotellese - Claude Code Global Configuration](https://joecotellese.com/posts/claude-code-global-configuration/)
- [Freek Van der Herten - My Claude Code Setup](https://freek.dev/3026-my-claude-code-setup)
- [shanraisshan - claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)

### 인프라/DevOps
- [VoltAgent - Terraform Engineer Subagent](https://github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/03-infrastructure/terraform-engineer.md)
- [DevOps Claude Skills](https://github.com/ahmedasmar/devops-claude-skills)
- [Mastering CLAUDE.md for DevOps](https://devops-geek.net/devops-lab/mastering-claude-md-the-devops-guide-to-ai-assisted-development/)

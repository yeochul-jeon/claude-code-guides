# Anthropic 공식 문서 원본 정리 (2026-06-02)

> **면책**: 이 문서는 Anthropic 공식 문서의 **WebSearch 기반 요약·대조**이며, 1차 출처는 각 절의 URL입니다.  
> `platform.claude.com` 문서는 JS 렌더링으로 직접 열람 불가 → WebSearch + 3자 미러 교차 검증. **미확정 항목은 명시됩니다.**  
> 이 문서는 `sources/원본자료-갭분석-20260601.md` §8 갭 백로그(P0-4, P1-W\*)의 **1차 출처 근거**용입니다.

| 항목 | 내용 |
|------|------|
| 생성일 | 2026-06-02 |
| 갱신일 | 2026-06-02 |
| 작성자 | 전여철 + Claude Sonnet 4.6 |
| 조사 방법 | WebSearch (에이전트 5개 병렬) + 3자 미러 교차 검증 |
| 조사 범위 | Anthropic 공식 문서 10개 (PyTorchKR 재구성본 인용 출처) |
| 선행 분석 | `sources/원본자료-갭분석-20260601.md` §8 참조 |

---

## 목차

1. [조사 대상·방법](#1-조사-대상방법)
2. [Prompt Engineering Overview](#2-prompt-engineering-overview)
3. [Models Overview](#3-models-overview)
4. [Migration Guide](#4-migration-guide)
5. [What's new in Claude Opus 4.8](#5-whats-new-in-claude-opus-48)
6. [Effort Parameter](#6-effort-parameter)
7. [Adaptive Thinking](#7-adaptive-thinking)
8. [Extended Thinking](#8-extended-thinking)
9. [Structured Outputs](#9-structured-outputs)
10. [Context Windows & Awareness](#10-context-windows--awareness)
11. [Agent Skills Overview](#11-agent-skills-overview)
12. [원본 ↔ PyTorchKR 재구성본 대조표](#12-원본--pytorchkr-재구성본-대조표)
13. [갭 백로그 §8 1차 출처 매핑](#13-갭-백로그-8-1차-출처-매핑)
14. [미확정·후속 확인 항목](#14-미확정후속-확인-항목)

---

## 1. 조사 대상·방법

### 조사 대상 10개

| # | 주제 | 공식 URL |
|---|------|---------|
| 1 | Prompt Engineering Overview | `platform.claude.com/docs/en/build-with-claude/prompt-engineering` |
| 2 | Models Overview | `platform.claude.com/docs/en/about-claude/models/overview` |
| 3 | Migration Guide | `platform.claude.com/docs/en/about-claude/models/migration-guide` |
| 4 | What's new in Claude Opus 4.8 | `platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-8` |
| 5 | Effort Parameter | `platform.claude.com/docs/en/build-with-claude/effort` |
| 6 | Adaptive Thinking | `platform.claude.com/docs/en/build-with-claude/adaptive-thinking` |
| 7 | Extended Thinking | `platform.claude.com/docs/en/build-with-claude/extended-thinking` |
| 8 | Structured Outputs | `platform.claude.com/docs/en/build-with-claude/structured-outputs` |
| 9 | Context Windows & Awareness | `platform.claude.com/docs/en/build-with-claude/context-windows` |
| 10 | Agent Skills Overview | `platform.claude.com/docs/en/agents-and-tools/agent-skills/overview` |

### WebSearch 제약 및 신뢰도

- `platform.claude.com`은 JS 렌더링 → WebFetch는 "Loading…"/404만 반환 (프로브 확인)
- `docs.anthropic.com`은 platform.claude.com으로 301 리다이렉트
- **주 도구**: WebSearch (페이지 내용 요약 반환 확인됨)
- **보조**: `anthropic.com/news`, `anthropic.com/engineering`, `github.com/anthropics` 직접 접근 가능 (1차)
- **3자 미러**: OpenRouter migration cookbook, AI Codex, DEV Community 등 — 1차/3자 명시 구분

---

## 2. Prompt Engineering Overview

**1차 출처**: `docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview`

### 원본 핵심

하위 기법 8가지 (Overview가 인덱스 역할, 각 기법은 별도 서브페이지):

| # | 기법 | 요약 |
|---|------|------|
| 1 | Be clear and direct | 범위·지역·기간 등 세부 파라미터를 명시. "모호함이 있으면 Claude도 혼동" |
| 2 | Use examples (Multishot prompting) | 원하는 결과 예시 2개 이상 제공. `<example>` 태그 구조화 권장 |
| 3 | Give Claude a role (System prompts) | 페르소나·역할 부여로 컨텍스트 설정. 한 문장도 효과적 |
| 4 | Chain of thought (CoT) | "단계적으로 생각하게" 요청. 복잡 작업은 Prompt Chaining으로 확장 |
| 5 | XML tags | `<instructions>`, `<data>`, `<examples>`, `<thinking>/<answer>` 태그로 구조화. Claude는 XML 인식 특화 훈련됨 |
| 6 | Long context tips | 긴 데이터(~20K+ 토큰)는 쿼리/지시 위에 먼저 배치. 문서를 쿼리 아래 두면 품질 최대 30% 하락 |
| 7 | Prompt chaining | 복잡 작업을 서브태스크로 분해하여 순차 처리 |
| 8 | Extended thinking | Multishot + CoT 조합으로 심화 추론 (현재 Adaptive Thinking으로 대체 경로 진행 중) |

- **Claude 4 전용 모범 사례**: 별도 페이지 `claude-4-best-practices`로 분리 (2026년 이후)
- **Console 도구**: Prompt generator, Templates & variables, Prompt improver 제공

### PyTorchKR 재구성본과의 차이

- 원본: Long context 데이터를 프롬프트 **상단**에 배치 시 품질 최대 **30% 하락 방지** 표현
- 재구성본: "30% 개선"으로 긍정형 전환 — 의미는 같으나 표현 방향이 반대. **원본 표현이 더 명확**
- 원본: `claude-4-best-practices` 페이지가 별도 존재 — 재구성본에서 통합하여 언급하지 않음

---

## 3. Models Overview

**1차 출처**: `docs.anthropic.com/en/docs/about-claude/models/overview`

### 원본 핵심: 현재 지원 모델 (2026-06-02 기준)

| 모델명 | API 문자열 | 특징 | 상태 |
|--------|-----------|------|------|
| Claude Opus 4.8 | `claude-opus-4-8` | 최고 성능. 1M 컨텍스트. Dynamic Workflows. 2026-05-28 출시 | ✅ 현재 |
| Claude Opus 4.7 | `claude-opus-4-7` | 새 토크나이저 (입력 ~35% 토큰 증가). 2026-04-16 출시 | ✅ 현재 |
| Claude Opus 4.6 | `claude-opus-4-6` | Desktop에서 제거됨, API는 유지 | ✅ API만 |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 속도·지능 균형. **공식 "recommended" 표기** | ✅ 현재 |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | 가장 빠르고 저렴. 단순 작업용 | ✅ 현재 |

- **API 문자열 규칙**: Claude 4.6 세대부터 날짜 접미사 없는 dateless ID 표준. Haiku 4.5는 날짜 접미사가 canonical일 수 있음 (미확정)
- **Deprecated 예정**: `claude-sonnet-4-20250514`, `claude-opus-4-20250514` → 2026-06-15 retirement
- **최신 권장**: Opus 4.8 (최고 성능) / Sonnet 4.6 (균형, recommended)
- 모든 현재 모델: 텍스트·이미지 입력, Vision 지원, 다국어

### PyTorchKR 재구성본과의 차이

- 원본: Deprecated 일정 (`claude-sonnet-4-20250514` 등, 2026-06-15) 명시 — 재구성본에서 생략됨
- 원본: Haiku 4.5 API 문자열에 날짜 접미사 (`-20251001`) 포함 가능성 — 재구성본에서 단순 `claude-haiku-4-5`로 표기

---

## 4. Migration Guide

**1차 출처**: `platform.claude.com/docs/en/about-claude/models/migration-guide`  
**교차 검증**: AI Codex, LaoZhang AI Blog, Agentpedia (3자)

### 원본 핵심: 버전별 브레이킹 체인지

#### Claude 4.7+ 브레이킹 체인지

| 변경 | 영향 모델 | 오류 | 대체 |
|------|---------|------|------|
| 프리필(Prefill) 종료 | Opus 4.8/4.7/4.6, Sonnet 4.6, Mythos Preview | `400 invalid_request_error` — `"Prefilling assistant messages is not supported for this model."` | Structured Outputs (`output_config.format`), 시스템 프롬프트, continuation → user 메시지로 이동 |
| `budget_tokens` 제거 | Opus 4.7+ | `400` — `"thinking.type.enabled is not supported..."` | `thinking: {type: "adaptive"}` + `output_config: {effort: "..."}` |
| `temperature`/`top_p`/`top_k` 비기본값 금지 | Opus 4.7+ | `400` | 파라미터를 요청에서 완전 제거 |
| 토크나이저 변경 | Opus 4.7 | 토큰 소비량 증가 (비용 영향) | 토큰 카운팅 로직 재검증 |

#### Opus 4.7 토크나이저 변경

- 새 토크나이저 도입 → 입력 토큰 소비량 4.6 대비 **1.0~1.35배** 증가 (콘텐츠 유형에 따라 다름)
- 한국어·CJK 문자 등 비영어 콘텐츠에서 증가 폭 더 큼

#### Opus 4.8 → 4.7 대비 변경사항

- **API 브레이킹 체인지 없음** (4.7과 동일)
- 프롬프트 캐싱 최소 길이: **1,024 tokens** (4.7보다 낮음) → 코드 변경 없이 더 짧은 프롬프트도 캐싱

### 프리필 종료 사유 (공식 3가지)

1. 유효한 JSON 보장 불가 (generation trick이었음)
2. 컨텍스트 토큰 소비 불명확 (attributable하지 않음)
3. Structured Outputs 등 신기능과 충돌

### PyTorchKR 재구성본과의 차이

- 원본: `temperature`/`top_p`/`top_k` 비기본값도 Opus 4.7+ 에서 400 — 재구성본에서 완전 생략됨 (**중요한 누락**)
- 원본: 토크나이저 변경 1.0~1.35배 출처 = Migration Guide — 재구성본에서 "Anthropic 공식 효율 계수"로 다른 맥락에서 언급

---

## 5. What's new in Claude Opus 4.8

**1차 출처**: `platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-8` + `anthropic.com/news/claude-opus-4-8` (직접 접근 확인)

### 원본 핵심

#### Dynamic Workflows (연구 미리보기)

- Claude Code에서 수백 개의 병렬 서브에이전트를 단일 세션에서 실행
- 대규모 코드베이스 마이그레이션(수십만 라인)을 킥오프~머지까지 자동화
- 가용 플랜: Enterprise, Team, Max
- 베타 피드백 기반 정식 출시 예정

#### Fast Mode (API 연구 미리보기)

- API 파라미터: `speed: "fast"`
- 동일 모델 대비 출력 토큰 속도 **최대 2.5배** 향상
- 이전 모델 fast mode 대비 **3배 저렴**
- 프리미엄 가격 적용 (정확한 배수 미확인)

#### Messages API 신규: Mid-conversation System Role

```python
# Opus 4.8 전용: 대화 중간에 system 지시 추가
messages = [
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."},
    {"role": "system", "content": "Apply new instructions here"},  # 신규
    {"role": "user", "content": "..."}
]
```

- 이전 모델(4.7 포함)에서는 동일 방식 시 `400` 오류
- 긴 대화 중 전체 시스템 프롬프트 재전송 없이 지시사항 업데이트 가능

#### 품질 개선

- 코드 결함 미보고 비율: Opus 4.7 대비 **약 4배** 감소
- 근거 없는 주장 감소, 불확실성 명시 향상

#### 프롬프트 캐싱 개선

- 최소 캐싱 가능 길이: **1,024 tokens** (4.7보다 낮음)

### PyTorchKR 재구성본과의 차이

- 원본: Fast Mode (`speed: "fast"`) 파라미터 — 재구성본에서 생략됨
- 원본: Mid-conversation system role — 재구성본에서 생략됨
- 원본: Dynamic Workflows는 **연구 미리보기** — 재구성본에서 정식 기능처럼 서술

---

## 6. Effort Parameter

**1차 출처**: `platform.claude.com/docs/en/build-with-claude/effort`  
**교차 검증**: liteLLM docs (1차 준수), Apiyi.com (3자)

### 원본 핵심

#### Effort 레벨 및 공식 권장 용도

| 레벨 | 공식 권장 용도 (원본 표현) |
|------|--------------------------|
| `low` | "simple classification, quick fact lookups, or high-volume jobs where a marginally better answer isn't worth the extra latency or spend" |
| `medium` | "balanced option when you want solid performance without the full token expenditure of high effort" |
| `high` | "minimum for most intelligence-sensitive workloads" |
| `xhigh` | "advanced coding and complex agentic work requiring extended exploration, such as repeated tool calling and detailed search" |
| `max` | "absolute highest capability with no constraints: the most thorough reasoning and deepest analysis" |

```python
# API 파라미터 정확한 이름
response = client.messages.create(
    model="claude-opus-4-8",
    output_config={"effort": "high"},  # 최상위: output_config, 중첩: effort
    messages=[...]
)
```

#### 핵심 정책

- **"behavioral signal, not a strict token budget"** — effort 수준에 따라 Claude가 스스로 사고 깊이를 조정
- **API 기본값**: `high` (Opus 4.8 기준)
- **beta 헤더 불필요**: `output_config.effort`는 별도 헤더 없이 사용
- **지원 모델**: Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 4.6, Opus 4.5, Claude Mythos Preview
- **공식 스텝다운 권고**: "Step down to medium for cost-sensitive workloads, or up to max only when your evals show measurable headroom at xhigh"

### PyTorchKR 재구성본과의 차이 ⚠️ 중요

| 항목 | 원본 (Anthropic 공식) | PyTorchKR 재구성본 | 판단 |
|------|----------------------|-------------------|------|
| API 기본값 | `high` | "대부분 코딩·에이전트 작업 → `xhigh`" (기본값 언급 없이 xhigh 강조) | **재구성본 강조점 이동** — xhigh가 추천된 것처럼 오해 소지 |
| "Recommended default" | 공식 표현 없음. medium은 비용 민감 워크로드용 권장 | "medium = 권장 기본값" 표에 명시 | **재구성본 의역** — 원본에 없는 "recommended default" 개념 추가 |
| `xhigh` 적용 범위 | "advanced coding + complex agentic work requiring extended exploration" | "대부분 코딩·에이전트 작업" | **범위 과대 일반화** |

---

## 7. Adaptive Thinking

**1차 출처**: `platform.claude.com/docs/en/build-with-claude/adaptive-thinking`  
**교차 검증**: DEV Community, AWS Bedrock docs (3자)

### 원본 핵심

- **기본 동작**: 기본적으로 **off** — `thinking` 필드 없으면 thinking 없이 실행
- **활성화 방법**: `thinking: {type: "adaptive"}` 명시 필요 (기존 `"enabled"` → `"adaptive"`)

```python
# 마이그레이션: budget_tokens → adaptive + effort
# Before (Opus 4.6 이하)
thinking = {"type": "enabled", "budget_tokens": 32000}

# After (Opus 4.7+)
thinking = {"type": "adaptive"}
output_config = {"effort": "high"}  # 또는 xhigh, medium
```

- **Interleaved thinking**: 도구 호출 사이사이에도 thinking 자동 실행 → 에이전틱 워크플로우에 특히 효과적
- **지원 모델**: Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 4.6, Claude Mythos Preview
- **성능 우위** (공식 표현): "Adaptive thinking can drive better performance than extended thinking with a fixed `budget_tokens` for many workloads, especially bimodal tasks and long-horizon agentic workflows"
- **effort 연동**: high/xhigh/max → 거의 항상 deep thinking; low/medium → 단순 문제는 thinking 생략

### PyTorchKR 재구성본과의 차이

- 원본: "bimodal tasks and long-horizon agentic workflows"에서 특히 우수 — 재구성본에서 "일반적으로 우수"로 일반화
- 원본: Interleaved thinking(도구 호출 사이 thinking) — 재구성본에서 언급 없음
- 재구성본의 과잉 사고 방지 프롬프트 예시 — 공식 문서에서 직접 확인되지 않음 (미확정)

---

## 8. Extended Thinking

**1차 출처**: `platform.claude.com/docs/en/build-with-claude/extended-thinking`

### 원본 핵심

| 모델 | budget_tokens 상태 |
|------|-------------------|
| Opus 4.6, Sonnet 4.6 | Deprecated (still accepted, 미래 제거 예정) |
| Opus 4.7+ | **완전 제거** — 400 Bad Request |
| Opus 4.8 | **완전 제거** — 400 Bad Request |

**에러 메시지 원문** (Opus 4.7+ 에서 `budget_tokens` 사용 시):

```
"thinking.type.enabled" is not supported for this model.
Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

- Extended Thinking 용어 자체는 문서에 여전히 존재하나, API 파라미터로서는 폐기 경로
- Adaptive Thinking이 공식 후속·대체

### PyTorchKR 재구성본과의 차이

- 원본: Opus 4.6에서 `budget_tokens` deprecated (not removed yet) — 재구성본에서 4.7+ 기준만 서술
- 원본: 정확한 에러 메시지 문자열 제공 — 재구성본에서 에러 코드 `400`만 언급

---

## 9. Structured Outputs

**1차 출처**: `platform.claude.com/docs/en/build-with-claude/structured-outputs`  
**교차 검증**: Towards Data Science, Helicone GitHub Issue (3자)

### 원본 핵심

#### GA 파라미터 (베타 헤더 불필요)

```python
response = client.messages.create(
    model="claude-opus-4-8",
    output_config={
        "format": {
            "type": "json_schema",
            "json_schema": {
                "name": "response_schema",
                "schema": {
                    "type": "object",
                    "properties": {
                        "result": {"type": "string"},
                        "confidence": {"type": "number"}
                    },
                    "required": ["result", "confidence"]
                }
            }
        }
    },
    messages=[...]
)
```

- **파라미터 변천**: `output_format` (베타) → `output_config.format` (GA 정식 명칭)
- **베타 헤더**: `anthropic-beta: structured-outputs-2025-11-13` (2025-11-14 public beta 시작) — GA 이후에도 전환 기간 동안 호환 유지

#### 두 가지 기능

1. **JSON outputs**: `output_config.format` — JSON schema 강제
2. **Strict tool use**: `strict: true` — tool 이름·입력 스키마 검증

#### API 레벨 검증 방식

- JSON schema를 grammar로 컴파일 → **inference 시 token 생성 자체를 제한**
- 결과: malformed JSON 반환 물리적으로 불가 (prompt engineering과 근본적으로 다름)

#### Prefill과의 공식 관계

- Structured Outputs = **프리필의 공식 대체 수단**
- 프리필은 "workaround"였음 — Structured Outputs가 proper API feature

#### 지원 모델 (GA)

Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 4.6, Sonnet 4.5, Opus 4.5, Haiku 4.5, Claude Mythos Preview

### PyTorchKR 재구성본과의 차이

- 원본: `output_config.format` (정확한 GA 파라미터명) — 재구성본에서 명시 없음
- 원본: Strict tool use (`strict: true`) 기능 별도 존재 — 재구성본에서 언급 없음
- 재구성본 내용 전반적으로 원본과 일치 (가장 충실한 재구성)

---

## 10. Context Windows & Awareness

**1차 출처**: `platform.claude.com/docs/en/build-with-claude/context-windows` + `platform.claude.com/docs/en/build-with-claude/compaction`  
**교차 검증**: AWS Bedrock 문서, Anthropic support 페이지 (1차 준수)

### 원본 핵심

#### 모델별 컨텍스트 윈도우 크기

| 모델 | 최대 컨텍스트 | 접근 조건 |
|------|------------|---------|
| Opus 4.8/4.7/4.6, Sonnet 4.6 | **1M 토큰** | API: `anthropic-beta: context-1m-2025-08-07` 헤더 + **Tier 4 조직 또는 custom rate limit 계정** 한정 |
| claude.ai 유료 플랜 | 500K 토큰 | — |
| Claude Code (Pro/Max/Team/Enterprise) | 1M 토큰 | — |
| 그 외 모든 모델 | 200K+ 토큰 (약 500페이지+) | — |

#### Compaction (컨텍스트 압축) 메커니즘

```python
# 베타 헤더 필요
headers = {"anthropic-beta": "compact-2026-01-12"}

# 커스텀 임계값 설정
compaction_config = {
    "type": "input_tokens",
    "value": 150000  # 최솟값: 50,000 토큰
}
```

- 지원 모델: Opus 4.8, Mythos Preview, Opus 4.7, Opus 4.6, Sonnet 4.6
- 트리거: 입력 토큰이 임계값 초과 시 자동 발동
- 동작: 대화 요약 생성 → `compaction block` 생성 → 이전 메시지 자동 삭제
- 모드: 자동(automatic) / `pause_after_compaction` 선택 가능

#### 성능 데이터

- **MRCR v2 1M 토큰** 기준: Opus 4.6 **78.3% 정확도** (1차 확인)
- **장문 컨텍스트 요금 프리미엄**: 2026년 3월부터 폐지 — 200K 초과도 표준 단가 통일

#### 공식 컨텍스트 소진 방지 전략

1. Compaction 활성화로 무제한 대화 지속
2. 서브에이전트에 독립 컨텍스트 윈도우 할당 (오케스트레이터 컨텍스트 보호)
3. Agent Skills의 Progressive Disclosure — 필요 시에만 관련 리소스 로드

### PyTorchKR 재구성본과의 차이 ⚠️ 중요

| 항목 | 원본 | 재구성본 | 판단 |
|------|------|---------|------|
| 1M 컨텍스트 접근 | **Tier 4 / custom rate limit 계정 한정** (베타 헤더 필요) | 제한 없이 사용 가능한 것처럼 서술 | **재구성본이 접근 제한 누락** |
| 30% 성능 개선 | "문서를 쿼리 아래 두면 품질 최대 30% 하락" (하락 방지 맥락) | "30% 개선"으로 서술 | 의미 동일하나 표현 방향 반대 |
| Compaction 메커니즘 | 별도 베타 헤더 필요, 설정 방법 구체적 | 언급 없음 | 재구성본에서 생략됨 |

> **⚠️ 주의**: "1M 컨텍스트 = 모든 API 사용자에게 열림"은 현재 **오해**. Tier 4 + 베타 헤더 필요.

---

## 11. Agent Skills Overview

**1차 출처**: `platform.claude.com/docs/en/agents-and-tools/agent-skills/overview` + `anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills` (직접 접근 확인) + `github.com/anthropics/skills`

### 원본 핵심

#### 공식 정의

- **Agent Skills**: instructions, metadata, optional resources(스크립트, 템플릿)를 패키징한 모듈식 에이전트 능력 단위
- **최소 구조**: `SKILL.md` 파일을 포함하는 디렉토리 (YAML frontmatter에 `name`, `description` 필수)
- **오픈 스탠다드**: Anthropic이 개발 후 공개 (Anthropic 독점 포맷 아님)

```
skills/
  my-skill/
    SKILL.md          # 필수: YAML frontmatter (name, description) + 지침
    resources/        # 선택: 스크립트, 템플릿, 참조 파일
    tools/            # 선택: 커스텀 도구 정의
```

#### Built-in vs Custom Skills

| 구분 | 내용 |
|------|------|
| Built-in (Pre-built) | PowerPoint, Excel, Word, PDF 등 공식 문서 작업용. claude.ai, Claude API, AWS Claude, Microsoft Foundry 제공 |
| Custom Skills | 직접 제작. 동일한 SKILL.md 구조 사용. 재배포 가능 |

#### Progressive Disclosure (핵심 설계 원칙)

- Claude는 초기에 시스템 프롬프트 + 스킬 메타데이터 요약만 보유
- 관련 요청 발생 시 SKILL.md 전문 로드
- 하위 리소스는 필요 시에만 추가 로드
- **결과**: 컨텍스트 비용 무제한 확장 방지 + 필요 정보만 제때 공급

#### 서브에이전트 공식 설계 원칙

- 서브에이전트 = `Task(Agent)` 도구로 부모 세션에서 생성하는 독립 Claude 세션
- 각 서브에이전트: 독립 컨텍스트 윈도우 + 독립 도구 목록 + 격리 전략(isolation strategy)
- 오케스트레이터는 서브에이전트 결과만 수신 (하위 작업의 전체 기록 미보유)
- 병렬화 최적 후보: 출력이 서로 의존하지 않는 독립 작업

#### 에이전트 설계 3원칙 (공식)

1. 단순성 유지 (Simplicity)
2. 계획 단계 명시적 표시 (Transparency)
3. 도구 문서화·테스트로 agent-computer interface 정교화

### PyTorchKR 재구성본과의 차이

- 원본: Skills = 디렉토리 단위 구조화 패키지 (SKILL.md + 리소스 포함) — 재구성본에서 언급 없음
- 원본: Progressive Disclosure가 **공식 설계 원칙으로 명명** — 재구성본에서 "필요할 때 로드"로 비공식 서술
- 원본: 오픈 스탠다드 — 재구성본에서 언급 없음

---

## 12. 원본 ↔ PyTorchKR 재구성본 대조표

| # | 주제 | 원본 | PyTorchKR 재구성본 | 유형 | 심각도 |
|---|------|------|------------------|------|:------:|
| 1 | Effort API 기본값 | `high` | 명시 없이 `xhigh` 강조 | 강조 이동 | ⚠️ |
| 2 | effort "recommended default" | 공식 표현 없음. `medium` = 비용 민감 워크로드 권장 | "medium = recommended default"로 표에 명시 | 의역·추가 | ⚠️ |
| 3 | `xhigh` 적용 범위 | "advanced coding + complex agentic requiring extended exploration" | "대부분 코딩·에이전트 작업" | 범위 과대 일반화 | ⚠️ |
| 4 | 1M 컨텍스트 접근 조건 | Tier 4 + 베타 헤더 필요 | 조건 없이 사용 가능한 것처럼 서술 | 접근 제한 누락 | 🔴 |
| 5 | Long context 30% | "쿼리 아래 두면 30% 하락" (하락 방지 맥락) | "30% 개선"으로 긍정형 전환 | 표현 방향 반전 | 🟡 |
| 6 | `temperature`/`top_p`/`top_k` 제거 | Opus 4.7+ 에서 400 에러 (마이그레이션 체인지) | 언급 없음 | 누락 | ⚠️ |
| 7 | Dynamic Workflows | 연구 미리보기 (research preview) | 정식 기능처럼 서술 | 상태 오표기 | 🟡 |
| 8 | Prefill 종료 사유 | API 레벨 3가지 이유 (기술적) | 모델 지능 향상으로 불필요해짐 (단순화) | 의역 | 🟡 |
| 9 | Extended Thinking | Opus 4.6에서 deprecated (not removed) | 바로 400 에러로 서술 | 오차 | 🟡 |
| 10 | Adaptive Thinking 공식 표현 | "bimodal tasks + long-horizon agentic" 특히 우수 | 일반적으로 우수 | 일반화 | 🟡 |
| 11 | Agent Skills 구조 | 디렉토리 패키지 (SKILL.md + resources) | 언급 없음 | 누락 | 🟡 |
| 12 | Deprecated 일정 | 구체적 날짜 (2026-06-15) | 생략 | 누락 | 🟡 |
| 13 | Fast Mode / Mid-conversation system role | Opus 4.8 신기능 | 언급 없음 | 누락 | 🟡 |

범례: 🔴 사실 오류·접근 제한 누락 / ⚠️ 강조 이동·의역으로 오해 소지 / 🟡 단순 누락·일반화

---

## 13. 갭 백로그 §8 1차 출처 매핑

| 갭 ID | 갭 제목 | 1차 출처 (본 문서 절) | 검증 결과 |
|-------|--------|----------------------|---------|
| P0-4 | 모델별 프롬프팅 전략 + 세대 마이그레이션 | §4 Migration Guide + §5 Opus 4.8 + §6 Effort + §7 Adaptive | ✅ 확인됨 |
| P1-W1 | 강압적 표현 → 과잉 발동 경고 | §2 Prompt Engineering (`claude-4-best-practices` 페이지) | 🟡 부분 확인 — 공식 페이지 존재 확인, 표현 강도 경고 세부는 미확정 |
| P1-W2 | 금지형 → 지시형 전환 원칙 | §2 Prompt Engineering | 🟡 부분 확인 — "be clear and direct" 원칙으로 간접 확인 |
| P1-W3 | 병렬 도구 호출 XML 패턴 | §11 Agent Skills (병렬 도구 호출 원칙) | ✅ 공식 원칙 확인됨 (XML 패턴은 claude-4-best-practices 미확인) |
| P1-W4 | 어드바이저 패턴 (Opus 조언자 + Sonnet 실행자) | §3 Models Overview + §11 Agent Skills | 🟡 부분 확인 — 공식 권장 패턴으로 직접 명명됐는지 미확인 |
| P1-W5 | 컨텍스트 압축 알림 프롬프트 | §10 Context Windows (Compaction) | ✅ Compaction 공식 메커니즘 확인됨 |
| P1-W6 | 프리필 종료 고지 | §4 Migration Guide + §8 Extended Thinking + §9 Structured Outputs | ✅ 완전 확인됨 (에러 메시지 원문 포함) |
| P1-W7 | 코드 환각 방지 지침 | §2 Prompt Engineering (`claude-4-best-practices`) | 🟡 부분 확인 — 공식 페이지 존재하나 해당 지침 세부 미확인 |
| P1-W8 | 과잉 엔지니어링 억제 XML | §2 Prompt Engineering | 🟡 부분 확인 — "scope/documentation" 규칙 직접 확인 필요 |
| P1-W9 | 가역성 기준 안전 행동 프롬프트 | §11 Agent Skills (에이전트 설계 원칙) | ✅ "단순성·투명성" 원칙으로 확인됨 |
| P1-W10 | 긴 컨텍스트 상단 배치 (30% 개선) | §2 Prompt Engineering (Long context tips) | ✅ 확인됨 — 단, 원문은 "30% 하락 방지" (재구성본은 "30% 개선"으로 표현 반전) |

---

## 14. 미확정·후속 확인 항목

WebSearch로 확정하지 못한 항목. **가이드 갱신 시 사용 전 추가 확인 필요**:

| # | 항목 | 현재 상태 | 확인 방법 |
|---|------|---------|---------|
| 1 | Effort "medium = recommended default" 공식 표현 존재 여부 | ❌ 미확인 — `high`가 API 기본값이고 `medium`은 비용 민감 권장으로 구분됨 | `platform.claude.com/docs/.../effort` 직접 열람 |
| 2 | Haiku 4.5 정확한 API 문자열 | 🟡 `claude-haiku-4-5-20251001` 추정 (dateless는 4.6 세대부터) | Models Overview 직접 확인 |
| 3 | Adaptive Thinking 과잉 사고 방지 공식 프롬프트 예시 | ❌ 공식 문서에서 직접 확인 불가 | Adaptive Thinking 페이지 직접 열람 |
| 4 | Effort 레벨별 비용·지연 공식 수치 | ❌ 3자 수치만 존재 (Apiyi.com: low=40% 토큰 감소) | Effort 문서 + 독립 벤치마크 확인 |
| 5 | Sonnet 4.6의 기본 effort 레벨 | 🟡 "high" 추정 — 일부 소스 "medium 권장" 혼재 | Sonnet 4.6 모델 페이지 확인 |
| 6 | Anthropic 공식 "어드바이저 패턴" 명칭 사용 여부 | 🟡 비공식 패턴 — 공식 문서에서 동일 명칭 사용 미확인 | Multi-agent 설계 가이드 확인 |
| 7 | `claude-4-best-practices` 페이지의 강압적 표현 경고 내용 | 🟡 페이지 존재 확인, 세부 지침 미확인 | 직접 열람 필요 |
| 8 | output_config.format 베타 헤더 sunset 날짜 | ❌ 미확인 | Migration Guide + Release Notes 확인 |
| 9 | budget_tokens 기본값 400 | ❌ 공식 문서에서 확인 불가. 이전 세션에서 "Opus 4.7+ 에서 400 에러"로 다른 의미로 사용됨 | Extended Thinking 직접 열람 — 이 수치 자체가 에러 코드 400과 혼동이었을 가능성 |
| 10 | Fast Mode 프리미엄 가격 배수 | ❌ 미확인 | Opus 4.8 pricing 페이지 확인 |

# Claude 4 세대 모델별 프롬프팅 전략

| 항목 | 내용 |
|------|------|
| 생성일 | 2026-06-02 |
| 변경일 | 2026-06-02 |
| 대상 | Claude API 사용자, Claude Code 자동화 파이프라인 운영자 |
| 출처 | Anthropic 공식 문서 (`platform.claude.com`) — `sources/anthropic-공식문서-정리-20260602.md` §3~8 |

> **면책**: 이 가이드는 1차 출처인 Anthropic 공식 문서를 기반으로 작성되었습니다.
> `platform.claude.com`은 JS 렌더링으로 직접 열람이 어려우므로 WebSearch + 3자 미러 교차 검증됨.
> 모델 API 문자열·파라미터는 실제 사용 전 공식 문서 최신본과 교차 확인할 것.

---

## 목차

1. [Claude 4 모델 현황](#1-claude-4-모델-현황)
2. [Effort 파라미터 — 사고 깊이 제어](#2-effort-파라미터--사고-깊이-제어)
3. [Adaptive Thinking — budget_tokens 대체](#3-adaptive-thinking--budget_tokens-대체)
4. [Prefill 종료 — 브레이킹 체인지 대응](#4-prefill-종료--브레이킹-체인지-대응)
5. [Opus 4.8 신기능](#5-opus-48-신기능)
6. [마이그레이션 체크리스트](#6-마이그레이션-체크리스트)
7. [모델 선택 가이드](#7-모델-선택-가이드)
8. [커뮤니티 vs 공식 수치 대조](#8-커뮤니티-vs-공식-수치-대조)

---

## 1. Claude 4 모델 현황

### 1.1 현재 지원 모델 (2026-06-02 기준)

| 모델명 | API 문자열 | 특징 | 상태 |
|--------|-----------|------|------|
| Claude Opus 4.8 | `claude-opus-4-8` | 최고 성능. 1M 컨텍스트. Dynamic Workflows | ✅ 현재 |
| Claude Opus 4.7 | `claude-opus-4-7` | 새 토크나이저. Adaptive Thinking 도입 | ✅ 현재 |
| Claude Opus 4.6 | `claude-opus-4-6` | Desktop에서 제거됨, API 유지 | ⚠️ API만 |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 속도·지능 균형. 공식 "recommended" | ✅ 현재 |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | 가장 빠르고 저렴. 단순 작업 | ✅ 현재 |

**Deprecated 예정**: `claude-sonnet-4-20250514`, `claude-opus-4-20250514` → 2026-06-15 retirement

### 1.2 모델 ID 규칙

- Claude 4.6 세대부터 날짜 접미사 없는 dateless ID 표준
- Haiku 4.5는 날짜 접미사(`-20251001`)가 canonical일 수 있음
- 코드에서 날짜 포함 구버전 ID를 쓰고 있다면 → 마이그레이션 체크리스트 §6 확인

---

## 2. Effort 파라미터 — 사고 깊이 제어

> **"behavioral signal, not a strict token budget"** — Anthropic 공식 문서

Effort는 토큰 예산이 아니라 Claude가 스스로 사고 깊이를 조절하는 신호다.

### 2.1 레벨 표

| 레벨 | 공식 권장 용도 | 비용 |
|------|--------------|------|
| `low` | 단순 분류, 빠른 팩트 조회, 대용량 배치 작업 | 가장 낮음 |
| `medium` | 비용 민감 워크로드, 균형 성능 | 낮음 |
| `high` | **대부분의 지능 민감 워크로드의 최소값** (API 기본값) | 보통 |
| `xhigh` | 고급 코딩, 반복 도구 호출이 필요한 복잡한 에이전틱 작업 | 높음 |
| `max` | 제약 없는 최고 추론·분석 | 가장 높음 |

**API 기본값**: `high` (Opus 4.8 기준, 명시하지 않으면 high로 동작)

### 2.2 API 사용법

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-8",
    output_config={"effort": "high"},  # output_config 안에 effort
    max_tokens=8096,
    messages=[{"role": "user", "content": "복잡한 코드 리뷰 요청"}]
)
```

```bash
# Claude Code CLI에서는 /effort 커맨드 또는 CLAUDE.md에 설정
# 아직 CLI 직접 파라미터 미확인 — API 경로 활용
```

### 2.3 스텝다운 전략 (공식 권고)

> "Step down to medium for cost-sensitive workloads, or up to max only when your evals show measurable headroom at xhigh"

실전 적용:

```
기본: high
  ↓ (비용 문제) → medium (배치, 대용량 분류)
  ↓ (더 저렴하게) → low (단순 yes/no 분류, 키워드 추출)

기본: high
  ↑ (성능 부족) → xhigh (에이전틱 루프, 반복 도구 호출)
  ↑ (eval에서 xhigh 포화 확인 후) → max
```

### 2.4 beta 헤더 불필요

```python
# effort 파라미터는 별도 anthropic-beta 헤더 없이 사용
# ❌ 잘못된 예
headers = {"anthropic-beta": "output-config-2025-08-22"}  # 불필요

# ✅ 올바른 예
output_config = {"effort": "high"}  # 그냥 output_config에 포함
```

---

## 3. Adaptive Thinking — budget_tokens 대체

Opus 4.7+ 에서 `budget_tokens` 파라미터가 제거되었다. Adaptive Thinking이 대체 방식이다.

### 3.1 마이그레이션

```python
# ❌ 이전 방식 (Opus 4.6 이하) — Opus 4.7+에서 400 Bad Request
thinking = {
    "type": "enabled",
    "budget_tokens": 32000
}

# ✅ 새 방식 (Opus 4.7+)
thinking = {"type": "adaptive"}
output_config = {"effort": "high"}  # 또는 xhigh, max

response = client.messages.create(
    model="claude-opus-4-8",
    thinking=thinking,
    output_config=output_config,
    messages=[...]
)
```

### 3.2 Adaptive vs Extended Thinking 차이

| 구분 | Extended Thinking (구) | Adaptive Thinking (신) |
|------|----------------------|----------------------|
| 파라미터 | `budget_tokens` 명시 | `effort` 레벨로 간접 제어 |
| 동작 | 고정 토큰 예산 소진 | Claude가 필요한 만큼 자동 조정 |
| 지원 | Opus 4.6까지 | Opus 4.7+ |
| 성능 | - | bimodal 태스크·장기 에이전틱 워크플로우에서 우수 |

> 공식: "Adaptive thinking can drive better performance than extended thinking with a fixed `budget_tokens` for many workloads, especially bimodal tasks and long-horizon agentic workflows"

### 3.3 Interleaved Thinking

Adaptive Thinking 활성화 시 도구 호출 **사이사이에도** thinking이 자동 실행된다.

```
[사용자 요청]
  → [thinking] → [도구 호출 1]
  → [thinking] → [도구 호출 2]   ← 이전에는 없었던 동작
  → [thinking] → [최종 응답]
```

에이전틱 워크플로우에서 특히 효과적. 도구 결과를 받은 뒤 재사고하는 행동이 자동화된다.

### 3.4 effort와 thinking 연동

| effort 레벨 | thinking 동작 |
|------------|--------------|
| `high` / `xhigh` / `max` | 거의 항상 deep thinking 실행 |
| `low` / `medium` | 단순 문제는 thinking 생략, 복잡한 문제만 실행 |

---

## 4. Prefill 종료 — 브레이킹 체인지 대응

### 4.1 무엇이 바뀌었는가

Prefill(assistant 메시지 미리 채우기)이 **Opus 4.7+** 에서 지원 종료되었다.

```python
# ❌ 더 이상 안 됨 (Opus 4.7+에서 400 오류)
messages = [
    {"role": "user", "content": "JSON으로 응답해줘"},
    {"role": "assistant", "content": "{"}  # Prefill — 이제 불가
]
# 오류: "Prefilling assistant messages is not supported for this model."
```

**영향 모델**: Opus 4.8, 4.7, 4.6, Sonnet 4.6, Mythos Preview

### 4.2 종료 사유 (공식 3가지)

1. 유효한 JSON 보장 불가 (generation trick에 불과했음)
2. 컨텍스트 토큰 소비 불명확 (attributable하지 않음)
3. Structured Outputs 등 신기능과 충돌

### 4.3 대체 방법

**방법 1 — Structured Outputs (권장)**

```python
response = client.messages.create(
    model="claude-opus-4-8",
    output_config={
        "format": {"type": "json_object"}  # 또는 json_schema
    },
    messages=[{"role": "user", "content": "이름과 나이를 JSON으로 알려줘"}]
)
```

**방법 2 — 시스템 프롬프트에 형식 지정**

```python
system = """
응답은 반드시 다음 JSON 형식을 따르세요:
{"name": "...", "age": 0}
다른 텍스트 없이 JSON만 출력하세요.
"""
```

**방법 3 — continuation을 user 메시지로 이동**

```python
# 이전: assistant prefill로 문장 이어받기
# 이후: user 메시지에서 "다음 내용을 이어서 써줘" 패턴
messages = [
    {"role": "user", "content": "이전 코드: ...\n이어서 작성해줘"}
]
```

### 4.4 temperature / top_p / top_k 제한

Opus 4.7+에서 비기본값 설정 시 `400` 오류.

```python
# ❌ Opus 4.7+에서 400 오류
response = client.messages.create(
    model="claude-opus-4-8",
    temperature=0.7,  # 비기본값 금지
    top_p=0.9,
    messages=[...]
)

# ✅ 파라미터 완전 제거
response = client.messages.create(
    model="claude-opus-4-8",
    messages=[...]
)
```

---

## 5. Opus 4.8 신기능

### 5.1 Dynamic Workflows (연구 미리보기)

- Claude Code에서 수백 개의 병렬 서브에이전트를 단일 세션에서 실행
- 대규모 코드베이스 마이그레이션(수십만 라인)을 킥오프~머지까지 자동화
- 가용 플랜: Enterprise, Team, Max

> **주의**: 2026-06-02 기준 연구 미리보기(Research Preview). 정식 기능 아님.

### 5.2 Fast Mode (API 연구 미리보기)

```python
response = client.messages.create(
    model="claude-opus-4-8",
    speed="fast",  # 신규 파라미터
    messages=[...]
)
```

- 동일 모델 대비 출력 토큰 속도 **최대 2.5배** 향상
- 이전 모델 fast mode 대비 **3배 저렴**
- 프리미엄 가격 적용

### 5.3 Mid-conversation System Role

```python
# Opus 4.8 전용: 대화 중간에 system 지시 추가 가능
messages = [
    {"role": "user", "content": "처음 요청"},
    {"role": "assistant", "content": "처음 응답"},
    {"role": "system", "content": "이후부터 더 간결하게 답변"},  # 신규
    {"role": "user", "content": "다음 요청"}
]
```

- 이전 모델(4.7 포함)에서는 동일 방식 시 `400` 오류
- 긴 대화 중 전체 시스템 프롬프트 재전송 없이 지시사항 업데이트

### 5.4 품질 개선

- 코드 결함 미보고 비율: Opus 4.7 대비 **약 4배** 감소
- 프롬프트 캐싱 최소 길이: **1,024 tokens** (4.7보다 낮음 → 짧은 프롬프트도 캐싱 혜택)

---

## 6. 마이그레이션 체크리스트

Opus 4.6 이하 → Opus 4.7+ 마이그레이션 시 반드시 확인할 6개 항목.

```
☐ 1. budget_tokens 제거
      thinking = {"type": "enabled", "budget_tokens": N}
   →  thinking = {"type": "adaptive"}
      output_config = {"effort": "high"}

☐ 2. Prefill 패턴 제거
      messages에서 assistant 역할 사전 채우기 → Structured Outputs 또는 시스템 프롬프트로 대체

☐ 3. temperature / top_p / top_k 파라미터 제거
      비기본값이면 → 파라미터 완전 삭제 (기본값만 허용)

☐ 4. 토큰 카운팅 로직 재검증
      Opus 4.7 신규 토크나이저: 동일 입력 대비 1.0~1.35배 토큰 증가
      한국어·CJK 문자에서 증가 폭 더 큼

☐ 5. 모델 API 문자열 업데이트
      claude-sonnet-4-20250514, claude-opus-4-20250514 → Deprecated (2026-06-15)
      → claude-sonnet-4-6, claude-opus-4-8 로 교체

☐ 6. Haiku 4.5 API 문자열 확인
      claude-haiku-4-5 또는 claude-haiku-4-5-20251001 — 날짜 접미사 포함 여부 공식 확인 필요
```

### 6.1 빠른 자가 진단

```python
# 코드베이스에서 위험 패턴 grep
grep -r "budget_tokens" .
grep -r "role.*assistant.*prefill\|prefill.*assistant" .
grep -r "temperature\s*=\s*[^0]\|top_p\s*=\|top_k\s*=" .
grep -r "claude-sonnet-4-20250514\|claude-opus-4-20250514" .
```

---

## 7. 모델 선택 가이드

### 7.1 태스크별 추천 모델

| 태스크 유형 | 추천 모델 | effort | 이유 |
|------------|---------|--------|------|
| 단순 분류·키워드 추출 | Haiku 4.5 | low | 최저 비용 |
| 요약·번역·문서 초안 | Sonnet 4.6 | medium | 균형 |
| 코드 리뷰·일반 코딩 | Sonnet 4.6 | high | 공식 recommended |
| 복잡한 에이전틱 루프 | Opus 4.8 | xhigh | 반복 도구 호출 최적 |
| 최고 품질 단발 추론 | Opus 4.8 | max | 비용 무관, 품질 최우선 |
| 대규모 배치 파이프라인 | Haiku 4.5 | low | 처리량 중시 |

### 7.2 Claude Code 파이프라인에서의 모델 배분

```yaml
# 예시: 멀티에이전트 파이프라인 모델 배분
orchestrator:
  model: claude-opus-4-8
  effort: high

subagents:
  - role: file-reader
    model: claude-sonnet-4-6
    effort: medium
  - role: code-writer
    model: claude-opus-4-8
    effort: xhigh
  - role: test-runner
    model: claude-sonnet-4-6
    effort: high
  - role: summarizer
    model: claude-haiku-4-5-20251001
    effort: low
```

---

## 8. 커뮤니티 vs 공식 수치 대조

커뮤니티 자료에서 공식 문서와 다르게 서술된 주요 항목.

| 항목 | 공식 문서 (Anthropic) | 일부 커뮤니티 자료 | 판단 |
|------|---------------------|-----------------|------|
| effort API 기본값 | `high` | xhigh 강조 → 기본값 불명확 | **공식 기준 사용** |
| medium 포지셔닝 | 비용 민감 워크로드 권장 | "권장 기본값"으로 표기 | **공식: 기본값 아님** |
| xhigh 적용 범위 | "고급 코딩 + 복잡한 에이전틱" | "대부분 코딩·에이전트 작업" | **과대 일반화 주의** |
| Dynamic Workflows | 연구 미리보기 | 정식 기능처럼 서술 | **아직 미리보기** |
| 토크나이저 변경 | Migration Guide (1.0~1.35배) | "Anthropic 공식 효율 계수"로 다른 맥락 | **출처 동일, 맥락 다름** |
| Long context 배치 | 쿼리 아래 배치 시 품질 하락 | "위에 넣으면 30% 개선" | **의미 동일, 표현 방향만 역전** |

---

## 부록 — 관련 가이드 링크

- 비용 계산 방법 → [claude-code-비용효율-측정-프레임워크.md](claude-code-비용효율-측정-프레임워크.md)
- 시스템 프롬프트 설계 → [claude-code-시스템프롬프트-설계원칙.md](claude-code-시스템프롬프트-설계원칙.md)
- Cross-agent 워크플로우 → [claude-code-cross-agent-워크플로우.md](claude-code-cross-agent-워크플로우.md)
- 10단계 마스터리 로드맵 → [claude-code-10단계-마스터리-로드맵.md](claude-code-10단계-마스터리-로드맵.md)

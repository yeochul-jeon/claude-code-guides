# Claude Code 비용효율 측정 프레임워크: Token ≠ Task

| 항목 | 날짜 |
|------|------|
| 생성일 | 2026-06-02 |
| 변경일 | 2026-06-02 |

> "AI는 더 똑똑해졌지만, 효율을 계산하는 방식도 함께 달라졌다."  
> 토큰 수로 비용을 판단하는 시대에서, **Task 기준으로 가치를 측정**하는 시대로의 전환을 다룬다.
>
> 출처: 유민수 개발자 발표 자료 (Deformatic, 2026), Anthropic 공식 Migration Guide / Opus 4.7+ 출시 노트

### 관련 문서
- [10단계 마스터리 로드맵 §6](claude-code-10단계-마스터리-로드맵.md) — 토큰 절감 실전 기법 (마크다운 제거 10x, 세션 메모 패턴)
- [개인 설정 가이드 §3](claude-code-개인설정-가이드.md) — effort 파라미터로 비용·품질 균형 조절

---

## 목차

1. [왜 지금 측정 기준이 달라지는가](#1-왜-지금-측정-기준이-달라지는가)
2. [Token ≠ 자연 단위: 오해와 현실](#2-token--자연-단위-오해와-현실)
3. [새로운 4대 지표](#3-새로운-4대-지표)
4. [Cost per Task 계산법](#4-cost-per-task-계산법)
5. [모델 전환 전 체크리스트](#5-모델-전환-전-체크리스트)
6. [팀·경영진 설득용 ROI 프레임](#6-팀경영진-설득용-roi-프레임)
7. [실전 측정 워크플로우](#7-실전-측정-워크플로우)

---

## 1. 왜 지금 측정 기준이 달라지는가

새 모델이 출시될 때마다 개발자와 PM들은 묻는다: "이전보다 더 효율적인가?"
그런데 이 질문에 답하기 위한 **기준 자체가 조용히 달라지고 있다.**

### 무엇이 실제로 바뀌었나

| 변화 | 공식 근거 | 실무 영향 |
|------|----------|---------|
| **Tokenizer 변경** | Claude Opus 4.7부터 새 토크나이저 도입. 동일 텍스트가 이전 대비 **1.0~1.35배** 토큰으로 계산 (Anthropic Migration Guide) | 같은 프롬프트인데 청구서가 달라짐 |
| **Reasoning 방식 변화** | `effort=high/xhigh` 설정 또는 에이전틱 작업 후반 턴에서 output 토큰이 더 늘 수 있음 | 생각하는 과정 자체가 토큰에 포함됨 |
| **성능 향상으로 반복 횟수 감소** | 동일 task를 더 적은 시도(turn)로 완료 가능 | 토큰은 늘었지만 총 비용은 낮아질 수 있음 |

> **핵심 통찰**: "토큰이 늘었다는 사실보다 더 중요한 것은, 무엇을 효율이라고 부를 것인가다."

### 두 가지 혼동하기 쉬운 개념

| 개념 | 설명 |
|------|------|
| **Tokens per Request** | 요청 1회 당 토큰 수. 비교하기 쉽지만 실제 비즈니스 가치와 연결이 약하다. |
| **Cost per Task** | 동일 task를 완료하는 데 드는 총 비용. 토큰, 반복 횟수, 인건비까지 포함. |

---

## 2. Token ≠ 자연 단위: 오해와 현실

### 자주 보는 오해

| 오해 | 현실 |
|------|------|
| "토큰이 늘면 무조건 더 비싸다" | input 토큰이 늘어도 output이 줄거나 반복이 줄면 총비용은 낮아질 수 있다 |
| "같은 프롬프트 = 같은 비용이어야 한다" | 토크나이저가 달라지면 같은 텍스트도 토큰 수가 달라진다 (버그가 아니라 설계 선택) |
| "모델 업그레이드는 항상 효율을 나쁘게 만든다" | 성능이 높아지면 같은 task를 더 적은 시도로 완료해 전체 비용이 오히려 낮아질 수 있다 |
| "토큰 인플레이션은 사용자에게 불리한 변화다" | 가격표(per million tokens)는 그대로지만 실질 청구는 사용 패턴에 따라 다르다 |

### 왜 사람들이 민감하게 반응하는가

사소한 숫자 변화가 아닌 이유:

- **예산과 과금**: API 비용은 팀 예산과 직결. 소량 테스트에선 괜찮다가 프로덕션 규모에서 폭증하는 패턴
- **Rate Limit**: 많은 플랜에서 토큰 사용량이 rate limit 기준. 한도에 더 빨리 부딪히면 서비스 중단 위험
- **팀과 제품 운영**: 에이전트 파이프라인, 자동화 워크플로우는 예상 토큰 소비를 기반으로 설계됨
- **체감 공정성**: 가격표는 그대로인데 실질 비용이 달라지면 "룰이 몰래 바뀐 것"처럼 느껴짐

---

## 3. 새로운 4대 지표

"토큰 수"를 넘어선 task 기반 복합 지표.

### 3.1 Cost per Task (작업당 실비용)

동일한 task를 완료하는 데 드는 **총 API 비용**.

- 단일 요청의 토큰 수가 아닌, **task 완료 단위**로 집계
- 에이전트 루프가 있다면 전체 루프 비용을 합산

```python
# Claude Code 에이전트 루프 비용 집계 예시
total_cost = sum(turn.input_tokens * INPUT_PRICE + turn.output_tokens * OUTPUT_PRICE
                 for turn in task.turns)
cost_per_task = total_cost / tasks_completed
```

### 3.2 Success Rate per Task (작업 성공률)

모델이 주어진 task를 **요구 기준 이상으로 완료하는 비율**.

- 비용이 낮아도 성공률이 낮으면 실질 효율은 나쁘다
- 반대로 비용이 높아도 성공률이 월등히 높으면 전체 운영은 더 저렴해질 수 있다

```bash
# 성공률 측정 — evals/ 회귀 세트 기반
success_rate = passed_evals / total_evals
```

### 3.3 Latency per Task (작업당 응답 시간)

사용자 경험과 직결되는 지표. 실시간 애플리케이션에서 latency는 비용 못지않게 중요한 제약 조건.

- effort 레벨이 높을수록 latency 증가
- `effort=low`/`medium`은 latency 민감 용도에 적합

### 3.4 Human Review Load (인간 검토 부하)

자동화 파이프라인에서 **사람이 개입해야 하는 빈도와 깊이**.

- 모델 품질이 높아지면 이 부하가 줄고, 그것이 실질적인 비용 절감으로 이어진다
- "API 비용만이 전체 비용의 전부가 아니다"

---

## 4. Cost per Task 계산법

### 기본 산식

```
Cost per Task = (총 API 비용) / (완료된 task 수)

총 API 비용 = Σ (input_tokens × input_price + output_tokens × output_price)
             에이전트 루프 전체 턴 합산
```

### 실전 측정 예시 — PR 코드 리뷰

| 지표 | 구버전 (Sonnet 3.5) | 신버전 (Sonnet 4.6) | 변화 |
|------|-------------------|-------------------|------|
| 토큰/요청 | 12,000 | 15,000 (+25%) | 증가 |
| 평균 재시도 횟수 | 2.3회 | 1.1회 | 감소 |
| Cost per Task | $0.023 | $0.019 | **-17%** |
| Success Rate | 78% | 91% | **+13%p** |

> 토큰 수는 25% 늘었지만, 재시도가 절반으로 줄어 Cost per Task는 오히려 낮아진 패턴.

### 주의: "비싼 모델이 더 저렴할 수 있다"

- 고급 모델은 같은 task를 더 적은 turn으로 완료 → 총 토큰이 줄 수 있다
- 더 정확한 결과 → Human Review Load 감소 → 인건비 절감
- 이 효과를 포함해야 진짜 ROI 비교가 된다

---

## 5. 모델 전환 전 체크리스트

새 모델 도입 또는 전환 시 필수 점검 항목.

### 조직/제품 관점

- [ ] 현재 벤치마크가 토큰 기반에만 치우쳐 있지 않은가?
- [ ] 모델 전환 시 A/B 테스트 기준이 명확히 정의되어 있는가?
- [ ] 에이전트 파이프라인은 전체 턴 수를 기준으로 비용을 집계하고 있는가?
- [ ] output 품질 기준이 정량적으로 정의되어 있는가?
- [ ] 토크나이저 변화를 반영한 예산 재산정을 했는가?
- [ ] 비용 알림(alert) 임계값이 새 모델 기준으로 업데이트되었는가?

### Claude Code 환경 특화 점검

```bash
# 현재 사용 비용 확인
/cost

# 특정 작업의 토큰 사용량 비교
claude -p "PR #42 리뷰해줘" --output-format json | jq '.usage'
```

---

## 6. 팀·경영진 설득용 ROI 프레임

"AI 도입이 효율적인가?"라는 질문에 답할 때 사용할 수 있는 4축 프레임.

| 측정 축 | 측정 방법 | 경영진 언어 |
|---------|----------|-----------|
| **Cost per Task** | API 비용 ÷ 완료 task 수 | "건당 처리 비용" |
| **Quality per Task** | eval 통과율, Human Review Load | "산출물 품질" |
| **Success Rate per Task** | 첫 시도 성공률 | "재작업 없는 완료율" |
| **Latency per Task** | 평균 응답 시간 | "처리 속도" |

### 설득 전략: "비교 대상 설정"

AI 도입의 ROI를 주장할 때 비교 기준이 없으면 설득력이 약하다.

```
비교 시나리오 A: AI 없이 사람이 동일 task 수행
  → 시간 × 인건비 vs. API 비용

비교 시나리오 B: 구버전 AI vs. 신버전 AI
  → Cost per Task 직접 비교 (토큰 수 아님)

비교 시나리오 C: 자동화 전 vs. 자동화 후 Human Review Load
  → 사람이 개입하는 시간 절감분이 실질적인 ROI
```

### 커뮤니티 수치 vs. 공식 수치 구분

| 수치 유형 | 예시 | 신뢰도 | 활용 방법 |
|---------|------|--------|---------|
| **공식 공인** | 토크나이저 1.0~1.35배 (Migration Guide) | 높음 | 예산 계획에 활용 |
| **공식 미공인** | "30~45% 토큰 체감 증가" (Hacker News 관측치) | 낮음 | 내부 가설용으로만, 공식 주장 X |
| **자체 측정** | 우리 파이프라인의 실제 Cost per Task | 가장 높음 | 의사결정의 1차 근거 |

> **원칙**: 모델 회사의 가격표만 믿지 말고, 실제 사용 패턴을 기반으로 한 자체 벤치마크를 구축하라. 이것이 이제 실무의 기본 역량이다.

---

## 7. 실전 측정 워크플로우

### 기본 파이프라인

```bash
# 1. 현재 baseline 측정
claude -p "PR #10 리뷰" --output-format json > baseline.json

# 2. 반복 측정으로 평균 계산 (5회 이상)
for i in {1..5}; do
  claude -p "PR #$i 리뷰" --output-format json >> measurements.jsonl
done

# 3. Cost per Task 집계
cat measurements.jsonl | jq -s '
  [.[].usage] |
  {
    avg_input: (map(.input_tokens) | add / length),
    avg_output: (map(.output_tokens) | add / length),
    samples: length
  }
'
```

### evals/ 디렉토리로 지속 측정

```
.
├── evals/
│   ├── pr-review-set/        # PR 리뷰 회귀 세트
│   │   ├── case-001.json     # 입력 + 기대 결과
│   │   └── run-results.jsonl # 모델별 측정 결과
│   └── code-gen-set/         # 코드 생성 회귀 세트
└── CLAUDE.md
```

모델 전환 시 동일 evals 세트로 **Cost per Task + Success Rate**를 비교하면 토큰 수만 보는 것보다 훨씬 정확한 판단이 가능하다.

> Eval 세트 구성법: [하네스 엔지니어링 방법론 §9](claude-code-하네스-엔지니어링-방법론.md)

---

## 이 변화가 보여주는 더 큰 흐름

```
이전  → Tokens per Request로 단순 비교
현재  → Cost per Task로 실제 비용 비교
미래  → Quality·Success Rate·Latency로 다차원 평가
```

> "효율이 나빠진 것이 아니라, 효율을 재는 자가 바뀌고 있다."  
> 모델의 성능이 상향 평준화되고 가격 정책이 복잡해질수록, **어떤 기준으로 비교하고 어떻게 측정할지** 결정하는 역량이 경쟁력이 된다.

---

## Sources

- Deformatic / 유민수 개발자 — "AI는 더 똑똑해졌지만, 효율을 계산하는 방식도 함께 달라졌다" (2026)
- [Anthropic — Claude Opus 4.7 Migration Guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide) — 토크나이저 변화 1.0~1.35배
- [Anthropic — What's new in Claude Opus 4.8](https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-8)
- [Bill Chambers — Tokenomics Leaderboard](https://huggingface.co/spaces/BillChambers/tokenomics-leaderboard) — 모델별 토큰 소비 패턴 측정
- Hacker News 토론 스레드 — 커뮤니티 관측치 (공식 수치 아님)

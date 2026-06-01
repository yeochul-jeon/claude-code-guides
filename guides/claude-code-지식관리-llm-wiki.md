# Claude Code로 구축하는 LLM-Wiki 지식관리 시스템

| 항목 | 내용 |
|------|------|
| 생성일 | 2026-06-02 |
| 변경일 | 2026-06-02 |
| 대상 | 연구자·지식 작업자·RAG 인프라 없이 개인 지식관리 하려는 개발자 |
| 출처 | 안준용 (고려대 바이오시스템의과학부), "LLM-Wiki를 이용한 개인화된 지식 체인 구축", 2026-04-30, 국립보건연구원 헬스케어인공지능연구과 |

> **핵심 메시지**: AI 시대의 지식관리는 "논문 읽기"가 아니라 **"질문 가능한 지식 그래프 운영"**으로 바뀐다.

### 관련 문서
- [CLAUDE.md 실전 작성법](claude-code-CLAUDE-md-실전-작성법.md) — 위키 거버넌스 CLAUDE.md 작성 패턴 (§5 거버넌스 규칙과 연결)
- [개인 설정 가이드](claude-code-개인설정-가이드.md) — `.claude/commands/` 커스텀 커맨드, skills 설정 (Paper Monitor 연동)
- [비용효율 측정 프레임워크](claude-code-비용효율-측정-프레임워크.md) — 지식 생산 비용 최적화 (effort 레벨 조정, 토크나이저 변경 영향)

---

## 목차

1. [왜 지금 개인 위키인가](#1-왜-지금-개인-위키인가)
2. [RAG vs LLM-Wiki — 패러다임 비교](#2-rag-vs-llm-wiki--패러다임-비교)
3. [3-tier 디렉토리 구조](#3-3-tier-디렉토리-구조)
4. [YAML frontmatter 스키마](#4-yaml-frontmatter-스키마)
5. [CLAUDE.md 거버넌스 규칙](#5-claudemd-거버넌스-규칙)
6. [Paper Monitor 자동화](#6-paper-monitor-자동화)
7. [Claude Code 연동 실전](#7-claude-code-연동-실전)
8. [Karpathy 관점 — 지식 생산 플랫폼으로](#8-karpathy-관점--지식-생산-플랫폼으로)

---

## 1. 왜 지금 개인 위키인가

### 1.1 문제: 기억은 유한, 문헌은 무한

사람의 워킹 메모리는 **7 ± 2개 청크**. 1년 전 읽은 논문의 핵심을 정확히 기억해내는 연구자는 드물다.

기존 해결책의 한계:

| 도구 | 한계 |
|------|------|
| 프린트 + 밑줄 | 검색·재활용 불가 |
| EndNote / Mendeley | 메타데이터만, 본문 지식 없음 |
| 이메일 자기 전송 | 사실상 블랙홀 |
| 한/영 메모 파편 | 여기저기 흩어짐 |
| Notion / Roam | LLM이 읽을 수 없는 구조 |

### 1.2 필요한 것

- **로컬 파일** — Markdown, Git 버전 관리
- **구조화된 스키마** — YAML frontmatter
- **AI와 공유 가능한 형태** — LLM이 직접 읽는 파일
- **쿼리 가능** — RAG 또는 파일시스템 grep
- **누적 가능** — Overview로 지식 합성

### 1.3 결론

> PI의 시간은 **"읽기"** → **"읽은 것을 축적하고 질문 가능하게 만드는 것"** 으로 이동해야 한다.

안준용 교수의 LLM-Wiki는 2026년 4월 기준 **1,427페이지** (Sources 1,055 · Overviews 130 · Categories 33 · 원본 PDF 3,000+)까지 성장했다.

---

## 2. RAG vs LLM-Wiki — 패러다임 비교

RAG와 LLM-Wiki는 같은 문제를 푸는 두 방식이 아니다. **시간에 대한 태도**가 근본적으로 다른 두 패러다임이다.

| 관점 | RAG | LLM-Wiki |
|------|-----|----------|
| 목적 | "검색해서 답을 만든다" | "답을 만들며 지식을 쌓는다" |
| 인프라 | Vector DB · Embedding 서버 · Chunker-Reranker | 로컬 파일 + Git + Claude Code |
| 디버깅 | 잘못된 답변 = 검색 실패? 생성 실패? (블랙박스) | 파일 직접 열어 확인 |
| 지식 누적 | Overview-연결이 생기지 않음 | wiki/ → overviews/ 로 합성 누적 |
| 유저 허들 | "내 지식을 RAG에 태우려면 무엇을 해야 하나?" 불명확 | Markdown 파일 한 장에서 시작 |
| 엔지니어링 부담 | 업데이트·재인덱싱 주기 관리 필요 | `git add` 한 줄 |

**실험적 증거**: OpenScholar (Asai et al., *Nature* 2025) — 스타일의 과학 RAG를 직접 시도 결과, 훌륭하지만 매일 글리겐 부담이 컸다. (안준용 교수 실제 운영 경험)

**결론**: "인프라가 연구의 주인이 되면 안 된다."

---

## 3. 3-tier 디렉토리 구조

```
my-wiki/
├── CLAUDE.md               # 거버넌스 규칙 (§5 참조)
│
├── sources/                # Tier 1 — 원본 자료
│   ├── papers/             # PDF 원본 (반드시 실제 복사, symlink 금지)
│   │   ├── attention-is-all-you-need.pdf
│   │   └── ...
│   └── notes/
│       ├── attention-2017.md
│       └── ...
│
├── wiki/                   # Tier 2 — 개념 문서 (LLM이 직접 읽는 층)
│   ├── transformer.md
│   ├── rna-seq-pipeline.md
│   └── ...
│
└── overviews/              # Tier 3 — 합성·요약 (Overview 채굴)
    ├── attention-mechanisms-overview.md
    └── ...
```

### 3.1 각 Tier의 역할

**Tier 1 — sources/** (원본 보존)
- 논문 PDF는 `papers/`에 실제 복사 (symlink 절대 금지)
- YAML frontmatter가 달린 Markdown 노트
- Ingest 후 건드리지 않음 (불변 원칙)

**Tier 2 — wiki/** (지식 생산)
- 논문에서 추출한 개념 문서
- LLM이 직접 읽고 쿼리할 수 있는 구조
- 모든 콘텐츠 **영어** (§5 언어 정책)
- Obsidian `[[wikilinks]]` 형식으로 상호 연결

**Tier 3 — overviews/** (지식 합성)
- 여러 wiki/ 문서를 Claude Code가 합성해 생성
- Grand Synthesis: "이 분야의 현재 지식 상태는?"
- 주기적으로 재생성 (지식 그래프 업데이트)

### 3.2 파일명 규칙

```
sources/notes/ : <first-author>-<year>-<keyword>.md
                  예) vaswani-2017-transformer.md

wiki/          : <concept-kebab-case>.md
                  예) self-attention-mechanism.md

overviews/     : <topic>-overview.md
                  예) attention-mechanisms-overview.md
```

---

## 4. YAML frontmatter 스키마

### 4.1 sources/notes/ 스키마

```yaml
---
title: "Attention Is All You Need"
authors: ["Vaswani, A.", "Shazeer, N.", "Parmar, N."]
year: 2017
venue: "NeurIPS"
doi: "10.48550/arXiv.1706.03762"
tags: [transformer, self-attention, nlp, sequence-modeling]
status: "read"           # read | skim | queued
added: 2026-04-15
summary: >
  Proposes the Transformer architecture based entirely on attention
  mechanisms, eliminating recurrence. Achieves SOTA on WMT14 En-De
  translation (28.4 BLEU).
key_findings:
  - Multi-head self-attention replaces RNN/CNN
  - Positional encoding preserves sequence order
  - Scales to 1M+ context with flash attention variants
limitations:
  - Quadratic attention complexity O(n²)
related: ["bert-2018.md", "gpt-2018.md"]
---
```

### 4.2 wiki/ 스키마

```yaml
---
title: "Self-Attention Mechanism"
category: "deep-learning"
tags: [attention, transformer, nlp]
created: 2026-04-15
updated: 2026-05-01
sources: ["vaswani-2017-transformer.md", "bahdanau-2015-attention.md"]
---
```

### 4.3 스키마 활용 전략

- `status: queued` 논문을 Paper Monitor가 자동 감지 → Ingest 큐
- `tags` 기반으로 Overview 자동 그룹핑
- `related` 링크가 Obsidian 그래프 뷰의 엣지가 됨

---

## 5. CLAUDE.md 거버넌스 규칙

Claude Code가 위키를 다룰 때 반드시 지켜야 할 원칙을 `CLAUDE.md`에 명시한다.

```markdown
# Wiki Operating Rules

## Highest Priority
Answer ONLY based on papers in sources/ and wiki/.
Do NOT make up citations or supplement with general knowledge.

## Web Access — PROHIBITED
- WebSearch and WebFetch are FORBIDDEN
- Only use tools explicitly permitted by the user
- If information is insufficient: answer "I don't know based on current wiki"

## Language Policy
- Wiki content: English only
- Conversation with user: Korean OK
- Paper writing / RAG-shared terms: English

## File & Git Rules
- PDFs must be physically copied to papers/ (NO symlinks, NO absolute paths)
- Do NOT create git worktrees
- Do NOT commit directly to main branch
- Branch per task: feature/ingest-<paper-slug>

## Obsidian Compatibility
- Use [[wikilinks]] format for cross-references
- Standard Markdown + Canvas only
- YAML frontmatter must conform to schema in CLAUDE.md §schema
```

### 5.1 웹검색 금지의 이유

RAG 없이 파일시스템만 쓰는 LLM-Wiki의 핵심 원칙: **답변 근거가 항상 추적 가능해야 한다**.

- WebSearch 허용 시 → "어디서 온 정보인가?" 추적 불가
- "불충분하면 없다고 답해" → 지식 공백이 가시화 → 다음 Ingest 우선순위 결정

```markdown
# CLAUDE.md에 추가할 웹 접근 차단 예시
## Tool Restrictions
allowed_tools:
  - Read
  - Write
  - Edit
  - Bash(git *)
  - Bash(grep *)
  - Bash(find *)
forbidden_tools:
  - WebSearch
  - WebFetch
```

---

## 6. Paper Monitor 자동화

새 논문을 위키에 자동으로 유입(Ingest)하는 파이프라인.

### 6.1 Ingest 파이프라인

```
[새 논문 PDF 도착]
       ↓
[sources/papers/에 복사]
       ↓
[Claude Code: PDF 읽기 + YAML 추출]
       ↓
[sources/notes/<slug>.md 생성]
       ↓
[wiki/<concept>.md 생성 또는 업데이트]
       ↓
[overviews/ 재채굴 트리거]
```

### 6.2 Paper Monitor 커맨드 예시

`.claude/commands/paper-monitor.md`:

```markdown
# Paper Monitor

새로 추가된 논문을 위키에 인제스트한다.

## 실행 조건
- sources/papers/에 노트가 없는 PDF 발견 시 자동 실행
- 또는 `/project:paper-monitor` 수동 호출

## 작업 순서
1. `find sources/papers/ -name "*.pdf"` 로 PDF 목록 확인
2. 대응하는 `sources/notes/*.md`가 없는 PDF 식별
3. 각 PDF에 대해:
   a. PDF 읽기 → title, authors, year, venue, DOI 추출
   b. `sources/notes/<first-author>-<year>-<keyword>.md` 생성
   c. key_findings, summary, limitations 추출 (최대 5개 bullet)
   d. 관련 wiki/ 페이지 링크 추가
4. 변경된 wiki/ 파일의 상위 overview 재채굴 여부 확인
5. 결과 요약 보고

## 제약
- WebSearch 금지: PDF 내용만 근거
- 불확실한 필드는 "unknown"으로 기재, 추측 금지
- 한 번에 최대 5편 (긴 작업은 나눠서)
```

### 6.3 Headless 자동화 (CI/CD 연동)

```bash
# 새 PDF가 papers/에 추가될 때마다 GitHub Actions로 실행
claude -p "$(cat .claude/commands/paper-monitor.md)" \
  --allowedTools "Read,Write,Edit,Bash(find *),Bash(git *)" \
  --output-format json \
  | jq '.result'
```

### 6.4 Overview 채굴 커맨드

```markdown
# Overview Miner

특정 태그의 wiki/ 페이지를 합성해 overviews/ 문서를 생성한다.

## 호출법
/project:overview-miner <tag>
예) /project:overview-miner transformer

## 작업
1. `grep -rl "tags:.*<tag>" wiki/` 로 관련 페이지 수집
2. 각 페이지의 key points 추출
3. overviews/<tag>-overview.md 생성:
   - Current state of knowledge
   - Key papers and findings
   - Open questions
   - Contradictions / debates
```

---

## 7. Claude Code 연동 실전

### 7.1 위키 초기 세팅

```bash
# 1. 위키 디렉토리 구조 생성
mkdir -p my-wiki/{sources/{papers,notes},wiki,overviews,.claude/commands}
cd my-wiki && git init

# 2. CLAUDE.md 작성 (§5 내용 기반)
# 3. Paper Monitor 커맨드 등록
# 4. 첫 번째 논문 Ingest

claude -p "sources/papers/에 있는 PDF를 Ingest해 sources/notes/에 YAML 노트를 만들어줘" \
  --allowedTools "Read,Write,Bash(find *),Bash(git *)"
```

### 7.2 일상 쿼리 패턴

```bash
# 특정 개념 쿼리
claude "transformer의 positional encoding 방식을 설명해줘. wiki/ 기반으로만"

# 논문 비교
claude "vaswani-2017과 devlin-2018의 접근 방식 차이를 sources/notes/ 기반으로 비교해줘"

# Grand Synthesis
claude "wiki/에 있는 RNA-seq 관련 페이지를 읽고 현재 우리 위키의 지식 상태를 요약해줘"
```

### 7.3 학생·협업자 온보딩

연구팀 단위 운영 시:

```markdown
# team-wiki-onboarding.md (커맨드)

1. wiki/ 디렉토리 구조 확인
2. CLAUDE.md 거버넌스 규칙 숙지 확인 (WebSearch 금지 이해)
3. YAML frontmatter 스키마 템플릿 복사
4. 첫 논문 Ingest 실습
5. 기존 wiki/ 페이지 하나 쿼리 실습
```

### 7.4 Notion / Gmail 연동

Claude Code의 MCP를 통한 외부 연동:

```bash
# Notion에서 읽기 목록 가져오기 → papers/ 큐에 추가
claude "Notion의 'Papers to Read' DB에서 이번 주 추가된 항목을 
sources/papers/ Ingest 큐에 추가해줘"

# Gmail 논문 알림 처리
claude "오늘 Gmail의 PubMed 알림에서 RNA-seq 관련 논문 제목과 DOI를 추출해줘"
```

---

## 8. Karpathy 관점 — 지식 생산 플랫폼으로

> 안준용 교수의 블로그 노트 "Karpathy의 LLM Wiki" 요약 — RAG에서 지식 생산으로의 본질적 전환.
> 원문: `joonanlab.github.io/notes/karpathy-llm-wiki`

### 8.1 기존 RAG의 한계

"관련 자료를 잘 찾아오는 시스템"은 연구자의 개인화된 지식 구조와 자연스럽게 연결되지 않는다. 찾은 자료가 내 머리 속 구조와 별개.

### 8.2 LLM-Wiki의 차별성

자료 검색 중심에서 벗어나, **"자료를 읽고 정리하고 연결하는 과정 자체를 LLM에 맡기며"** 개념 문서로 풀어 위키 구조를 만들어 간다.

```
RAG = "정보를 찾는 시스템"
LLM-Wiki = "지식을 만들어가는 시스템"
```

### 8.3 본질적 전환

이것은 연구자의 본질적 작업 — 스스로 질문하고 문헌을 연결해 사고체계를 구축하는 일 — 과 정확히 맞닿아 있다.

→ 위키는 연구자와 AI가 함께 성장하는 **"Co-Scientist 플랫폼"**이다.

### 8.4 핵심 정리

| | RAG | LLM-Wiki |
|--|-----|----------|
| 활동 유형 | 검색 | 지식 생산 |
| 시간 관계 | 그때그때 꺼내 쓰기 | 시간-복리 축적 |
| 출력 | 답변 | 답변 + 지식 그래프 성장 |
| AI 역할 | 검색 도우미 | Co-Scientist |

---

## 부록 A — 최소 시작 템플릿

```bash
# 30분 안에 시작하는 LLM-Wiki

mkdir my-wiki && cd my-wiki
git init

# 디렉토리
mkdir -p sources/{papers,notes} wiki overviews .claude/commands

# CLAUDE.md 최소 버전
cat > CLAUDE.md << 'EOF'
# Wiki Rules
- Answer only from sources/ and wiki/
- WebSearch: FORBIDDEN
- Language: wiki=English, chat=Korean OK
- PDFs: copy to sources/papers/ (no symlinks)
EOF

# 첫 논문 추가
cp ~/Downloads/some-paper.pdf sources/papers/

# Ingest
claude "sources/papers/ 의 PDF를 읽고 sources/notes/에 YAML 노트를 만들어줘" \
  --allowedTools "Read,Write,Bash(find *)"
```

---

## 부록 B — 관련 가이드 링크

- 비용 효율 측정 → [claude-code-비용효율-측정-프레임워크.md](claude-code-비용효율-측정-프레임워크.md)
- CLAUDE.md 작성법 → [claude-code-CLAUDE-md-실전-작성법.md](claude-code-CLAUDE-md-실전-작성법.md)
- 커맨드 등록·관리 → [claude-code-통제센터-해부도.md](claude-code-통제센터-해부도.md)
- 하네스 구성 → [claude-code-harness-추천구성.md](claude-code-harness-추천구성.md)

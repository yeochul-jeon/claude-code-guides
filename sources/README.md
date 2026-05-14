# Sources — 원본 자료 인덱스

| 항목 | 내용 |
|------|------|
| 생성일 | 2026-05-14 |
| 변경일 | 2026-05-14 |
| 관리자 | 전여철 |

---

## 목차

1. [디렉토리 목적](#디렉토리-목적)
2. [명명 규칙](#명명-규칙)
3. [파일 목록](#파일-목록)
4. [주의사항](#주의사항)

---

## 디렉토리 목적

마크다운 가이드(`guides/`) 작성의 **참고/아카이브용 원본 자료**를 보관합니다.

```
sources/
├── README.md      ← 이 파일 (인덱스)
├── pdf/           ← PDF 원본
├── docx/          ← Word 문서 원본
└── _drafts/       ← 로컬 작업 임시 폴더 (.gitignore 처리)
```

---

## 명명 규칙

- 형식: `<주제>[-<연도>].pdf`
- 소문자, 한글/공백 대신 하이픈 사용
- 원제(한글)는 아래 파일 목록 표의 "원제" 컬럼에 기재
- 예시: `prompt-engineering-2025.pdf`, `claude-code-best-practices-2026.pdf`

---

## 파일 목록

> 파일을 추가할 때마다 아래 표에 행을 추가하세요. 출처/라이선스는 파악 시 기재.

### PDF (`sources/pdf/`)

| 파일명 | 원제 | 출처 | 라이선스 | 추가일 |
|--------|------|------|:--------:|--------|
| `ai-efficiency-calculation-shift-yoo.pdf` | AI는 더 똑똑해졌지만, 효율을 계산하는 방식도 함께 달라졌다 | 유민수 개발자 | 미상 | 2026-05-14 |
| `ai-coding-assistant-control-center-guide.pdf` | AI코딩 어시스턴트 통제센터 구축가이드 | 미상 | 미상 | 2026-05-14 |
| `claude-design-system-prompt-analysis-yoo.pdf` | Claude Design 시스템 프롬프트 해부 공유하기 | 유민수 개발자 | 미상 | 2026-05-14 |
| `claude-code-100x-system.pdf` | Claude Code 100x System | 미상 | 미상 | 2026-05-14 |
| `harness-seminar-aispace-20260423.pdf` | Harness Seminar 20260423 AISpace | AISpace | 미상 | 2026-05-14 |
| `harness-engineering.pdf` | Harness Engineering | 미상 | 미상 | 2026-05-14 |
| `llm-wiki-personalized-knowledge-chain.pdf` | LLM-WIKI를 이용한 개인화 지식체인 구축 | 미상 | 미상 | 2026-05-14 |
| `fastcampus-vibecoding-harness-engineering-yoo.pdf` | [패스트캠퍼스] 바이브코딩 상급 노하우 - 하네스 엔지니어링 편 | 패스트캠퍼스 / 유민수 개발자 | 미상 | 2026-05-14 |
| `harness-engineering-seminar.pdf` | Harness Engineering Seminar | 미상 | 미상 | 2026-05-14 |
| `claude-code-10-step-roadmap.pdf` | 클로드코드 10단계 로드맵 | 미상 | 미상 | 2026-05-14 |
| `claude-code-best-practices.pdf` | 클로드코드 잘사용하기 | 미상 | 미상 | 2026-05-14 |
| `harness-engineering-practical-guide.pdf` | 하네스 엔지니어링 실무가이드 | 미상 | 미상 | 2026-05-14 |
| `harness-engineering-overview.pdf` | 하네스엔지니어링 | 미상 | 미상 | 2026-05-14 |

### DOCX (`sources/docx/`)

| 파일명 | 원제 | 출처 | 라이선스 | 추가일 |
|--------|------|------|:--------:|--------|
| `claude-playbook-all-in-one-v3.docx` | Claude Playbook All-in-One v3 | 미상 | 미상 | 2026-05-14 |

---

## 주의사항

- **단일 파일 100MB 초과 금지** — GitHub 파일 크기 제한. 초과 시 git-lfs 도입 재논의
- **사내 비공개 자료 금지** — 파일명에 `-internal` 또는 `-confidential` 접미사 사용 시 `.gitignore`에 의해 자동 제외됨
- **출처/라이선스 기재 권장** — 저작권 사고 방지를 위해 파악 시 위 표에 기록
- **커밋 분리** — 원본 자료 추가 커밋은 가이드 작성 커밋과 분리 (예: `docs: sources/pdf에 X 자료 추가`)

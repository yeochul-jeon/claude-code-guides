# Confluence CLI 설정 가이드 (2026)

| 항목 | 날짜 |
|------|------|
| 생성일 | 2026-03-12 |
| 변경일 | 2026-03-12 |

> CJ 사내 Confluence Server/DC에 **confluence-cli**를 연결하여 터미널에서 페이지를 읽고, 검색하고, 관리하는 방법을 단계별로 안내한다.

---

## 목차

1. [개요](#1-개요)
2. [사전 준비](#2-사전-준비)
3. [환경변수 설정](#3-환경변수-설정)
4. [연결 테스트](#4-연결-테스트)
5. [주요 사용법](#5-주요-사용법)
6. [트러블슈팅](#6-트러블슈팅)

---

## 1. 개요

### Confluence CLI란?

[confluence-cli](https://www.npmjs.com/package/confluence-cli)는 Atlassian Confluence의 REST API를 래핑한 CLI 도구다.
터미널에서 페이지 조회, 검색, 생성, 수정, 삭제, 첨부파일 관리까지 모두 수행할 수 있으며,
Claude Code의 Skill로 연동하면 AI 에이전트가 Confluence를 직접 읽고 쓸 수 있다.

### 사내 Confluence 환경

| 항목 | 값 |
|------|-----|
| 제품 | Confluence Server / Data Center |
| 도메인 | `cjics.cj.net` |
| API 경로 | `/confluence/rest/api` |
| 인증 방식 | **Bearer** (Personal Access Token) |

> **Cloud vs Server/DC 차이:** Atlassian Cloud(`*.atlassian.net`)는 `basic` 인증 + email이 필요하지만,
> 사내 Server/DC는 **PAT(Personal Access Token)** 기반의 `bearer` 인증을 사용하므로 email이 불필요하다.

---

## 2. 사전 준비

### 2.1 confluence-cli 설치

```bash
# npm 전역 설치
npm install -g confluence-cli

# 설치 확인
confluence --version
```

> Node.js가 필요하다. `asdf`, `nvm`, `fnm` 등 버전 관리 도구로 Node.js를 미리 설치하자.

### 2.2 PAT(Personal Access Token) 발급

1. 브라우저에서 사내 Confluence에 로그인한다.
2. 우측 상단 프로필 아이콘 > **개인 설정(Profile)** 클릭
3. 좌측 메뉴에서 **Personal Access Tokens** 선택
4. **Create token** 클릭
5. 토큰 이름을 입력하고 (예: `confluence-cli`), 만료일을 설정한 뒤 **Create** 클릭
6. 생성된 토큰 값을 **즉시 복사** (이후 다시 확인 불가)

> **보안 주의:** 토큰은 비밀번호와 동일하게 취급한다. Git에 커밋하거나 Slack으로 공유하지 말 것.

---

## 3. 환경변수 설정

`~/.zshrc` (또는 `~/.bashrc`)에 아래 4개 환경변수를 추가한다.

```bash
# Confluence CLI
export CONFLUENCE_DOMAIN="cjics.cj.net"
export CONFLUENCE_API_PATH="/confluence/rest/api"
export CONFLUENCE_AUTH_TYPE="bearer"
export CONFLUENCE_API_TOKEN="your-personal-access-token"
```

### 각 환경변수 역할

| 변수 | 설명 | 예시 |
|------|------|------|
| `CONFLUENCE_DOMAIN` | Confluence 서버 호스트명 | `cjics.cj.net` |
| `CONFLUENCE_API_PATH` | REST API 기본 경로. Server/DC는 `/confluence/rest/api`, Cloud는 `/wiki/rest/api` | `/confluence/rest/api` |
| `CONFLUENCE_AUTH_TYPE` | 인증 방식. Server/DC PAT는 `bearer`, Cloud는 `basic` | `bearer` |
| `CONFLUENCE_API_TOKEN` | PAT 토큰 값. **2.2에서 발급한 토큰을 붙여넣기** | `your-personal-access-token` |

> **bearer 인증은 email이 불필요하다.** `CONFLUENCE_EMAIL` 변수를 설정할 필요 없다.

---

## 4. 연결 테스트

### 4.1 환경변수 로드

```bash
source ~/.zshrc
```

### 4.2 스페이스 목록 조회

```bash
confluence spaces
```

### 4.3 정상 출력 예시

연결이 성공하면 접근 가능한 스페이스 목록이 출력된다:

```
Spaces:
  DEVOPS - DevOps 팀
  BACKEND - 백엔드 개발
  INFRA - 인프라 운영
  QA - QA 팀
  ...
```

> 위 목록은 예시이며, 실제 출력은 본인이 접근 가능한 스페이스에 따라 다르다.

---

## 5. 주요 사용법

### 5.1 스페이스 목록 조회

```bash
confluence spaces
```

### 5.2 페이지 검색

```bash
# 키워드 검색
confluence search "배포 가이드"

# 특정 스페이스 내 검색 (CQL 사용)
confluence search "type=page AND space=DEVOPS" --limit 20
```

### 5.3 페이지 제목으로 찾기

```bash
# 정확한 제목 또는 부분 일치
confluence find "API 가이드" --space BACKEND
```

### 5.4 페이지 읽기

```bash
# 텍스트 형식 (기본)
confluence read 123456789

# 마크다운 형식
confluence read 123456789 --format markdown
```

> Page ID는 Confluence URL의 `pageId=` 파라미터에서 확인하거나, `search`/`find` 결과에서 얻을 수 있다.

### 5.5 페이지 메타데이터 조회

```bash
confluence info 123456789
```

### 5.6 하위 페이지 트리 조회

```bash
# 트리 형식으로 보기
confluence children 123456789 --recursive --format tree --show-id
```

### 5.7 페이지 생성 (마크다운)

```bash
confluence create "새 페이지 제목" MYSPACE --content "# 제목" --format markdown
```

### 5.8 페이지 수정

```bash
confluence update 123456789 --file ./updated.md --format markdown
```

---

## 6. 트러블슈팅

### 네트워크/VPN 연결 실패

```
Error: connect ETIMEDOUT ...
```

사내 Confluence는 VPN 연결이 필요하다. VPN이 활성화되어 있는지 확인하자.

```bash
# 네트워크 연결 확인
ping cjics.cj.net
```

### 토큰 만료 또는 인증 실패

```
Error: 401 Unauthorized
```

PAT 토큰이 만료되었거나 잘못된 경우 발생한다.

1. Confluence 웹에서 **개인 설정 > Personal Access Tokens** 확인
2. 토큰이 만료되었으면 새로 발급
3. `~/.zshrc`의 `CONFLUENCE_API_TOKEN` 값을 교체
4. `source ~/.zshrc` 실행

### curl로 직접 API 테스트

confluence-cli 없이 API 연결을 직접 확인하는 방법:

```bash
curl -s -H "Authorization: Bearer $CONFLUENCE_API_TOKEN" \
  "https://cjics.cj.net/confluence/rest/api/space?limit=5" | python3 -m json.tool
```

정상 응답 시 JSON 형태의 스페이스 목록이 반환된다. 이 방법으로 토큰과 네트워크를 독립적으로 검증할 수 있다.

### `No configuration found` 에러

```bash
# 환경변수가 제대로 로드되었는지 확인
echo $CONFLUENCE_DOMAIN
echo $CONFLUENCE_AUTH_TYPE
```

값이 출력되지 않으면 `source ~/.zshrc`를 다시 실행하거나, `.zshrc`에 오타가 없는지 확인한다.

### Claude Code에서 Confluence Skill 사용

Claude Code에서 Confluence를 사용하려면 Skill을 설치해야 한다:

```bash
cd your-project
confluence install-skill
```

이 명령은 `.claude/skills/confluence/SKILL.md`를 생성하여 Claude Code가 confluence-cli 사용법을 자동으로 인식하게 한다.

# GitHub Copilot CLI 핸즈온 가이드 — 터미널에서 멀티 에이전트 개발하기

> **GitHub Copilot CLI**의 주요 기능(Custom Agent · Skill · MCP · Instructions · 슬래시 커맨드 ·
> Plan/Autopilot · 서브에이전트)을 **개념부터 실습까지** 한 문서로 익히는 가이드입니다. VS Code의
> Copilot Chat(Ask/Edit/Agent 모드)과 무엇이 다른지도 함께 다룹니다.
>
> 이 저장소의 `.github/` 설정과 `src/`의 Microsoft Agent Framework 예제를 **실습 대상**으로 사용합니다.
> **Azure 리소스나 Python 실행 없이도** 가이드를 완주할 수 있으며, 별도의
> [Microsoft Agent Framework 핸즈온 랩](https://github.com/junwoojeong100/agent-framework-labs)과는 독립적입니다.

이 문서는 세 부분입니다. **① 개념편**으로 무엇인지 이해하고 → **② 실습편**을 위에서 아래로 따라 하며
손에 익히고 → 필요할 때 **③ 레퍼런스편**에서 전체 기능을 찾아보세요.

## 목차

**① 개념편 — 이해하기**

- [1. Copilot CLI란?](#1-copilot-cli란)
- [2. VS Code Copilot과 무엇이 다른가](#2-vs-code-copilot과-무엇이-다른가)
- [3. 주요 기능 한눈에](#3-주요-기능-한눈에)

**② 실습편 — 따라하기** (약 45–60분, 준비물: Copilot 구독 · Node.js 22+)

- [실습 0. 설치 · 인증 · 첫 실행](#실습-0-설치--인증--첫-실행)
- [실습 1. 대화 · 파일 멘션 · 모드 전환](#실습-1-대화--파일-멘션--모드-전환)
- [실습 2. `.github/` 설정으로 Copilot 조종하기](#실습-2-github-설정으로-copilot-조종하기)
- [실습 3. Custom Agent로 멀티 에이전트 개발](#실습-3-custom-agent로-멀티-에이전트-개발)
- [실습 4. MCP로 외부 도구 연결](#실습-4-mcp로-외부-도구-연결)
- [실습 5. 바이브 코딩 — 설정만으로 코드 생성·리뷰](#실습-5-바이브-코딩--설정만으로-코드-생성리뷰)
- [실습 6. 자동화 — Plan · Autopilot · 서브에이전트 · 스케줄](#실습-6-자동화--plan--autopilot--서브에이전트--스케줄)
- [실습 7. 가드레일(AGENTS.md)로 안전하게 커밋·PR](#실습-7-가드레일agentsmd로-안전하게-커밋pr)

**③ 레퍼런스편 — 찾아보기**

- [슬래시 커맨드 전체](#슬래시-커맨드-전체)
- [키보드 단축키](#키보드-단축키)
- [CLI 실행 옵션(플래그)](#cli-실행-옵션플래그)
- [설정 파일·환경변수](#설정-파일환경변수)
- [Custom Agent·Skill 만들기](#custom-agentskill-만들기)

**부록**

- [트러블슈팅](#트러블슈팅)
- [더 알아보기](#더-알아보기)
- [예제 코드를 실제로 실행하고 싶다면](#예제-코드를-실제로-실행하고-싶다면)
- [프로젝트 구조](#프로젝트-구조)

---

# ① 개념편 — 이해하기

## 1. Copilot CLI란?

**GitHub Copilot CLI**는 **터미널에서 직접 AI 코딩 에이전트와 대화**하는 도구입니다. GitHub Copilot
coding agent와 동일한 에이전틱 하네스를 기반으로, 자연어 지시 → **계획 → 실행 → 검증** 루프를 돌며
코드를 빌드·디버그·리팩터링하고, 파일을 만들고, 명령을 실행하고, GitHub 이슈·PR과 상호작용합니다.

| 특징 | 설명 |
|------|------|
| **터미널 네이티브** | IDE 전환 없이 CLI에서 바로 작업. SSH·서버·CI 등 GUI 없는 환경에서도 동작 |
| **에이전틱** | 복잡한 작업을 스스로 계획·실행·검증. 서브에이전트로 작업을 병렬 위임 |
| **안전 우선** | 디렉토리 신뢰 + 파일 변경·명령 실행 **전 승인**. 신뢰 환경에서만 `--yolo`/샌드박스 |
| **확장 가능** | MCP 서버·커스텀 에이전트·스킬·LSP·플러그인으로 기능 확장 |
| **GitHub 통합** | 리포·이슈·PR을 자연어로 접근(GitHub MCP 기본 내장), `/delegate`로 클라우드에 위임 |
| **모델 선택** | `/model`로 Claude Sonnet/Opus·GPT-5 등 선택(또는 `auto`) |

### 안전 모델 — 신뢰와 승인

처음 폴더에서 `copilot`을 실행하면 **이 폴더를 신뢰하는지** 묻습니다(이번 세션만 / 항상 신뢰 / 종료).
이후 Copilot이 파일을 수정하거나 명령(`node`·`sed`·`rm` 등)을 실행하려 할 때마다 **승인을 요청**합니다.

| 승인 선택지 | 의미 |
|------------|------|
| **Yes** | 이번 한 번 허용 (다음에 또 물음) |
| **Yes, and approve … for the session** | 이 세션 동안 해당 도구를 다시 묻지 않음 |
| **No, and tell Copilot what to do** | 거부하고 다른 방식을 자연어로 지시 |

> 💡 매번 승인이 번거롭고 **신뢰할 수 있는 환경**이라면 `--yolo`(자동 승인) 또는 `/sandbox enable`
> (로컬 샌드박스)·`copilot --cloud`(클라우드 샌드박스)를 사용합니다.

## 2. VS Code Copilot과 무엇이 다른가

GitHub Copilot은 **VS Code(IDE)**와 **Copilot CLI(터미널)**에서 모두 쓸 수 있고, `.github/`
커스터마이징을 **상당 부분 공유**합니다. 하지만 작동 방식과 강점이 다릅니다.

### VS Code Copilot Chat의 3가지 모드

| 모드 | 용도 | 특징 |
|------|------|------|
| **Ask** | 질문·설명·조언 | 대화형 Q&A. 코드를 바꾸지 않고 답변·제안만 |
| **Edit** | 코드 변형·리팩터링 | 선택 영역/파일을 자연어로 수정, **diff 미리보기** 후 수락/거부 |
| **Agent** | 자율 작업 | 여러 파일에 걸친 다단계 작업을 계획·수정·테스트 실행까지 자동 수행 |

VS Code Agent 모드와 Copilot CLI는 **둘 다 에이전틱**(다단계 자율 작업)이라는 점에서 가장 가깝습니다.
차이는 *어디서, 어떻게* 동작하느냐입니다.

### 비교표

| 항목 | 🖥️ VS Code Copilot Chat | 💻 Copilot CLI |
|------|---|---|
| **실행 위치** | 에디터(GUI) 내 채팅·인라인 | 터미널 — SSH·서버·CI·헤드리스 가능 |
| **상호작용 모드** | Ask · Edit · Agent (모드 선택) | 단일 대화 + `Shift+Tab`로 Interactive ↔ Plan, `--autopilot` |
| **공유 설정** | `copilot-instructions.md`, `instructions/`, `skills/`, `prompts/`, `agents/` | 위 + **`AGENTS.md`·`CLAUDE.md`·`GEMINI.md`** 추가 인식 |
| **에이전트 호출** | 에이전트 피커 | `/agent` · `--agent <name>` · 자연어로 이름 언급 |
| **프롬프트 파일** | `/프롬프트명` (채팅) | 직접 호출 없음 → 내용을 자연어로 요청 |
| **MCP 서버** | 설정 기반 자동 연결 | `/mcp`로 관리, `/mcp add`로 추가 |
| **자동화** | Agent 모드 | `--autopilot`·`/fleet`(병렬 서브에이전트)·`/every`·`/after`(스케줄)·CI 스크립트 |
| **클라우드 위임** | Coding Agent에 이슈 할당 | `/delegate`로 세션을 GitHub에 보내 PR 생성 |
| **변경 확인 / 리뷰** | Git 패널 / 리뷰 에이전트 | `/diff` · `/review` 명령 |

> 🔑 **핵심**: `.github/copilot-instructions.md`와 `instructions/*`는 **IDE와 CLI 양쪽에서 자동 적용**
> 됩니다. 한 번 잘 만들어 두면 VS Code에서 코딩하든 CLI에서 자동화하든 동일한 규칙이 일관되게 반영됩니다.

## 3. 주요 기능 한눈에

아래 기능들은 [실습편](#실습-0-설치--인증--첫-실행)에서 직접 써 보고, [레퍼런스편](#슬래시-커맨드-전체)에서
전체 목록을 확인할 수 있습니다.

| 기능 | 무엇인가 | 어떻게 |
|------|----------|--------|
| **슬래시 커맨드** | 세션 제어 명령 | `/help`로 전체 보기 — `/plan`·`/model`·`/mcp`·`/agent`·`/diff`·`/review` 등 |
| **멘션** | 입력 보조 | `@`파일 · `#`이슈/PR · `!`로컬 셸 명령 직접 실행 |
| **모드** | 진행 방식 | Interactive(단계 승인) · Plan(먼저 계획) · Autopilot(끝까지 자동) |
| **Custom Agent** | 역할·도구가 제한된 전용 에이전트 | 내장(Explore·Task·Research 등) + `.github/agents/*.agent.md` 커스텀, `--agent`로 실행 |
| **Skill** | 주입하는 전문 지식·패턴 묶음 | `.github/skills/*/SKILL.md` — 관련 작업 감지 시 자동 로드, `/skills` 관리 |
| **Instructions** | 항상/조건부 적용 규칙 | `copilot-instructions.md`(전역) + `instructions/*`(`applyTo` 글롭) + `AGENTS.md` |
| **MCP 서버** | 외부 시스템을 도구로 연결 | GitHub MCP 기본 내장, `.copilot/mcp-config.json`로 추가(Azure·Learn 등) |
| **LSP** | 코드 인텔리전스 | `.github/lsp.json` — go-to-definition·hover·진단 |
| **서브에이전트** | 작업 병렬 위임 | 모델이 자동 위임하거나 `/fleet`로 병렬 실행, `/tasks`로 관리 |
| **세션/컨텍스트** | 대화 관리 | `/compact`·`/context`·`/usage`·`/resume`·`copilot --continue`·`/share`·`/memory` |
| **자동화** | 손 안 대고 진행 | `--autopilot`·`/delegate`(클라우드 PR)·`/every`·`/after`(스케줄) |
| **코드 작업** | 개발 보조 | `/diff`·`/review`·`/pr`·`/research`·`/ide` |

---

# ② 실습편 — 따라하기

> 👉 각 실습의 코드 블록을 **위에서 아래로** 그대로 입력하고, 끝의 ✅ **확인**으로 점검한 뒤 다음으로
> 넘어가세요. 세션 프롬프트는 `>`로 표시합니다.

## 실습 0. 설치 · 인증 · 첫 실행

### 1) 설치하고 버전 확인

```bash
# 전 플랫폼 (Node.js 22+ 필요)
npm install -g @github/copilot
# 또는 macOS/Linux: curl -fsSL https://gh.io/copilot-install | bash
# 또는: brew install copilot-cli   /   winget install GitHub.Copilot

copilot --version   # 1.0.x 출력
```

> 플랫폼별 설치 옵션(Homebrew·WinGet·프리릴리즈·`PREFIX` 커스텀 경로)은
> [CLI 실행 옵션·설치](#cli-실행-옵션플래그)를 참고하세요.

### 2) 실행 · 폴더 신뢰 · 로그인

```bash
# 이 저장소에서 실행해야 .github/·.copilot/ 설정을 함께 읽습니다
cd copilot-cli-labs
copilot
```

처음 실행하면 **이 폴더를 신뢰하는지** 묻습니다 → `Yes, proceed`를 선택합니다. 로그인이 안 되어
있으면 세션에서 `/login`을 입력하고 브라우저 인증을 완료합니다.

```text
> /login
# 브라우저가 열리고 device code 인증을 안내합니다. 완료하면 세션으로 돌아옵니다.
```

> 💡 PAT로 인증하려면 토큰을 환경변수로 두고 실행합니다(우선순위 `COPILOT_GITHUB_TOKEN > GH_TOKEN >
> GITHUB_TOKEN`). 자세히는 [설정 파일·환경변수](#설정-파일환경변수).

✅ **확인**: 배너가 뜨고 로그인 상태가 되면 완료입니다. 막히면 [트러블슈팅](#트러블슈팅)을 보세요.

## 실습 1. 대화 · 파일 멘션 · 모드 전환

세션 프롬프트에 자연어로 입력합니다.

```text
> 이 저장소의 구조를 한 문단으로 설명해줘
```

`@`로 특정 파일을, `#`로 이슈/PR을, `!`로 로컬 셸 명령을 바로 실행할 수 있습니다.

```text
> @src/01_single_agent.py 이 파일이 하는 일을 설명해줘
> !ls -la src
```

모드와 모델을 바꿔 봅니다.

| 동작 | 방법 |
|------|------|
| Plan(계획 우선) ↔ Interactive 전환 | `Shift+Tab` |
| 모델 변경 | `/model` (Claude Sonnet/Opus·GPT-5 등, `auto` 가능) |
| 추론 과정 표시 토글 | `Ctrl+T` |
| 전체 슬래시 커맨드 | `/help` |

```text
> /plan
> 동시 워크플로우에 비용 리뷰 에이전트를 추가하려면 어떤 순서로 해야 할지 계획만 세워줘
```

Plan 모드에서는 코드를 바꾸기 전에 **구현 계획**을 먼저 제안합니다.

✅ **확인**: 파일 멘션(`@`)에 대한 한국어 설명을 받고, `Shift+Tab`으로 Plan↔Interactive를 오갈 수
있으면 완료입니다.

## 실습 2. `.github/` 설정으로 Copilot 조종하기

Copilot은 작업 디렉토리의 `.github/` 설정과 `AGENTS.md`를 읽어 **동작 방식을 바꿉니다.** 이 저장소
구성은 다음과 같습니다.

```text
.github/
├── copilot-instructions.md   # 전역 페르소나·코딩 스타일·프로젝트 규칙
├── instructions/             # 경로/언어별 세부 규칙 (applyTo 글롭)
│   ├── python.instructions.md   azure.instructions.md
│   ├── korean.instructions.md   git-commit.instructions.md
├── prompts/                  # 재사용 프롬프트 (add-agent · review-code)
├── agents/                   # 커스텀 에이전트 7개 (--agent <name>)
└── skills/                   # SDK 사용법·패턴 주입 (agent-framework-codegen)
```

| 구성요소 | 역할 | CLI에서 |
|----------|------|---------|
| `copilot-instructions.md` | 항상 적용되는 전역 규칙 | ✅ 자동 |
| `instructions/*` | `applyTo` 글롭으로 특정 파일/언어에만 적용 | ✅ 자동 |
| `skills/*` | SDK 사용법·패턴을 "교육" | ✅ 자동 로드 + `/skills` |
| `prompts/*` | 반복 작업 템플릿 | 자연어로 풀어서 요청 |
| `agents/*` | 역할별 에이전트 | `/agent` · `--agent` |

세션에서 무엇이 로드됐는지 직접 확인합니다.

```text
> !cat .github/copilot-instructions.md     # 전역 규칙 직접 열기
> /env                                      # 로드된 인스트럭션·스킬·에이전트·MCP 확인
> 지금 적용 중인 코딩 규칙과 커밋 메시지 규칙을 요약해줘
```

✅ **확인**: `/env`에 `copilot-instructions.md`·`instructions/*`가 보이고, 마지막 질문에 Copilot이
"커밋 메시지는 영문 Conventional Commits" 같은 **이 저장소의 규칙**을 답하면 성공입니다. 같은 질문도
`.github/` 설정에 따라 답이 달라진다는 것을 확인한 것입니다.

> 📄 `SKILL.md`·`*.agent.md` 형식과 재사용 범위는 [Custom Agent·Skill 만들기](#custom-agentskill-만들기) 참고.

## 실습 3. Custom Agent로 멀티 에이전트 개발

**Custom Agent**는 역할·도구가 제한된 전용 Copilot입니다. CLI에는 **내장 에이전트**가 있고, 저장소에
**커스텀 에이전트**를 추가할 수 있습니다.

### 내장 에이전트 (CLI 기본 제공)

| 에이전트 | 역할 |
|---------|------|
| **Explore** | 빠른 코드베이스 분석 — 메인 컨텍스트를 늘리지 않고 질문 |
| **Task** | 테스트·빌드 실행 — 성공 시 요약, 실패 시 전체 출력 |
| **General purpose** | 복잡한 다단계 작업을 별도 컨텍스트에서 처리 |
| **Code review** | 변경점 리뷰 — 진짜 문제만 표면화 |
| **Research** | 코드·관련 리포·웹을 아우른 심층 리서치(인용 포함) |
| **Rubber duck** | 건설적 비평 — Copilot이 필요 시 자동 호출 |

> 모델은 더 효과적이라 판단하면 작업을 **서브에이전트**에 자동 위임합니다. Rubber duck처럼 일부는
> 사용자가 직접 부르지 않아도 백그라운드에서 자동 활용됩니다.

### 이 저장소의 커스텀 에이전트 7개

```bash
# 오케스트레이터 — 요청 분석 후 최적 협업 패턴 자동 선택
copilot --agent orchestrator --yolo

# 4가지 협업 패턴
copilot --agent planner_executor --yolo    # 📐 계획-실행
copilot --agent debate_critic --yolo       # ⚔️ 토론-비평
copilot --agent generator_evaluator --yolo # ⚡ 생성-평가
copilot --agent code_generation --yolo     # 🏗️ 코드 생성

# 단독 전문 에이전트
copilot --agent reviewer                    # 코드 리뷰 (읽기 전용)
copilot --agent debugger                    # 환경/런타임 진단
```

`orchestrator`는 요청 의도를 분석해 패턴을 자동 선택합니다.

| 사용자 의도 | 선택 패턴 |
|------------|----------|
| "구현해줘", "셋업해줘", "마이그레이션" | 📐 Planner-Executor |
| "비교해줘", "장단점", "뭐가 나을까" | ⚔️ Debate & Critic |
| "생성해줘", "리뷰해줘", "개선해줘" | ⚡ Generator-Evaluator |
| "설계하고 구현해줘", "코드 작성하고 리뷰해줘" | 🏗️ Code Generation |

세션 안에서 `/agent`로 에이전트를 고르거나, 자연어로 이름을 언급해도 됩니다.

```text
> /agent
> reviewer 에이전트로 src/04_concurrent_workflow.py를 검토해줘
```

✅ **확인**: `/agent` 목록에 7개 에이전트가 보이고, `reviewer`가 읽기 전용으로 코드 리뷰를 내놓으면
완료입니다. (패턴별 팀 구성·협업 흐름은 [Custom Agent·Skill 만들기](#custom-agentskill-만들기) 참고)

## 실습 4. MCP로 외부 도구 연결

**MCP(Model Context Protocol)** 서버를 붙이면 Copilot이 외부 시스템을 **도구**로 사용합니다. Copilot은
**GitHub MCP를 기본 내장**하며, 이 저장소는 `.copilot/mcp-config.json`에 3개 서버를 추가로 설정합니다.

```json
{
  "mcpServers": {
    "github":        { "type": "http",  "url": "https://api.githubcopilot.com/mcp/",
                       "headers": { "Authorization": "Bearer ${GITHUB_PERSONAL_ACCESS_TOKEN}" }, "tools": ["*"] },
    "azure":         { "type": "local", "command": "npx",
                       "args": ["-y", "@azure/mcp@latest", "server", "start"], "tools": ["*"] },
    "microsoftLearn":{ "type": "http",  "url": "https://learn.microsoft.com/api/mcp", "tools": ["*"] }
  }
}
```

| 서버 | 용도 | 인증 |
|------|------|------|
| **github** | 이슈·PR·리포 탐색/조작 | PAT — `GITHUB_PERSONAL_ACCESS_TOKEN` |
| **azure** | 구독 내 Azure 리소스 조회·관리 | `az login` 세션 |
| **microsoftLearn** | Microsoft/Azure 공식 문서·코드 샘플 검색 | 불필요 |

```bash
# (선택) github·azure 서버를 쓸 때만 인증
export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxx
az login

copilot
> /mcp        # 등록된 서버 목록·상태 확인 ( /mcp add 로 새 서버 추가 )
```

```text
> /mcp
  github          http    ✓ connected
  azure           local   ✓ connected
  microsoftLearn  http    ✓ connected
```

> 💡 PAT·Azure 없이도 **인증이 필요 없는 `microsoftLearn`** 서버만으로 이 실습을 진행할 수 있습니다.
> `tools: ["*"]`는 모든 도구를 허용하므로, 특히 `azure`는 **조회뿐 아니라 변경/삭제**도 가능합니다
> (실제 범위는 `az login` 계정의 RBAC로 제한되고 실행 전 승인을 받습니다). 읽기 전용만 노출하려면
> `tools`를 도구명으로 좁히세요.

✅ **확인**: `/mcp`에 서버가 `connected`로 보이면 완료입니다.

## 실습 5. 바이브 코딩 — 설정만으로 코드 생성·리뷰

**바이브 코딩**은 손으로 코드를 쓰는 대신 `.github/` 설정(instructions·prompts·skills)으로 의도를
정의하고 Copilot이 코드를 생성하게 하는 방식입니다.

```text
┌──────────────────────────────────────────────────────────────┐
│ 1. copilot 실행 → .github/ 설정 자동 인식                     │
│ 2. /plan 으로 구현 계획 (선택)                                 │
│ 3. 자연어로 생성: "UX 리뷰 에이전트를 동시 워크플로우에 추가해줘" │
│ 4. /diff 로 변경 확인 → copilot --agent reviewer 로 리뷰       │
│ 5. 커밋 또는 /delegate 로 PR 생성 위임                         │
└──────────────────────────────────────────────────────────────┘
```

저장소의 `agent-framework-codegen` 스킬이 import 경로·async·instructions 규칙을 자동 주입하므로,
자연어 요청만으로 규칙에 맞는 코드가 생성됩니다.

```text
> 동시 워크플로우(04_concurrent_workflow.py)에 'UX 리뷰' 전문 에이전트를 추가해줘
> /diff                                  # 변경사항 확인
> reviewer 에이전트로 방금 변경을 검토해줘
```

(선택, Python 설치 시) 문법만 검증 — Azure 불필요:

```bash
python -m py_compile src/04_concurrent_workflow.py   # 오류 없으면 아무 출력 없이 종료(정상)
```

✅ **확인**: `.github/` 설정만으로 규칙에 맞는 새 에이전트 코드를 생성·리뷰하게 만들 수 있으면 이
실습의 목표를 달성한 것입니다. (실제 실행은 선택 — [예제 실행](#예제-코드를-실제로-실행하고-싶다면))

## 실습 6. 자동화 — Plan · Autopilot · 서브에이전트 · 스케줄

손을 덜 대고 작업을 끝까지 진행시키는 방법들입니다.

| 기능 | 명령 | 설명 |
|------|------|------|
| **Plan 모드** | `Shift+Tab` 또는 `/plan` | 실행 전 구현 계획을 먼저 수립 |
| **Autopilot** | `copilot --autopilot` 또는 `/autopilot` | 매 단계 승인 없이 끝까지 자동 진행 |
| **병렬 서브에이전트** | `/fleet` | 여러 서브에이전트를 병렬 실행, `/tasks`로 관리 |
| **클라우드 위임** | `/delegate` | 세션을 GitHub에 보내 Copilot이 PR 생성 |
| **반복 스케줄** | `/every 1h <프롬프트>` | 지정 간격마다 프롬프트 자동 실행 |
| **1회 지연 실행** | `/after 30m <프롬프트>` | 지정 시간 뒤 한 번 실행 |
| **세션 재개** | `copilot --continue` / `/resume` | 직전/특정 세션을 컨텍스트째 이어서 |

```text
> /plan
> 새 RAG 예제를 추가하는 작업을 단계로 나눠줘
# 계획이 마음에 들면 Shift+Tab 으로 Interactive 전환 후 실행하거나, --autopilot 로 끝까지 자동 진행
```

> ⚠️ Autopilot·`--yolo`는 파일 변경·명령을 자동 실행합니다. **신뢰할 수 있는 환경**에서만 쓰고,
> 위험한 작업은 `/sandbox enable`(로컬 샌드박스)이나 `copilot --cloud`(클라우드)에서 시도하세요.

✅ **확인**: `/plan`으로 계획을 받고, `/fleet`·`/tasks`·`/every` 같은 자동화 명령이 무엇인지 이해했다면
완료입니다.

## 실습 7. 가드레일(AGENTS.md)로 안전하게 커밋·PR

루트의 [`AGENTS.md`](AGENTS.md)는 **모든** Copilot 에이전트가 git/외부 명령 실행 전에 따르는 안전
규칙입니다. CLI는 `AGENTS.md`를 자동 인식합니다.

| 규칙 | 내용 |
|------|------|
| **Rule 1** | 기능 브랜치 push 허용, **보호 브랜치(main) 직접 push·force push·`--all`/`--mirror` 금지** |
| **Rule 2** | 커밋/PR 메시지는 **영문 + Conventional Commits** (`feat:`·`fix:` …) |
| **Rule 3** | PR은 항상 `--base main` 명시 + `--draft`로 생성 |

```bash
# ✅ 허용
git checkout -b feat/add-cost-reviewer
git add . && git commit -m "feat: add cost reviewer agent to concurrent workflow"
git push -u origin feat/add-cost-reviewer
gh pr create --draft --base main --title "feat: add cost reviewer agent" --body "Summary in English."

# ❌ 금지
git checkout main && git push origin main      # 보호 브랜치 직접 push
git push --force-with-lease origin <branch>     # force push
```

✅ **확인**: 기능 브랜치에서 영문 Conventional Commits로 커밋하고 `--draft --base main`으로 PR을 만들
수 있으면 완료입니다. Copilot에게 커밋/PR을 맡겨도 이 가드레일을 따릅니다.

---

# ③ 레퍼런스편 — 찾아보기

## 슬래시 커맨드 전체

> 세션에서 `/help`(또는 `?`)로 언제든 확인할 수 있습니다.

**에이전트 환경**

| 커맨드 | 설명 |
|--------|------|
| `/init` | 리포지토리용 Copilot 인스트럭션 초기화 |
| `/agent` | 사용 가능한 에이전트 탐색·선택 |
| `/skills` | 스킬 관리 |
| `/mcp` | MCP 서버 설정 관리 (`/mcp add`로 추가) |
| `/plugin` | 플러그인·마켓플레이스 관리 |
| `/env` | 로드된 환경 확인(인스트럭션·MCP·스킬·에이전트·플러그인·LSP·확장) |

**모델·서브에이전트·자동화**

| 커맨드 | 설명 |
|--------|------|
| `/model` | 모델 선택 (`auto` 가능) |
| `/delegate` | 세션을 GitHub에 보내 Copilot이 PR 생성 |
| `/fleet` | 병렬 서브에이전트 실행 모드 |
| `/autopilot` | Autopilot 모드 토글 |
| `/tasks` · `/sidekicks` | 백그라운드 태스크 / 실행 중 서브에이전트 관리 |
| `/plan` | 코딩 전 구현 계획 작성 |
| `/every` · `/after` | 프롬프트 반복 스케줄 / 1회 지연 실행 |

**코드 작업**

| 커맨드 | 설명 |
|--------|------|
| `/diff` | 현재 디렉토리 변경사항 리뷰 |
| `/review` | 코드 리뷰 에이전트 실행 |
| `/pr` | 현재 브랜치의 PR 작업 |
| `/research` | GitHub·웹 소스를 활용한 심층 리서치 |
| `/ide` | IDE 워크스페이스 연결 |
| `/lsp` | 언어 서버 설정 관리 |
| `/terminal-setup` | 멀티라인 입력 설정 (Shift+Enter) |

**권한·샌드박스**

| 커맨드 | 설명 |
|--------|------|
| `/allow-all` | 모든 권한 활성화 (도구·경로·URL) |
| `/sandbox` | 로컬 샌드박스 토글 (`/sandbox enable`) |
| `/add-dir` · `/list-dirs` | 파일 접근 허용 디렉토리 추가·표시 |
| `/cwd` · `/cd` | 작업 디렉토리 변경·표시 |
| `/reset-allowed-tools` | 허용된 도구 목록 초기화 |

**세션·컨텍스트**

| 커맨드 | 설명 |
|--------|------|
| `/resume` · `/rename` · `/new` | 세션 전환·이름 변경·새 대화 |
| `/session` · `/clear` | 세션 조회·관리 / 현재 세션 폐기 후 새로 시작 |
| `/context` · `/usage` | 토큰 사용량 시각화 / 세션 통계(AI Credits·시간·편집 라인·모델별 토큰) |
| `/compact` | 히스토리 요약으로 컨텍스트 절약 (95% 근접 시 자동) |
| `/share` · `/copy` | 세션 공유(md·HTML·Gist) / 마지막 응답 복사 |
| `/memory` | 세션 간 메모리 토글 |
| `/rewind` · `/undo` | 마지막 턴 되돌리기 + 파일 변경 복원 |
| `/remote` | GitHub 웹·모바일에서 원격 제어 토글 |
| `/chronicle` · `/search` | 세션 히스토리 도구 / 타임라인 검색 |

**도움말·기타**

| 커맨드 | 설명 |
|--------|------|
| `/help` · `?` | 도움말 |
| `/changelog` | 버전별 변경 로그 (`summarize`로 AI 요약) |
| `/update` · `/version` | 업데이트 / 버전 정보 |
| `/restart` · `/exit` | CLI 재시작(세션 유지) / 종료 |
| `/instructions` | 인스트럭션 파일 확인·토글 |
| `/voice` | 음성 입력(받아쓰기) 모드 |
| `/theme` · `/statusline` · `/footer` · `/streamer-mode` | 색상·상태줄·스트리머 모드 |
| `/experimental` | 실험적 기능 관리 |
| `/feedback` | 피드백 제출 |
| `/login` · `/logout` · `/user` | 로그인·로그아웃·GitHub 사용자 목록 관리 |
| `/ask` | 히스토리에 남기지 않는 빠른 곁가지 질문 |
| `/keep-alive` | 시스템 절전 방지 토글 |

## 키보드 단축키

**일반**

| 단축키 | 기능 | 단축키 | 기능 |
|--------|------|--------|------|
| `Shift+Tab` | 모드 전환 | `Ctrl+T` | 추론 과정 표시 토글 |
| `Ctrl+S` | 프롬프트 임시 저장/복원 | `Ctrl+Q` | 프롬프트 대기열에 추가 |
| `Ctrl+R` | 히스토리 역방향 검색 | `Ctrl+O`/`Ctrl+E` | 타임라인 확장 |
| `Ctrl+C` | 취소 (`×2` 종료) | `Esc` | 현재 작업 취소 |
| `Ctrl+D` | 종료 | `Ctrl+Z` | 일시 중단 |
| `Ctrl+L` | 화면 지우기 | `Ctrl+X → B` | 현재 작업 백그라운드로 |
| `Ctrl+X → O` | 최근 링크 열기 | `↑` `↓` | 명령 히스토리 |

**입력 편집**

| 단축키 | 기능 | 단축키 | 기능 |
|--------|------|--------|------|
| `Ctrl+A`/`Ctrl+E` | 줄 처음/끝 이동 | `Ctrl+H`/`Ctrl+W` | 이전 글자/단어 삭제 |
| `Ctrl+U`/`Ctrl+K` | 줄 시작/끝까지 삭제 | `Meta+←`/`→` | 단어 단위 이동 |
| `Shift+Enter` | 줄바꿈 삽입 | `Ctrl+G` | `$EDITOR`에서 편집 |

## CLI 실행 옵션(플래그)

```bash
copilot --agent orchestrator --autopilot --yolo
```

| 플래그 | 설명 |
|--------|------|
| `--agent <name>` | 특정 에이전트로 시작 |
| `--prompt "..."` | 비대화형으로 프롬프트 전달(스크립트·CI) |
| `--autopilot` | Autopilot으로 시작 |
| `--yolo` / `--allow-all` | 자동 승인 — 모든 도구·경로·URL 허용 |
| `--cloud` | 클라우드 샌드박스 세션으로 시작 |
| `--continue` | 가장 최근 로컬 세션 이어서 |
| `--resume` | 세션을 골라 재개 |
| `--banner` | 시작 배너 다시 표시 |
| `--experimental` | 실험적 기능 활성화 |

**설치(플랫폼별)**

```bash
# 스크립트 (macOS/Linux): 루트 설치는 | sudo bash, 버전·경로는 VERSION/PREFIX
curl -fsSL https://gh.io/copilot-install | bash
brew install copilot-cli            # @prerelease 로 프리릴리즈
winget install GitHub.Copilot       # .Prerelease 로 프리릴리즈
npm install -g @github/copilot       # @prerelease 로 프리릴리즈
```

## 설정 파일·환경변수

| 위치 | 설명 |
|------|------|
| `~/.copilot/settings.json` | CLI 설정 (`copilot help config`) |
| `~/.copilot/mcp-config.json` | 유저 레벨 MCP 서버 (리포는 `.copilot/mcp-config.json`) |
| `~/.copilot/lsp-config.json` | 유저 레벨 LSP (리포는 `.github/lsp.json`) |
| `~/.copilot/agents/` | 유저 레벨 커스텀 에이전트 |

| 환경변수 | 설명 |
|----------|------|
| `COPILOT_HOME` | 설정 디렉토리 경로 변경(기본 `~/.copilot`) |
| `COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN` | PAT 인증 (이 우선순위로 적용) |
| `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` | 추가 인스트럭션 디렉토리 |
| `GITHUB_PERSONAL_ACCESS_TOKEN` | `.copilot/mcp-config.json`의 github 서버 토큰 |

**인스트럭션 인식·우선순위** — Copilot은 아래 위치를 자동 인식합니다. 여러 파일이 있으면 **모두 동시
적용**되며, 충돌 시 단일 우선순위로 단정할 수 없습니다(조합에 따라 비결정적). 현재 적용은
`/instructions`로 확인합니다.

```text
AGENTS.md (git 루트 & cwd) · CLAUDE.md · GEMINI.md
.github/copilot-instructions.md
.github/instructions/**/*.instructions.md
~/.copilot/copilot-instructions.md  (+ COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
```

## Custom Agent·Skill 만들기

### 커스텀 에이전트 (`.github/agents/<이름>.agent.md`)

**YAML frontmatter + 본문 지시문** 구조입니다. 파일명에서 `.agent.md`를 뺀 부분이 에이전트 이름이
됩니다. 정의 위치에 따라 적용 범위가 다릅니다.

| 레벨 | 위치 | 범위 |
|------|------|------|
| 유저 | `~/.copilot/agents/` | 모든 프로젝트 |
| 리포 | `.github/agents/` | 현재 프로젝트 |
| 조직/엔터프라이즈 | `.github-private`의 `/agents` | 조직 전체 |

```markdown
---
name: 에이전트 표시 이름
description: 역할 요약 (필수 — Copilot이 자동 선택 시 참고)
tools: [read, search, edit, execute, agent, web]   # 생략=전체, []=없음
model: auto                                         # 선택
---

# My Agent
## 역할 (Role)
이 에이전트는 [구체적 역할]을 수행합니다.
## 규칙 (Rules)
1. 항상 [규칙]을 따릅니다.
## 워크플로우 (Workflow)
1. 요청 분석 → 2. [단계별 수행] → 3. 결과 문서화
```

호출은 3가지: `copilot --agent <name>` · 세션에서 `/agent` 선택 · 자연어로 이름 언급.

### 이 저장소의 4가지 협업 패턴

각 패턴은 여러 전문 에이전트가 역할을 분담하며, 모든 팀에 과정을 문서화하는 **Scribe**가 있습니다.

| 패턴 | 목적 | 핵심 루프 | 팀 구성 |
|---|---|---|---|
| 📐 **Planner-Executor** | 체계적 실행 | 계획→실행→검증 (3회 Revise) | Planner·Executor·Validator·Scribe |
| ⚔️ **Debate & Critic** | 최선의 결론 | 제안→반론→평가 (3 Rounds) | Proposer·Opponent·Critic·Synthesizer·Scribe |
| ⚡ **Generator-Evaluator** | 반복 품질 개선 | 생성→평가→개선 (3 Cycles) | Generator·Evaluator·Refiner·Scribe |
| 🏗️ **Code Generation** | 설계 기반 생성 | 설계→구현→리뷰 (3 Cycles) | Architect·Developer·Reviewer·Scribe |

### 스킬 (`.github/skills/<스킬명>/SKILL.md`)

에이전트가 특정 작업 시 참고하는 **전문 지식/절차 문서**입니다. `description`에 **언제 쓰는지**
(USE FOR / DO NOT USE FOR)를 적으면 Copilot이 관련 작업을 감지해 **자동 로드**(점진적 공개 → 토큰
절약)하고, `/skills`로 관리·직접 호출할 수 있습니다. 이 저장소 예시는
`agent-framework-codegen`(Agent Framework 코드 생성 패턴)입니다.

| 필드 | 설명 |
|------|------|
| `name` | 스킬 식별자 |
| `description` | 언제 쓰는지 — Copilot이 로드 여부 결정 |
| 본문 | SDK 패턴·예제 (관련 작업일 때만 로드) |

---

# 부록

## 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| `copilot: command not found` | 설치 경로가 PATH에 없음 | 재설치 후 새 터미널. `npm prefix -g`/`ls ~/.local/bin/copilot` 확인, 필요 시 `export PATH="$HOME/.local/bin:$PATH"` |
| 로그인 안내만 반복 | 인증 미완료 | `/login` 후 브라우저 인증(자동으로 안 열리면 표시 URL 수동 열기), 또는 `export GH_TOKEN=...`. 조직 정책 비활성화 시 관리자 확인 |
| `/mcp`에 서버 안 보임 | `.copilot/mcp-config.json` 미인식/JSON 오류 | 저장소 루트에서 실행, JSON 문법 확인 후 세션 재시작 |
| github 서버 인증 오류 | `GITHUB_PERSONAL_ACCESS_TOKEN` 미설정 | PAT를 `export` 하거나 `github` 블록 제거(기본 내장 사용) |
| azure 서버 연결 실패 | `az login` 세션 없음/만료 | `az login` 재실행, `az account show` 확인 |
| `--agent <name>` 안 됨 | 파일명/위치 불일치 | `ls .github/agents/*.agent.md` 확인, `/agent`로 목록 확인 |
| 응답 품질 저하 / 컨텍스트 초과 | 대화가 너무 김 | `/compact`·`/context`·`/clear` (95% 근접 시 자동 압축) |
| Node.js 버전 오류 | Node 22 미만 | `node -v` 확인 후 22+ 설치 (<https://nodejs.org>) |

## 더 알아보기

- [GitHub Copilot CLI 공식 문서](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) ·
  [사용 가이드](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli) ·
  [Copilot 플랜 및 가격](https://github.com/features/copilot/plans)
- [GitHub 멀티 계정 설정 가이드](docs/github-multi-account-setup.md) — 한 머신에서 Git 작업용 계정과
  Copilot 구독 계정을 분리해 사용하는 방법
- 세션에서 `copilot help config` · `copilot help environment` · `copilot help permissions`로 추가 정보 확인

## 예제 코드를 실제로 실행하고 싶다면

`src/`의 Microsoft Agent Framework 예제는 이 가이드에서 **바이브 코딩의 대상(예시 도메인)** 으로
포함됩니다. 생성·수정한 코드를 **실제로 실행**(= Azure·Python 필요)해 보려면, 단계별 실행 가이드가 있는
**[Microsoft Agent Framework 핸즈온 랩](https://github.com/junwoojeong100/agent-framework-labs)** 의 사전 준비를 따르세요.

```bash
python src/04_concurrent_workflow.py   # 실제 실행 (Azure·Python 필요, 선택)
```

## 프로젝트 구조

```
.
├── README.md                       # 이 가이드 (개념 + 실습 + 레퍼런스)
├── AGENTS.md                       # 에이전트 공통 가드레일 (push 금지·영문 커밋·PR 규칙)
├── .copilot/
│   └── mcp-config.json             # MCP 서버 설정 (github · azure · microsoftLearn)
├── .github/
│   ├── copilot-instructions.md     # 프로젝트 전역 인스트럭션
│   ├── instructions/               # python · azure · korean · git-commit 규칙
│   ├── prompts/                    # add-agent · review-code (재사용 프롬프트)
│   ├── agents/                     # orchestrator + 4 패턴 + reviewer · debugger (7개)
│   ├── skills/
│   │   └── agent-framework-codegen/SKILL.md   # MAF 코드 생성 패턴
│   └── workflows/
│       └── smoke.yml               # 예제 스크립트 바이트컴파일 스모크 CI
├── docs/                           # GitHub 멀티 계정 설정 가이드
└── src/                            # 바이브 코딩 예시 도메인 (Microsoft Agent Framework 예제)
```

---

## 라이선스

[MIT](LICENSE)

# GitHub Copilot CLI 핸즈온 랩 — CLI로 멀티 에이전트 개발 가속하기

> **GitHub Copilot CLI 자체를 개발 도구로 활용**하는 법(설치 · `.github/` 설정 · 멀티 에이전트
> 패턴 · 바이브 코딩 · 가드레일)을 단계별로 익히는 **자체 완결형 핸즈온 랩**입니다.
>
> 이 문서 **하나만으로** 완주할 수 있으며, [Microsoft Agent Framework 핸즈온 랩](https://github.com/junwoojeong100/agent-framework-labs)과는
> 서로 의존하지 않습니다. 같은 저장소의 Microsoft Agent Framework 코드(`src/`)를 **예시 도메인**으로
> 삼아 "Copilot에게 에이전트 코드를 생성·리뷰시키는" 흐름을 보여줍니다. **Azure 리소스나 Python 코드
> 실행 없이도** 이 랩을 완주할 수 있습니다.

이 문서는 두 부분으로 구성됩니다. 먼저 **핸즈온 랩(Part 1–5)** 을 위에서 아래로 따라가며 손에 익히고,
필요할 때 뒤쪽의 **레퍼런스(심화)** 에서 설치 옵션·슬래시 커맨드·설정 파일·에이전트 패턴의 상세를
찾아보세요.

## 🎯 이 랩을 마치면

1. Copilot CLI를 설치·인증해 터미널에서 대화하고,
2. `.github/` 설정으로 Copilot의 동작을 "조종"하며,
3. 커스텀 에이전트 + MCP 도구로 멀티 에이전트 개발을 수행하고,
4. 바이브 코딩으로 규칙에 맞는 코드를 생성·리뷰하고,
5. 가드레일(`AGENTS.md`)로 안전하게 커밋/PR 할 수 있습니다.

⏱️ 전체 예상 소요: 약 45–60분 (Azure/Python으로 실제 실행하는 부분은 선택)

👉 **진행 방법**: 각 Part의 코드 블록을 **위에서 아래로** 그대로 따라 입력하고, 끝에 있는 ✅ **확인**
체크포인트로 결과를 점검한 뒤 다음 Part로 넘어가세요.

---

## 사전 준비

| 도구 | 필수/선택 | 용도 | 설치 |
|------|-----------|------|------|
| **GitHub Copilot 구독** | 필수 | Copilot CLI 사용 권한 | <https://github.com/features/copilot> |
| **GitHub Copilot CLI** | 필수 | 터미널 AI 에이전트 | `npm install -g @github/copilot` |
| **Node.js 22+** | 필수 | CLI 런타임 (+ `npx`로 실행되는 MCP 서버) | <https://nodejs.org> |
| **이 저장소 클론** | 필수 | `.github/`·`.copilot/` 설정을 실습 대상으로 사용 | `git clone <repo>` |
| **GitHub PAT** | 선택 | `github` MCP 블록 사용 시에만 (Part 3) | <https://github.com/settings/tokens> |
| **Azure CLI + `az login`** | 선택 | `azure` MCP 서버 인증 시에만 (Part 3) | `az upgrade --yes` |

> 💡 **범위 안내**: 이 랩은 같은 저장소의 Microsoft Agent Framework 코드(`src/`)를 **예시 도메인**으로
> 삼아 "Copilot에게 에이전트 코드를 생성·리뷰시키는" 흐름을 보여줍니다. 다만 생성된 코드를 **실제로
> 실행(= Azure·Python 필요)하는 것은 선택**이며, CLI 학습 자체에는 필요하지 않습니다. 실행까지 해보고
> 싶다면 [Microsoft Agent Framework 핸즈온 랩](https://github.com/junwoojeong100/agent-framework-labs)의 사전 준비를 따르세요.

## 핵심 개념 — Copilot CLI를 조종하는 요소

| 기술 | 무엇인가 | 핵심 기능 | 장점 |
|------|----------|-----------|------|
| **GitHub Copilot CLI** | 터미널에서 동작하는 에이전틱 코딩 도구 | 자연어 지시 → 계획·실행·검증 루프, 슬래시 커맨드(`/plan`·`/fleet`·`/model`), MCP·커스텀 에이전트 확장 | IDE 없이 터미널·CI에서 동작, 명령 실행 전 승인으로 안전, 모델 자유 선택 |
| **Custom Agent**<br/>(`.github/agents/*.agent.md`) | 역할·도구가 제한된 전용 에이전트 | frontmatter로 `description`·`tools`·`model` 지정, `copilot --agent <name>` 실행 | 역할 격리(리뷰어=읽기전용)로 안전·집중, 재사용·팀 공유 |
| **Skill**<br/>(`.github/skills/*/SKILL.md`) | Copilot에 주입하는 전문 지식·패턴 묶음 | `description`으로 트리거, 필요 시에만 본문 로드(점진적 공개) | 정확한 SDK 호출 유도, 토큰 절약, 환각 감소 |
| **Instructions**<br/>(`.github/*instructions.md`) | 항상/조건부로 적용되는 규칙 | `copilot-instructions.md`(전역) + `instructions/*`(`applyTo` 글롭) | 일관된 스타일·규칙 자동 준수, 반복 지시 제거 |

## 목차

**핸즈온 랩**

- [Part 1. Copilot CLI 시작하기](#part-1-copilot-cli-시작하기)
- [Part 2. Copilot을 "조종"하는 `.github/` 설정](#part-2-copilot을-조종하는-github-설정)
- [Part 3. 멀티 에이전트 패턴으로 개발하기](#part-3-멀티-에이전트-패턴으로-개발하기)
- [Part 4. 바이브 코딩 — 설정만으로 코드 생성하기](#part-4-바이브-코딩--설정만으로-코드-생성하기)
- [Part 5. 가드레일 (AGENTS.md)](#part-5-가드레일-agentsmd)

**레퍼런스 (심화)**

- [레퍼런스 A. Copilot CLI 상세](#레퍼런스-a-copilot-cli-상세) — 설치·인증·기본 사용법·슬래시 커맨드·LSP·팁
- [레퍼런스 B. `.github/` 설정 상세](#레퍼런스-b-github-설정-상세) — SKILL·agent frontmatter·파일 유형·재사용 범위
- [레퍼런스 C. 멀티 에이전트 패턴 상세](#레퍼런스-c-멀티-에이전트-패턴-상세) — 에이전트 작성·호출·4가지 협업 패턴

**부록**

- [부록. 트러블슈팅](#부록-트러블슈팅)
- [참고: VS Code vs CLI](#참고-vs-code-vs-cli)
- [더 알아보기 (선택, 심화)](#더-알아보기-선택-심화)
- [예제 코드를 실제로 실행하고 싶다면](#예제-코드를-실제로-실행하고-싶다면)
- [프로젝트 구조](#프로젝트-구조)

---

## Part 1. Copilot CLI 시작하기

> 🎯 **목표**: Copilot CLI를 설치·인증하고 터미널에서 첫 대화를 나눠 봅니다.
> ⏱️ 예상 소요: 약 10분 · 준비물: GitHub Copilot 구독, Node.js 22+

### 1단계 — 설치하고 버전 확인

```bash
# 설치 (전 플랫폼, Node.js 22+ 필요)
npm install -g @github/copilot
# 또는 macOS/Linux: curl -fsSL https://gh.io/copilot-install | bash
# 또는: brew install copilot-cli   /   winget install GitHub.Copilot

# 설치 확인
copilot --version
```

설치가 끝나면 버전 번호가 출력됩니다.

```text
1.0.x
```

> 💡 플랫폼별 설치 옵션(Homebrew·WinGet·curl/wget 스크립트·프리릴리즈·`PREFIX` 커스텀 경로)은
> [레퍼런스 A — 설치 방법](#레퍼런스-a-copilot-cli-상세)을 참고하세요.

### 2단계 — 실행하고 로그인

```bash
# 이 저장소 디렉토리에서 실행해야 .github/·.copilot/ 설정을 함께 읽습니다
cd copilot-cli-labs
copilot
```

처음 실행하면 배너가 뜨고, 로그인이 안 되어 있으면 안내가 나옵니다. 세션 프롬프트(`>`)에
`/login`을 입력하세요.

```text
> /login
# 브라우저가 열리고 device code 인증을 안내합니다. 완료하면 세션으로 돌아옵니다.
```

> 💡 PAT로 인증하려면 `export GH_TOKEN=...` 후 `copilot`을 실행합니다. 토큰 종류·우선순위는
> [레퍼런스 A — 인증](#레퍼런스-a-copilot-cli-상세)을 참고하세요.

### 3단계 — 첫 대화 해보기

세션 프롬프트(`>`)에 자연어로 입력합니다.

```text
> 이 저장소의 구조를 한 문단으로 설명해줘
```

Copilot이 파일을 읽고(필요하면 실행 승인을 먼저 요청) 한국어로 요약해 줍니다. 파일 변경이나
명령 실행이 필요한 작업은 **실행 전에 항상 승인을 묻습니다**(기본 Interactive 모드).

### 4단계 — 모드 전환·모델 선택·종료

| 동작 | 방법 |
|------|------|
| Plan(계획 우선) ↔ Interactive 모드 전환 | `Shift+Tab` |
| 모델 변경 | `/model` (Claude Sonnet/Opus, GPT-5 등) |
| 전체 슬래시 커맨드 보기 | `/help` |
| 세션 종료 | `Ctrl+D` 또는 `Ctrl+C` 두 번 |

자주 쓰는 슬래시 커맨드:

```text
/plan      # 구현 계획 수립 (실행 전 설계)
/fleet     # 병렬 서브에이전트 실행
/model     # 모델 선택 (Claude Sonnet/Opus, GPT-5 등)
/diff      # 변경사항 리뷰
/mcp       # 등록된 MCP 서버 상태 확인
/env       # 로드된 인스트럭션·스킬·에이전트 확인
```

특성: **에이전트 코딩**(계획→실행→검증), **안전 실행**(명령 실행 전 승인, 신뢰 환경에서만 `--yolo`),
**MCP 확장**(외부 시스템을 도구로 연결), **커스텀 에이전트**(`copilot --agent <name>`).

> ✅ **확인**: `copilot` 세션이 열리고 첫 질문에 한국어 답변을 받았다면 Part 1 완료입니다.
> 막히면 [부록. 트러블슈팅](#부록-트러블슈팅)을 보세요.

> 💡 이 랩에서는 Copilot CLI에게 "이런 에이전트를 만들어줘"라고 지시하고, 생성된 코드를
> 검토·실행합니다. `src/`의 예제는 그 결과물의 완성본입니다.
>
> 📄 **더 알아보기**: 전체 슬래시 커맨드·키보드 단축키·CLI 실행 옵션·기본 사용법은
> [레퍼런스 A](#레퍼런스-a-copilot-cli-상세)에 정리되어 있습니다. IDE와의 차이는
> [참고: VS Code vs CLI](#참고-vs-code-vs-cli)를 보세요.

---

## Part 2. Copilot을 "조종"하는 `.github/` 설정

Copilot CLI/Chat는 작업 디렉토리의 `.github/` 설정과 `AGENTS.md`를 읽어 **동작 방식을 바꿉니다.**
이 저장소의 구성은 다음과 같습니다.

```text
.github/
├── copilot-instructions.md   # 전역 페르소나·코딩 스타일·프로젝트 규칙
├── instructions/             # 경로/언어별 세부 규칙 (applyTo 글롭)
│   ├── python.instructions.md
│   ├── azure.instructions.md
│   ├── korean.instructions.md
│   └── git-commit.instructions.md
├── prompts/                  # 재사용 프롬프트 (VS Code Chat에서 /프롬프트명)
├── agents/                   # 커스텀 에이전트 (copilot --agent <name>)
└── skills/                   # SDK 사용법·패턴 주입 (SKILL.md)
```

| 구성요소 | 역할 |
|----------|------|
| `copilot-instructions.md` | 프로젝트 전반에 항상 적용되는 규칙 |
| `instructions/*` | `applyTo` 글롭으로 특정 파일/언어에만 적용되는 규칙 |
| `prompts/*` | 반복 작업 템플릿. **VS Code Chat**에서 `/프롬프트명` 호출(CLI는 자연어로 동일 요청) |
| `agents/*` | `copilot --agent`로 실행하는 역할별 에이전트 |
| `skills/*` | SDK 사용법·패턴을 Copilot에 "교육"하는 전문 지식 |

### 직접 확인해 보기

`copilot` 세션을 연 상태에서 다음을 순서대로 해봅니다.

```text
1. (셸) 전역 규칙 파일을 직접 열어 봅니다 — ! 로 시작하면 로컬 셸에서 바로 실행됩니다
   > !cat .github/copilot-instructions.md
2. (CLI) 현재 세션에 무엇이 로드됐는지 확인합니다
   > /env
3. (CLI) 규칙이 실제로 적용되는지 물어봅니다
   > 지금 적용 중인 코딩 규칙과 커밋 메시지 규칙을 요약해줘
```

`/env` 출력에는 로드된 인스트럭션·스킬·커스텀 에이전트·MCP 서버가 함께 표시됩니다.

> ✅ **확인**: `/env`에 `copilot-instructions.md`와 `instructions/*`가 로드되어 있고, 3번 질문에
> Copilot이 "커밋 메시지는 영문 Conventional Commits" 같은 **이 저장소의 규칙**을 답하면 성공입니다.
> 즉, 같은 질문이라도 `.github/` 설정에 따라 답이 달라진다는 것을 직접 확인한 것입니다.

> 📄 **더 알아보기**: `SKILL.md`·`*.agent.md` frontmatter 구조, 파일 유형별 동작·선택 기준,
> 재사용 범위(공용 vs 프로젝트 전용), 전체 관계도는 [레퍼런스 B](#레퍼런스-b-github-설정-상세)를
> 참고하세요.

---

## Part 3. 멀티 에이전트 패턴으로 개발하기

> 이 Part는 **① 에이전트 정의(`.github/agents/`) → ② 에이전트가 쓸 도구 연결(MCP 서버) →
> ③ 둘을 합친 실습** 순서로 진행합니다. 에이전트는 "일꾼", MCP 서버는 "일꾼이 쓰는 연장"이라
> 같은 Part에서 함께 다룹니다.

`.github/agents/`에 역할별 에이전트를 정의하고 `copilot --agent <name>`으로 실행합니다.
이 저장소에는 **7개** 에이전트(오케스트레이터 + 4가지 협업 패턴 + reviewer·debugger)가 포함되어
있습니다.

```bash
# 오케스트레이터 — 요청 분석 후 최적 패턴 자동 선택
copilot --agent orchestrator --yolo

# 4가지 협업 패턴 에이전트 직접 실행
copilot --agent planner_executor --yolo    # 📐 계획-실행 패턴
copilot --agent debate_critic --yolo       # ⚔️ 토론-비평 패턴
copilot --agent generator_evaluator --yolo # ⚡ 생성-평가 패턴
copilot --agent code_generation --yolo     # 🏗️ 코드 생성 패턴

# 단독 전문 에이전트
copilot --agent reviewer                   # 코드 리뷰 (읽기 전용)
copilot --agent debugger                   # 환경/런타임 문제 진단
```

`orchestrator`는 요청을 분석해 4가지 협업 패턴 중 하나를 선택해 위임합니다:

| 사용자 의도 | 선택 패턴 |
|------------|----------|
| "구현해줘", "셋업해줘", "마이그레이션" | 📐 Planner-Executor |
| "비교해줘", "장단점", "뭐가 나을까" | ⚔️ Debate & Critic |
| "생성해줘", "리뷰해줘", "개선해줘" | ⚡ Generator-Evaluator |
| "설계하고 구현해줘", "코드 작성하고 리뷰해줘" | 🏗️ Code Generation |

> 📄 **더 알아보기**: 에이전트 파일 형식·작성법·호출 방법(3가지)·스킬 연동·4가지 패턴의 팀 구성과
> 협업 흐름·비교표는 [레퍼런스 C](#레퍼런스-c-멀티-에이전트-패턴-상세)를 참고하세요.

### 에이전트에게 줄 도구 — MCP 서버 연결

`.copilot/mcp-config.json`로 MCP 서버를 Copilot CLI에 붙입니다. 이 저장소에는 **Azure · GitHub ·
Microsoft Learn** 세 가지 서버가 설정되어 있습니다.

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": { "Authorization": "Bearer ${GITHUB_PERSONAL_ACCESS_TOKEN}" },
      "tools": ["*"]
    },
    "azure": {
      "type": "local",
      "command": "npx",
      "args": ["-y", "@azure/mcp@latest", "server", "start"],
      "tools": ["*"]
    },
    "microsoftLearn": {
      "type": "http",
      "url": "https://learn.microsoft.com/api/mcp",
      "tools": ["*"]
    }
  }
}
```

| 서버 | 유형 | 용도 | 인증 |
|------|------|------|------|
| **github** | 원격(http) | 이슈·PR·리포지토리 탐색/조작 | PAT — `GITHUB_PERSONAL_ACCESS_TOKEN` 환경변수 |
| **azure** | 로컬(npx) | 구독 내 Azure 리소스 조회·관리 (Foundry 포함) | `az login` 세션 |
| **microsoftLearn** | 원격(http) | Microsoft/Azure 공식 문서·코드 샘플 검색 | 불필요 |

설정 적용 및 확인:

```bash
# 1) (선택) github 서버를 쓸 때만 — GitHub PAT를 환경변수로 등록
export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxx

# 2) (선택) azure 서버를 쓸 때만 — Azure CLI 로그인 세션을 사용
#    여기서 az login은 'azure MCP 서버 인증' 용도일 뿐, CLI 학습 자체에는 필요 없습니다.
az login

# 3) Copilot CLI 세션에서 등록된 MCP 서버 확인
copilot
> /mcp           # 설정된 서버 목록·상태 확인
> /env           # 로드된 MCP 서버·인스트럭션·스킬 확인
```

`/mcp`를 실행하면 등록된 서버와 연결 상태가 표시됩니다(이름·유형은 환경에 따라 다를 수 있음).

```text
> /mcp
  github          http    ✓ connected
  azure           local   ✓ connected
  microsoftLearn  http    ✓ connected
```

> 💡 위 1)·2)는 **모두 선택**입니다. PAT·Azure 없이도 `microsoftLearn`(인증 불필요) 서버만으로
> 이 Part의 흐름을 따라갈 수 있습니다.
>
> 참고: Copilot CLI는 GitHub MCP 서버를 기본 내장하고 있어, 위 `github` 항목 없이도 기본 GitHub
> 기능은 사용할 수 있습니다. **단, `github` 블록을 유지하면 `GITHUB_PERSONAL_ACCESS_TOKEN`이 반드시
> 설정되어 있어야 인증 오류가 나지 않습니다.** PAT를 쓰지 않으려면 이 블록을 제거하고 기본 내장
> 서버를 사용하세요. `tools`는 `["*"]`로 모든 도구를 허용하므로, 읽기 전용만 노출하려면 서버 문서의
> 도구명으로 좁히세요. (유저/리포지토리 레벨 설정 파일 경로는 [레퍼런스 A — MCP·LSP 설정](#레퍼런스-a-copilot-cli-상세) 참고)

> ⚠️ **Azure MCP 권한 주의**: `azure` 서버는 `tools: ["*"]`이므로 구독 리소스를 **조회뿐 아니라
> 변경/삭제**할 수 있습니다. 실제 가능한 작업은 `az login` 계정의 **RBAC 권한** 범위로 제한되며,
> Copilot CLI는 실행 전 명령을 확인받습니다. 조회만 허용하려면 `tools`를 읽기 전용 도구명으로
> 좁히거나, 읽기 권한만 가진 계정으로 `az login` 하세요.

> **실습**: `copilot --agent orchestrator --yolo`를 띄우고
> *"Microsoft Learn에서 Agent Framework Concurrent 오케스트레이션 문서를 찾아 동시 워크플로우에
> 비용 리뷰 전문 에이전트를 추가하고 리뷰해줘"* 라고 요청해 보세요. (Learn 문서 검색 + 코드 생성 + 리뷰 연계)

> ✅ **확인**: `/mcp`에 서버가 `connected`로 보이고, 위 요청으로 오케스트레이터가 Learn 문서를
> 검색해 새 에이전트 코드를 제안하면 Part 3 완료입니다. (PAT·Azure 없이 `microsoftLearn`만으로도 가능)

---

## Part 4. 바이브 코딩 — 설정만으로 코드 생성하기

> 🎯 **목표**: 손으로 코드를 쓰지 않고 `.github/` 설정만으로 Copilot이 규칙에 맞는 코드를
> 생성·리뷰하게 만듭니다.
> ⏱️ 예상 소요: 약 15분 · Azure/Python 실행은 선택

**바이브 코딩**은 코드를 손으로 쓰는 대신, `.github/`의 instructions·prompts·skills로 의도를
정의하고 Copilot이 코드를 생성하게 하는 방식입니다. 핵심은 **"무엇을 만들 것인가"와 "어떤 패턴을
따를 것인가"를 정확히 문서화**하면, AI가 일관된 품질의 코드를 생성한다는 것입니다.

| 개발자가 준비 | Copilot이 수행 |
|---------------|----------------|
| `instructions/` — 기술 스택·코딩 규칙 | 규칙을 지킨 코드 생성 |
| `prompts/` — 반복 작업 템플릿 | 일관된 산출물 생성 |
| `skills/` — SDK 사용법·패턴 | 정확한 SDK 호출 |
| `agents/` — 리뷰/디버그 역할 | 자동 리뷰·디버깅 |

### 바이브 코딩 워크플로우 한눈에 보기

```text
┌─────────────────────────────────────────────────────────────────┐
│  1. 시작: copilot 실행 → .github/ 설정 자동 인식                  │
│     copilot-instructions.md, instructions/*.md → 자동 적용       │
├─────────────────────────────────────────────────────────────────┤
│  2. 계획: /plan 모드로 구현 계획 수립 (선택)                       │
│     "동시 워크플로우에 리뷰어를 추가하려면 어떤 순서로?"            │
├─────────────────────────────────────────────────────────────────┤
│  3. 생성: 자연어로 코드 생성/수정 요청                              │
│     "UX 리뷰 전문 에이전트를 04_concurrent_workflow.py에 추가해줘" │
├─────────────────────────────────────────────────────────────────┤
│  4. 검증: /diff로 변경사항 확인 → /review로 코드 리뷰              │
├─────────────────────────────────────────────────────────────────┤
│  5. 완료: 커밋 또는 /delegate로 PR 생성을 Copilot에 위임           │
└─────────────────────────────────────────────────────────────────┘
```

### 실습 흐름 (Azure 없이 진행)

```text
1. (CLI) "UX 리뷰 전문 에이전트를 동시 워크플로우에 추가해줘"라고 자연어로 요청
   (VS Code Copilot Chat이라면 /add-agent 프롬프트로 호출)
2. Copilot이 agent-framework-codegen 스킬 규칙(import·async·instructions)에 맞춰 코드 생성
3. /diff 로 변경사항 확인 → copilot --agent reviewer 로 리뷰 → 수정
4. (선택: Python이 설치된 경우) `python -m py_compile src/04_concurrent_workflow.py`로 문법 검증
   (Azure 불필요)
```

> 💡 **선택 사항**: `py_compile` 단계는 Python이 설치된 환경에서만 실행 가능합니다. 이 문서의 핵심은
> Copilot CLI 학습이므로, Python이 없다면 `/diff`와 `reviewer` 에이전트 검토만으로도 실습을 계속
> 진행할 수 있습니다.
>
> `py_compile`은 문법 오류가 없으면 **아무것도 출력하지 않고 종료**합니다(종료 코드 0). 그게 정상입니다.

> 위 흐름은 **CLI 학습이 목적**이므로 Azure 리소스나 실제 실행이 필요 없습니다. `/diff`·`reviewer`
> 에이전트·`py_compile`만으로 "Copilot이 규칙에 맞는 코드를 생성했는가"를 확인합니다.

이후에도 자연어로 요청하면 `.github/copilot-instructions.md`와 `instructions/*.md`의 규칙을 반영한
코드를 생성합니다.

```text
> RAG 에이전트에 top_k를 5로 변경해줘
> 새 MCP 도구를 추가해줘
> 동시 워크플로우에 새로운 검토 관점을 추가해줘
```

### 재사용 팁

- **공용 인스트럭션 복사**: `instructions/python.instructions.md`·`azure.instructions.md`·
  `korean.instructions.md`·`git-commit.instructions.md`를 새 프로젝트의 `.github/instructions/`에
  복사하면 동일한 컨벤션·보안·작성 규칙이 즉시 적용됩니다.
- **새 예제 추가 규칙**: 예제는 `src/`에 `NN_<name>.py` 규칙으로 추가하고, 진입점은
  `if __name__ == "__main__": asyncio.run(main())` 패턴을 따릅니다.
- **설정 구조 참고**: `.github/` 설정의 동작 방식과 구성요소별 역할은
  [레퍼런스 B](#레퍼런스-b-github-설정-상세)를 참고하세요.

### (선택) 생성한 코드를 실제로 실행해 보기

생성된 에이전트를 **런타임에서 동작**시켜 보고 싶다면, [Microsoft Agent Framework 핸즈온 랩](https://github.com/junwoojeong100/agent-framework-labs)의
사전 준비(Azure/Microsoft Foundry 리소스 + Python + `.env`)를 끝낸 뒤 다음을 실행합니다.

```bash
python src/04_concurrent_workflow.py   # 실제 실행 (Azure·Python 필요, 선택)
```

> ✅ **최종 체크포인트**: 직접 만든 `.github/` 설정만으로 Copilot이 규칙에 맞는 새 에이전트/기능을
> 생성·리뷰하게 만들 수 있으면, **이 CLI 랩의 목표를 달성**한 것입니다. (실제 실행 여부는 선택)

---

## Part 5. 가드레일 (AGENTS.md)

루트의 [`AGENTS.md`](AGENTS.md)는 **모든** Copilot 에이전트가 git/외부 명령 실행 전에 따르는
안전 규칙입니다.

| 규칙 | 내용 |
|------|------|
| **Rule 1** | 기능 브랜치 push 허용, **보호 브랜치(main) 직접 push·force push·`--all`/`--mirror` 금지** |
| **Rule 2** | 커밋/PR 메시지는 **영문 + Conventional Commits**(`feat:`, `fix:` …) |
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

> ✅ **확인**: 기능 브랜치(`feat/*`)에서 영문 Conventional Commits로 커밋하고 `--draft --base main`
> 으로 PR을 만들 수 있으면 Part 5 완료입니다. Copilot에게 커밋/PR을 맡겨도 이 가드레일을 따릅니다.

---

# 레퍼런스 (심화)

> 핸즈온 랩에서 다룬 내용을 더 깊이 파고들 때 참고하는 레퍼런스입니다. 필요한 항목만 찾아보세요.

## 레퍼런스 A. Copilot CLI 상세

### 설치 방법

**지원 플랫폼**: macOS · Linux · Windows(PowerShell v6 이상)
**사전 요구**: [GitHub Copilot](https://github.com/features/copilot) 구독 활성화

> ⚠️ 조직(Organization)/엔터프라이즈로 Copilot을 사용하는 경우 관리자가 Copilot CLI를 비활성화했을 수
> 있습니다. [조직의 Copilot 정책](https://docs.github.com/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-github-copilot-features-in-your-organization/managing-policies-for-copilot-in-your-organization)을 확인하세요.

```bash
# 설치 스크립트 (macOS / Linux) — curl 또는 wget
curl -fsSL https://gh.io/copilot-install | bash
wget -qO- https://gh.io/copilot-install | bash
# 루트 설치는 | sudo bash, 특정 버전·경로는 VERSION/PREFIX 환경변수 지정
curl -fsSL https://gh.io/copilot-install | VERSION="<desired-version>" PREFIX="$HOME/custom" bash

# Homebrew (macOS / Linux)
brew install copilot-cli
brew install copilot-cli@prerelease        # 프리릴리즈

# WinGet (Windows)
winget install GitHub.Copilot
winget install GitHub.Copilot.Prerelease   # 프리릴리즈

# npm (전 플랫폼)
npm install -g @github/copilot
npm install -g @github/copilot@prerelease  # 프리릴리즈
```

### 인증 (GitHub 계정 · PAT)

첫 실행 후 세션에서 `/login`을 입력하고 브라우저 안내에 따라 인증합니다. **PAT**로도 인증할 수
있습니다.

1. <https://github.com/settings/personal-access-tokens/new> 에서 토큰 생성
2. "Permissions" → "Copilot Requests" 권한 추가
3. 환경변수로 설정 (우선순위 `COPILOT_GITHUB_TOKEN > GH_TOKEN > GITHUB_TOKEN`)

```bash
export COPILOT_GITHUB_TOKEN="your-token-here"
# 또는 export GH_TOKEN=...  /  export GITHUB_TOKEN=...
```

### 기본 사용법

| 기능 | 방법 |
|------|------|
| **모델 선택** | `/model` — Claude Sonnet/Opus, GPT-5 등에서 선택(기본값은 버전·환경에 따라 다름) |
| **모드 전환** | `Shift+Tab` — Interactive(단계별 승인) ↔ Plan(실행 전 계획 수립) |
| **파일 멘션** | 프롬프트에서 `@경로` 입력 → 해당 파일을 컨텍스트에 포함 (예: `@src/01_single_agent.py 설명해줘`) |
| **로컬 셸 실행** | `!`로 시작하면 Copilot을 거치지 않고 로컬 셸에서 직접 실행 (예: `!ls -la`, `!npm run test`) |

**키보드 단축키 — 일반**

| 단축키 | 기능 |
|--------|------|
| `Ctrl+S` | 현재 프롬프트 임시 저장/복원 (stash/pop) |
| `Ctrl+T` | 모델 추론 과정 표시 토글 |
| `Ctrl+O` / `Ctrl+E` | 최근 / 전체 타임라인 확장 (입력 없을 때) |
| `↑` `↓` | 명령 히스토리 탐색 |
| `Ctrl+C` | 취소 / 입력 지우기 / 선택 복사 (`×2` 종료) |
| `Esc` | 현재 작업 취소 |
| `Ctrl+D` | 종료 |
| `Ctrl+L` | 화면 지우기 |
| `Ctrl+X → O` | 최근 타임라인의 링크 열기 |

**키보드 단축키 — 편집**

| 단축키 | 기능 |
|--------|------|
| `Ctrl+A` / `Ctrl+E` | 줄 맨 앞 / 맨 끝으로 이동 (입력 중) |
| `Ctrl+H` / `Ctrl+W` | 이전 글자 / 이전 단어 삭제 |
| `Ctrl+U` / `Ctrl+K` | 커서부터 줄 시작 / 줄 끝까지 삭제 |
| `Meta+← →` | 단어 단위로 커서 이동 |
| `Ctrl+G` | 외부 에디터에서 프롬프트 편집 |

### CLI 실행 옵션

| 플래그 | 설명 |
|--------|------|
| `--agent <name>` | 특정 에이전트를 지정하여 실행 |
| `--autopilot` | Autopilot 모드로 바로 실행 |
| `--yolo` | 자동 승인 모드 — 도구 실행을 매번 확인하지 않음 |
| `--experimental` | 실험적 기능 활성화 |
| `--banner` | 시작 시 애니메이션 배너 다시 표시 |

```bash
copilot --agent orchestrator --yolo --autopilot   # 에이전트 + 자동 승인 + Autopilot
```

> ⚠️ `--yolo`는 모든 파일 변경·명령어가 자동 실행됩니다. **신뢰할 수 있는 환경에서만** 사용하세요.

### 슬래시 커맨드 레퍼런스

> 💡 자주 쓰는 주요 커맨드입니다. 전체 목록은 CLI 내에서 `/help`로 확인하세요.

**에이전트 환경**

| 커맨드 | 설명 |
|--------|------|
| `/init` | 리포지토리용 Copilot 인스트럭션 초기화 |
| `/agent` | 사용 가능한 에이전트 탐색·선택 |
| `/skills` | 스킬 관리 (Azure 등 확장 기능) |
| `/mcp` | MCP 서버 설정 관리 |
| `/env` | 현재 세션에 로드된 전체 환경 확인 (인스트럭션·MCP·스킬·에이전트) |
| `/plugin` | 플러그인 및 마켓플레이스 관리 |

**모델 및 서브에이전트**

| 커맨드 | 설명 |
|--------|------|
| `/model` | AI 모델 선택 |
| `/delegate` | 세션을 GitHub에 보내 Copilot이 PR 생성 |
| `/fleet` | 병렬 서브에이전트 실행 모드 활성화 |
| `/tasks` | 백그라운드 태스크(서브에이전트, 셸 세션) 관리 |

**코드 작업**

| 커맨드 | 설명 |
|--------|------|
| `/ide` | IDE 워크스페이스 연결 |
| `/diff` | 현재 디렉토리의 변경 사항 리뷰 |
| `/pr` | 현재 브랜치의 PR 관련 작업 |
| `/review` | 코드 리뷰 에이전트 실행 |
| `/plan` | 코딩 전에 구현 계획 작성 |
| `/research` | GitHub 검색과 웹 소스를 활용한 심층 리서치 |
| `/lsp` | 언어 서버 설정 관리 |
| `/terminal-setup` | 멀티라인 입력 지원 설정 (Shift+Enter) |

**권한**

| 커맨드 | 설명 |
|--------|------|
| `/allow-all` | 모든 권한 활성화 (도구, 경로, URL) |
| `/add-dir` · `/list-dirs` | 파일 접근 허용 디렉토리 추가 · 목록 표시 |
| `/cwd` | 작업 디렉토리 변경 또는 표시 |
| `/reset-allowed-tools` | 허용된 도구 목록 초기화 |

**세션**

| 커맨드 | 설명 |
|--------|------|
| `/resume` · `/rename` · `/new` | 세션 전환 · 이름 변경 · 새 대화 시작 |
| `/context` · `/usage` | 컨텍스트 토큰 사용량 · 세션 사용량 통계 |
| `/session` | 세션 조회·관리 |
| `/compact` | 대화 히스토리 요약으로 컨텍스트 절약 |
| `/share` · `/copy` | 세션 공유(마크다운·HTML·Gist) · 마지막 응답 복사 |
| `/rewind` | 마지막 턴 되돌리기 및 파일 변경 복원 |

**도움말 및 피드백**

| 커맨드 | 설명 |
|--------|------|
| `/help` | 도움말 표시 |
| `/autopilot` | Autopilot 모드 전환 |
| `/changelog` | CLI 버전별 변경 로그 (`summarize` 추가 시 AI 요약) |
| `/feedback` · `/theme` | 피드백 제출 · 색상 모드 설정 |
| `/update` · `/version` | 최신 버전 업데이트 · 버전 정보 |
| `/experimental` | 실험적 기능 관리 |
| `/clear` | 현재 세션 폐기 후 새로 시작 |
| `/instructions` | 커스텀 인스트럭션 파일 확인·토글 |
| `/streamer-mode` | 스트리머 모드 토글 (모델명·할당량 숨김) |

### 커스텀 인스트럭션 파일 인식·우선순위

Copilot CLI는 다양한 위치의 인스트럭션 파일을 자동으로 인식합니다.

| 파일 | 설명 |
|------|------|
| `AGENTS.md` | 에이전트 공통 가드레일 (Git 루트 & cwd) |
| `CLAUDE.md` / `GEMINI.md` | Claude / Gemini 모델 전용 인스트럭션 |
| `.github/instructions/**/*.instructions.md` | 디렉토리별 세부 인스트럭션 |
| `.github/copilot-instructions.md` | 리포지토리 전체 인스트럭션 |
| `~/.copilot/copilot-instructions.md` | 유저 전체 글로벌 인스트럭션 |

환경변수 `COPILOT_CUSTOM_INSTRUCTIONS_DIRS`로 추가 디렉토리를 지정할 수 있습니다. 여러 파일이
존재하면 **모두 동시에 적용**되며, 충돌 시 단일 우선순위로 단정할 수 없습니다(조합에 따라 동작이
비결정적일 수 있음). 현재 적용 파일은 `/instructions`로 확인합니다. 이 프로젝트는 [`AGENTS.md`](AGENTS.md)로
모든 에이전트에 공통 가드레일을 적용합니다.

### MCP·LSP 설정 파일 위치

**MCP 서버** 설정(예시는 [Part 3](#part-3-멀티-에이전트-패턴으로-개발하기)):

| 레벨 | 경로 | 적용 범위 |
|------|------|----------|
| 유저 | `~/.copilot/mcp-config.json` | 모든 프로젝트 |
| 리포지토리 | `.copilot/mcp-config.json` | 해당 프로젝트만 |

**LSP 서버** 를 연결하면 go-to-definition, hover, 진단 등 코드 인텔리전스를 사용할 수 있습니다.

| 레벨 | 경로 | 적용 범위 |
|------|------|----------|
| 유저 | `~/.copilot/lsp-config.json` | 모든 프로젝트 |
| 리포지토리 | `.github/lsp.json` | 해당 프로젝트만 |

```json
{
  "lspServers": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "fileExtensions": { ".ts": "typescript", ".tsx": "typescript" }
    },
    "python": {
      "command": "pylsp",
      "args": [],
      "fileExtensions": { ".py": "python" }
    }
  }
}
```

서버 바이너리 설치: `npm install -g typescript-language-server`, `pip install python-lsp-server`.
상태 확인은 `/lsp`.

### 유용한 팁

- **업데이트**: `/update` 또는 패키지 매니저(`brew upgrade copilot-cli` / `npm update -g @github/copilot` /
  `winget upgrade GitHub.Copilot`). Copilot CLI는 빠르게 개발 중이므로 최신 버전 유지를 권장합니다.
- **Autopilot 모드**: `--autopilot` 또는 세션 내 `/autopilot`. 작업이 끝날 때까지 자동으로 진행합니다.
- **컨텍스트 관리**: 대화가 길어지면 `/compact`로 히스토리를 요약하고, `/context`로 토큰 사용량을
  확인합니다. 새로 시작하려면 `/clear`.
- **세션 공유**: `/share`로 마크다운·HTML·GitHub Gist로 내보냅니다.
- **Premium Request**: 프롬프트를 보낼 때마다 월간 프리미엄 요청 할당량이 1개 차감됩니다
  ([자세히](https://docs.github.com/copilot/managing-copilot/monitoring-usage-and-entitlements/about-premium-requests)).

---

## 레퍼런스 B. `.github/` 설정 상세

> Part 2의 `.github/` 트리·구성요소 역할표를 전제로, 각 파일 형식과 동작·재사용 범위를 깊이 다룹니다.

### `SKILL.md` 구조 (Skill)

| 필드 | 설명 |
|------|------|
| `name` | 스킬 식별자 |
| `description` | **언제 쓰는지**(USE FOR / DO NOT USE FOR). Copilot이 이 설명을 보고 로드 여부 결정 |
| 본문 | SDK 패턴·예제. 관련 작업일 때만 로드(**점진적 공개** → 토큰 절약) |

### `*.agent.md` frontmatter (Custom Agent)

| 필드 | 설명 |
|------|------|
| `name` | 선택 (생략 시 파일명 사용) |
| `description` | **필수** — 에이전트의 역할 |
| `tools` | 허용 도구 별칭: `read`·`search`·`edit`·`execute`·`agent`·`web` (생략=전체 허용, `[]`=없음) |
| `model` / `target` | 선택 — 사용할 모델, 실행 대상(`vscode`/`github-copilot`) |

### 파일 유형별 동작 방식

| 파일 유형 | 위치 | 호출 방법 | 적용 방식 |
|----------|------|----------|----------|
| **글로벌 인스트럭션** | `.github/copilot-instructions.md` | 없음 | ✅ 자동 — 모든 Copilot 요청에 항상 포함 |
| **파일 패턴 인스트럭션** | `.github/instructions/*.instructions.md` | 없음 | ✅ 자동 — `applyTo` 패턴 매칭 파일 작업 시 포함 |
| **스킬** | `.github/skills/*/SKILL.md` | `/스킬명` | ✅ 자동 + 🔘 수동 — 관련 주제 감지 시 자동 로드, 명시적 호출도 가능 |
| **재사용 프롬프트** | `.github/prompts/*.prompt.md` | `/프롬프트명` | 🔘 수동 |
| **커스텀 에이전트** | `.github/agents/*.agent.md` | `/agent` 선택 또는 `--agent <name>` | 🔘 수동 |

### 언제, 어떤 파일 유형을 사용해야 하나요?

| 파일 유형 | 주요 용도 | 사용 시점 예시 |
|----------|----------|--------------|
| **인스트럭션** | **"항상 이 규칙을 따라라"** — 코드 컨벤션·보안 정책·언어 규칙 등 상시 자동 적용 | Python마다 Google docstring·타입 힌트, Azure 코드에서 항상 `AzureCliCredential` |
| **스킬** | **"이 분야의 전문 지식을 참고해라"** — 특정 SDK·프레임워크 API·패턴·트러블슈팅 | Agent Framework SDK로 에이전트를 만들 때 올바른 import·API 사용법 |
| **재사용 프롬프트** | **"지금 이 작업을 이 방식으로 해라"** — 반복 작업 템플릿 | 새 예제를 `NN_<name>.py` 규칙으로 추가, 매번 같은 체크리스트로 리뷰 |
| **커스텀 에이전트** | **"이 역할을 맡아서 수행해라"** — 역할별 페르소나·도구 권한 | 요청에 맞는 협업 패턴 자동 선택, 에러를 환경→인증→런타임 순으로 진단 |

> **요약**: 인스트럭션은 "상시 규칙", 스킬은 "참고 자료", 프롬프트는 "작업 템플릿", 에이전트는 "전문가 역할".

### CLI에서 `.github/` 설정 동작 방식

| `.github/` 파일 | CLI 인식 | CLI에서의 활용 |
|------|:---:|------|
| `copilot-instructions.md` | ✅ 자동 | 프로젝트 기술 스택·코드 패턴·컨벤션이 모든 요청에 반영 |
| `instructions/*.instructions.md` | ✅ 자동 | Python·Azure·한국어·Git 커밋 규칙이 자동 적용 |
| `skills/*/SKILL.md` | ✅ `/skills`, `/스킬명` | `/skills`로 목록·관리, `/스킬명` 또는 프롬프트 내 명시로 호출 |
| `prompts/*.prompt.md` | ⚠️ 직접 호출 불가 | 프롬프트 내용을 자연어로 풀어서 요청 |
| `agents/*.agent.md` | ✅ `/agent` | `/agent`로 에이전트 선택·전환 (orchestrator, reviewer, debugger 등) |

> **핵심**: `copilot-instructions.md`와 `instructions/` 파일들은 **IDE와 CLI 모두에서 자동 적용**됩니다.

### 재사용 범위: 공용 🌐 vs 프로젝트 전용 🔒

**🌐 공용 — 다른 프로젝트에 복사하여 즉시 재사용 가능**

| 파일 | `applyTo` | 역할 |
|------|-----------|------|
| `instructions/python.instructions.md` | `**/*.py` | Python 가상환경·의존성·컨벤션(Google docstring·타입 힌트·import 순서) |
| `instructions/azure.instructions.md` | `**/*.py` | 키리스 인증(`AzureCliCredential`), `.env` 관리, 민감정보 보안 |
| `instructions/korean.instructions.md` | `**` | 한국어 응답·주석·docstring, 변수명은 영어, 기술 용어 병기 |
| `instructions/git-commit.instructions.md` | `**` | 영문 Conventional Commits, 트레일러 규칙 |

**🔒 프로젝트 전용 — 이 프로젝트의 기술 스택(Agent Framework·Foundry·MCP)에 특화**

| 파일 | 역할 |
|------|------|
| `copilot-instructions.md` | 프로젝트 전체 규칙 — 주제·스크립트 정의, SDK import 경로, `FoundryChatClient`/`SequentialBuilder` API 패턴 |
| `prompts/add-agent.prompt.md` | 새 에이전트·예제 추가 템플릿 (`NN_<name>.py` 규칙) |
| `prompts/review-code.prompt.md` | 코드 리뷰 — 패턴 준수·보안·비동기 정합성 체크리스트 |
| `skills/agent-framework-codegen/SKILL.md` | Agent Framework 코드 생성 스킬 — import 경로, 에이전트·워크플로우·MCP·RAG 패턴 |
| `agents/*.agent.md` | orchestrator + 4가지 협업 패턴 + reviewer(읽기 전용)·debugger(진단) |

### Copilot 커스텀 파일 전체 관계도

```text
.github/
├── copilot-instructions.md          ← 🔒 항상 자동 적용 (프로젝트 전체 규칙)
│
├── instructions/                    ← 🌐 파일 패턴별 자동 적용 (재사용 가능)
│   ├── python.instructions.md       ← *.py 편집 시 자동
│   ├── azure.instructions.md        ← *.py 편집 시 자동
│   ├── korean.instructions.md       ← 모든 파일 편집 시 자동
│   └── git-commit.instructions.md   ← 모든 파일 편집 시 자동
│
├── skills/                          ← 🔒 자동 로드 + /스킬명으로 수동 호출
│   └── agent-framework-codegen/     ← MAF SDK 코드 생성 시
│       └── SKILL.md
│
├── prompts/                         ← 🔒 /이름으로 수동 호출 (CLI는 자연어)
│   ├── add-agent.prompt.md          ← /add-agent
│   └── review-code.prompt.md        ← /review-code
│
└── agents/                          ← 🔒 @이름 또는 --agent로 호출
    ├── orchestrator.agent.md        ← 패턴 자동 선택
    ├── planner_executor.agent.md    ← 계획-실행
    ├── debate_critic.agent.md       ← 토론-비평
    ├── generator_evaluator.agent.md ← 생성-평가
    ├── code_generation.agent.md     ← 코드 생성
    ├── reviewer.agent.md            ← @reviewer (읽기 전용)
    └── debugger.agent.md            ← @debugger (진단)
```

---

## 레퍼런스 C. 멀티 에이전트 패턴 상세

> Part 3의 에이전트 목록·오케스트레이터 의도→패턴 매핑을 전제로, 작성·호출·협업 패턴을 깊이 다룹니다.

### 커스텀 에이전트 파일 형식·만들기

커스텀 에이전트는 `.github/agents/<이름>.agent.md` 경로에 저장하며 **YAML frontmatter + 본문 지시문**
구조입니다. 파일명에서 `.agent.md`를 제외한 부분이 에이전트 이름이 됩니다.

```markdown
---
name: 에이전트 표시 이름
description: 에이전트 역할 요약 (Copilot이 자동 선택 시 참고)
---

# My Agent

> 에이전트의 역할과 목적을 설명하는 한 줄 요약

## 역할 (Role)
이 에이전트는 [구체적인 역할]을 수행합니다.

## 규칙 (Rules)
1. 항상 [특정 규칙]을 따릅니다
2. [제약 조건]을 준수합니다

## 워크플로우 (Workflow)
1. 사용자 요청을 분석합니다
2. [단계별 수행 절차]
3. 결과를 문서화합니다
```

frontmatter 아래 본문에는 역할·행동 규칙·출력 형식·위임 기준을 마크다운으로 작성합니다. 실제 예시는
이 저장소의 [`.github/agents/`](.github/agents/) 디렉토리(예: `orchestrator.agent.md`,
`reviewer.agent.md`)를 참고하세요.

### 에이전트 호출 방법 (3가지)

1. **CLI 플래그** — 세션 시작 시 역할 고정: `copilot --agent <에이전트명>`
2. **슬래시 명령** — 대화 중 `/agent` 입력 후 선택
3. **자연어 지시** — 프롬프트에서 이름을 직접 언급 (예: `reviewer 에이전트로 검토해줘`)

CLI 플래그는 처음부터 역할을 고정할 때, `/agent`·자연어 지시는 대화 도중 전환할 때 편리합니다.

### 스킬(Skill) 연동

커스텀 스킬은 `.github/skills/<스킬명>/SKILL.md` 경로의 **전문 지식 / 절차 문서**입니다.

- **자동 로드**: 관련 작업이 감지되면 Copilot이 적절한 스킬을 자동 참조
- **수동 호출**: `/skills`로 목록 확인, `/스킬명`으로 직접 호출

이 저장소의 실제 예시는 `.github/skills/agent-framework-codegen/SKILL.md`로, Microsoft Agent Framework
코드 생성·워크플로우 구성·Foundry 연동 지침이 정리되어 있습니다.

### 협업 패턴 4가지 상세

각 패턴은 **여러 전문 에이전트가 역할을 분담**하여 협업하며, 모든 팀에는 과정·결과를 문서화하는
**Scribe**가 포함됩니다.

#### 📐 Planner-Executor — 계획 수립과 실행을 분리하여 체계적으로 완수

| 에이전트 | 역할 |
|---------|------|
| **Planner** | 요구사항 분석 → 태스크 목록·의존성·완료 기준 정의 |
| **Executor** | 계획에 따라 태스크를 순서대로 구현 |
| **Validator** | 각 태스크 검증 — Pass/Revise 판정 |
| **Scribe** | 계획·실행·검증 과정 문서화 |

```text
요구사항 → Planner → Executor → Validator →(Revise)→ Planner
                                          →(Pass)→ 다음 태스크 → … → Scribe
```

적합한 작업: 구현, 마이그레이션, 리팩토링, 단계별 셋업

#### ⚔️ Debate & Critic — 대립적 논증과 비평으로 최선의 결론 도출

| 에이전트 | 역할 |
|---------|------|
| **Proposer** | 찬성/제안 측 입장과 근거 제시 |
| **Opponent** | 반대 논증 및 대안 제시 |
| **Critic** | 양측 논증의 강점·약점 객관적 분석 |
| **Synthesizer** | 논의 종합 후 수렴 판단 — 수렴 시 권고안 도출 |
| **Scribe** | 논의 과정·최종 결론 문서화 |

```text
주제 → Proposer → Opponent → Critic → Synthesizer →(수렴)→ Scribe
                                                  →(미수렴)→ Round 2 (최대 3 Rounds)
```

적합한 작업: 기술 선택, 아키텍처 비교, 장단점 분석

#### ⚡ Generator-Evaluator — 생성과 평가를 반복하여 품질 향상

| 에이전트 | 역할 |
|---------|------|
| **Generator** | 요구사항을 충족하는 초안 생성 |
| **Evaluator** | 기준표 기반 품질 평가 — Pass/Fail 판정 |
| **Refiner** | Evaluator 피드백 반영하여 산출물 개선 |
| **Scribe** | Cycle별 개선 이력·최종 결과 문서화 |

```text
요구사항 → Generator → Evaluator →(Pass)→ Scribe
                               →(Fail)→ Refiner → Evaluator (최대 3 Cycles)
```

적합한 작업: 코드·문서 생성, 리뷰 기반 반복 개선

#### 🏗️ Code Generation — 설계 → 구현 → 리뷰를 체계적으로 연결

| 에이전트 | 역할 |
|---------|------|
| **Architect** | 코드 구조·인터페이스·의존성·패턴 설계 |
| **Developer** | Architect 설계에 따라 구현, Reviewer 피드백 반영 수정 |
| **Reviewer** | 보안·코드 품질·설계 준수 검증 — Pass/Revise 판정 |
| **Scribe** | 설계·구현·리뷰 과정·최종 명세 문서화 |

```text
요구사항 → Architect → Developer → Reviewer →(Pass)→ Scribe
                                            →(Revise)→ Developer (최대 3 Cycles)
```

적합한 작업: 신규 기능 설계·구현·리뷰 통합

### 패턴별 비교

| 패턴 | 목적 | 핵심 루프 | 최대 반복 | 팀 구성 |
|---|---|---|---|---|
| 📐 **Planner-Executor** | 체계적 실행 | 계획→실행→검증 | 3회 Revise | Planner·Executor·Validator·Scribe |
| ⚔️ **Debate & Critic** | 최선의 결론 도출 | 제안→반론→평가 | 3 Rounds | Proposer·Opponent·Critic·Synthesizer·Scribe |
| ⚡ **Generator-Evaluator** | 반복 개선으로 품질 향상 | 생성→평가→개선 | 3 Cycles | Generator·Evaluator·Refiner·Scribe |
| 🏗️ **Code Generation** | 설계 기반 코드 생성 | 설계→구현→리뷰 | 3 Cycles | Architect·Developer·Reviewer·Scribe |

---

# 부록

## 부록. 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| `copilot: command not found` | 설치 경로가 PATH에 없음 | `npm install -g @github/copilot` 재실행 후 새 터미널. 경로 확인 `npm prefix -g` / `ls ~/.local/bin/copilot`, 필요 시 `export PATH="$HOME/.local/bin:$PATH"` |
| 실행 시 로그인 안내만 반복 | GitHub 인증 미완료 | 세션에서 `/login` 후 브라우저 인증(자동으로 안 열리면 표시된 URL 수동 열기), 또는 `export GH_TOKEN=...`. 조직 정책으로 비활성화됐을 수 있으니 관리자 확인 |
| `/mcp`에 서버가 안 보임 | `.copilot/mcp-config.json` 미인식 또는 JSON 오류 | 저장소 루트에서 `copilot` 실행, JSON 문법 확인 후 세션 재시작 |
| github 서버 인증 오류 | `GITHUB_PERSONAL_ACCESS_TOKEN` 미설정 | PAT를 `export` 하거나, `github` 블록을 제거하고 기본 내장 GitHub 서버 사용 |
| azure 서버 연결 실패 | `az login` 세션 없음/만료 | `az login` 재실행, `az account show`로 로그인 상태 확인 |
| MCP 서버 연결 실패(일반) | 서버 바이너리 미설치/PATH 누락 | `/mcp`로 상태 확인, 서버 바이너리 설치 및 PATH 포함 여부 확인 |
| `--agent <name>` 실행 안 됨 | 에이전트 파일명/위치 불일치 | `ls .github/agents/*.agent.md` 확인, 세션에서 `/agent`로 목록 확인 |
| 응답 품질 저하 / 컨텍스트 초과 | 대화가 길어져 컨텍스트 윈도우 초과 | `/compact`(히스토리 요약) · `/context`(사용량 확인) · `/clear`(세션 초기화) |
| Node.js 버전 오류 | Node 22 미만 | `node -v` 확인 후 22+ 설치 (<https://nodejs.org>) |

## 참고: VS Code vs CLI

GitHub Copilot은 **VS Code(IDE)**와 **Copilot CLI(터미널)**에서 일부 `.github/` 커스터마이징을
공유하지만, 지원 범위와 적용 방식은 서로 다릅니다. 개발 환경에 따라 두 가지 방식을 선택하거나
병행할 수 있습니다.

| 항목 | 🖥️ VS Code (IDE) | 💻 Copilot CLI (터미널) |
|------|:---:|:---:|
| **설정 파일 인식** | `copilot-instructions.md`, `instructions/`, `skills/`, `prompts/`, `agents/` | `copilot-instructions.md`, `instructions/`, `skills/`, `agents/` + `AGENTS.md` 등 |
| **코드 생성** | 에디터 내 인라인 + 채팅 패널 | 터미널에서 직접 파일 생성/수정 |
| **프롬프트 호출** | `/프롬프트명` (채팅) | 프롬프트 파일 직접 호출 없음 (일반 자연어 입력 사용) |
| **에이전트 호출** | 에이전트 피커에서 선택 | `/agent` 선택 또는 `--agent` 플래그 |
| **스킬 관리** | 자동 로드 + `/스킬명` | `/skills`로 관리 |
| **MCP 서버** | 설정 기반 자동 연결 | `/mcp`로 관리 |
| **코드 리뷰** | 리뷰용 커스텀 에이전트 또는 적절한 에이전트 선택 | `/review` 명령어 |
| **변경사항 확인** | Git 패널 | `/diff` 명령어 |
| **플랜 모드** | Plan 에이전트(코딩 에이전트) 선택 | `/plan` 명령어 |
| **PR 생성 위임** | Copilot Coding Agent에 이슈 할당 | `/delegate`로 Copilot에 위임 |

## 더 알아보기 (선택, 심화)

- [GitHub 멀티 계정 설정 가이드](docs/github-multi-account-setup.md) — 한 머신에서 Git 작업용 계정과
  Copilot 구독 계정을 분리해 사용하는 방법
- [GitHub Copilot CLI 공식 문서](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) ·
  [사용 가이드](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli) ·
  [Copilot 플랜 및 가격](https://github.com/features/copilot/plans)

## 예제 코드를 실제로 실행하고 싶다면

`src/`의 Microsoft Agent Framework 예제는 이 랩에서 **바이브 코딩의 대상(예시 도메인)** 으로
포함되어 있습니다. 생성·수정한 코드를 **실제로 실행**(= Azure·Python 필요)해 보고 싶다면, 단계별
실행 가이드가 있는 **[Microsoft Agent Framework 핸즈온 랩](https://github.com/junwoojeong100/agent-framework-labs)**
저장소의 사전 준비를 따르세요.

## 프로젝트 구조

```
.
├── README.md                       # 이 핸즈온 랩 (메인 문서: 랩 + 레퍼런스)
├── AGENTS.md                       # 에이전트 공통 가드레일 (push 금지·영문 커밋·PR 규칙)
├── .copilot/
│   └── mcp-config.json             # MCP 서버 설정 (azure · github · microsoftLearn)
├── .github/
│   ├── copilot-instructions.md     # 프로젝트 전역 인스트럭션
│   ├── instructions/               # python · azure · korean · git-commit 규칙
│   ├── prompts/                    # add-agent · review-code (재사용 프롬프트)
│   ├── agents/                     # orchestrator · planner_executor · debate_critic · generator_evaluator · code_generation · reviewer · debugger
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

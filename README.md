# GitHub Copilot CLI 핸즈온 랩 — CLI로 멀티 에이전트 개발 가속하기

> **GitHub Copilot CLI 자체를 개발 도구로 활용**하는 법(설치 · `.github/` 설정 · 멀티 에이전트
> 패턴 · 바이브 코딩 · 가드레일)을 단계별로 익히는 **자체 완결형 핸즈온 랩**입니다.

이 저장소는 같은 저장소의 Microsoft Agent Framework 코드(`src/`)를 **예시 도메인**으로 삼아
"Copilot에게 에이전트 코드를 생성·리뷰시키는" 흐름을 보여줍니다. **Azure 리소스나 Python 코드 실행
없이도** 이 랩을 완주할 수 있습니다.

> 📄 **메인 실습 문서 → [docs/copilot-cli-lab.md](docs/copilot-cli-lab.md)** 부터 시작하세요.

---

## 🎯 이 랩을 마치면

1. Copilot CLI를 설치·인증해 터미널에서 대화하고,
2. `.github/` 설정으로 Copilot의 동작을 "조종"하며,
3. 커스텀 에이전트 + MCP 도구로 멀티 에이전트 개발을 수행하고,
4. 바이브 코딩으로 규칙에 맞는 코드를 생성·리뷰하고,
5. 가드레일(`AGENTS.md`)로 안전하게 커밋/PR 할 수 있습니다.

⏱️ 전체 예상 소요: 약 45–60분 (Azure/Python으로 실제 실행하는 부분은 선택)

---

## 사전 준비

| 도구 | 필수/선택 | 용도 | 설치 |
|------|-----------|------|------|
| **GitHub Copilot 구독** | 필수 | Copilot CLI 사용 권한 | <https://github.com/features/copilot> |
| **GitHub Copilot CLI** | 필수 | 터미널 AI 에이전트 | `npm install -g @github/copilot` |
| **Node.js 22+** | 필수 | CLI 런타임 (+ `npx`로 실행되는 MCP 서버) | <https://nodejs.org> |
| **이 저장소 클론** | 필수 | `.github/`·`.copilot/` 설정을 실습 대상으로 사용 | `git clone <repo>` |
| **GitHub PAT** | 선택 | `github` MCP 블록 사용 시에만 | <https://github.com/settings/tokens> |
| **Azure CLI + `az login`** | 선택 | `azure` MCP 서버 인증 시에만 | `az upgrade --yes` |

---

## 학습 경로

메인 실습 문서 [docs/copilot-cli-lab.md](docs/copilot-cli-lab.md)를 **위에서 아래로** 따라가세요.

| Part | 내용 |
|------|------|
| **1** | Copilot CLI 시작하기 (설치 · 인증 · 첫 대화) |
| **2** | Copilot을 "조종"하는 `.github/` 설정 |
| **3** | 멀티 에이전트 패턴으로 개발하기 |
| **4** | 바이브 코딩 — 설정만으로 코드 생성하기 |
| **5** | 가드레일 (`AGENTS.md`) |

### 더 알아보기 (선택, 심화)

- [Copilot CLI 가이드](docs/copilot-cli-guide.md) — 설치·인증·슬래시 커맨드 전체 레퍼런스
- [Copilot을 "조종"하는 `.github/` 설정](docs/github-config-guide.md)
- [멀티 에이전트(Custom Agent) 패턴](docs/custom-agents-guide.md)
- [VS Code(IDE) vs Copilot CLI(터미널) 비교](docs/vscode-vs-copilot-cli.md)
- [GitHub 멀티 계정 설정 가이드](docs/github-multi-account-setup.md)

---

## 프로젝트 구조

```
.
├── README.md                       # 이 가이드
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
├── docs/                           # CLI 실습 문서 (메인: copilot-cli-lab.md)
└── src/                            # 바이브 코딩 예시 도메인 (Microsoft Agent Framework 예제)
```

---

## 예제 코드를 실제로 실행하고 싶다면

`src/`의 Microsoft Agent Framework 예제는 이 랩에서 **바이브 코딩의 대상(예시 도메인)** 으로
포함되어 있습니다. 생성·수정한 코드를 **실제로 실행**(= Azure·Python 필요)해 보고 싶다면, 단계별
실행 가이드가 있는 **[Microsoft Agent Framework 핸즈온 랩](https://github.com/junwoojeong100/agent-framework-labs)**
저장소의 사전 준비를 따르세요.

---

## 라이선스

[MIT](LICENSE)

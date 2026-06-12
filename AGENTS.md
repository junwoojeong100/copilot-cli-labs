# AGENTS.md — Agent Harness

> 모든 에이전트가 반드시 준수해야 하는 제약 조건 및 가드레일.
> Copilot CLI/Chat의 모든 에이전트는 역할(Role)에 관계없이 이 문서의 규칙을 따라야 한다.

---

## 적용 범위

- **대상:** **모든** 에이전트 (Developer, Reviewer, Debugger, Orchestrator 등 역할 무관)
- **시점:** 에이전트가 Git 명령어 또는 외부 시스템 호출을 실행하기 **전에** 이 문서를 참조해야 한다
- **우선순위:** 개별 에이전트 정의와 이 Harness가 충돌할 경우, **Harness가 우선**한다

---

## Harness Rules

### Rule 1: 기능 브랜치 push 허용 / 보호 브랜치 push 절대 금지 — 🟡 조건부 허용

#### 허용되는 명령어

에이전트는 **기능 브랜치(feature branch)** 에 한해 다음 형태의 push를 실행할 수 있다:

```bash
git push -u origin <feature-branch>
git push origin <feature-branch>
git push --set-upstream origin <feature-branch>
```

기능 브랜치란 `main`, `master`, `develop`, `release/*`, `hotfix/*` 등 **보호 브랜치가 아닌**
모든 브랜치를 말한다 (예: `feat/*`, `fix/*`, `docs/*`, `chore/*`, `refactor/*`).

#### 금지 명령어

다음은 **절대 실행해서는 안 된다**:

```bash
# 보호 브랜치에 직접 push 금지
git push origin main
git push origin master
git push origin develop

# 히스토리 재작성 push 금지 (모든 브랜치 대상)
git push --force
git push --force-with-lease
git push -f

# 광역 push 금지
git push --all
git push --mirror
git push --tags          # 태그는 사용자가 직접 푼다
```

#### 이유

보호 브랜치는 PR + 리뷰로만 반영해 Human Review·히스토리 무결성·감사 추적을 보장하고,
에이전트가 실수로 보호 브랜치를 덮어쓰거나 force push로 공유 히스토리를 손상시키는 사고를 차단한다.

#### 표준 워크플로우

```bash
git checkout -b feat/my-change
git add . && git commit -m "feat: describe the change in English"
git push -u origin feat/my-change
gh pr create --draft --base main --title "PR title in English" --body "..."
```

---

### Rule 2: 커밋 메시지는 영문으로 작성 — 🟢 필수

- 모든 `git commit` 메시지(제목·본문)는 **영문**으로 작성한다.
- [Conventional Commits](https://www.conventionalcommits.org/) 스타일(`feat:`, `fix:`, `docs:`,
  `chore:`, `refactor:`, `test:` 등)을 권장한다.
- 명령형 현재 시제(imperative mood)로 작성한다. 예: `Add`, `Fix`, `Update`.
- PR 제목과 본문도 영문으로 작성한다.

```bash
# ✅ 올바른 예시
git commit -m "feat: add refund specialist agent to handoff workflow"
git commit -m "fix: handle missing PROJECT_ENDPOINT in single agent sample"

# ❌ 잘못된 예시
git commit -m "환불 에이전트 추가"
```

---

### Rule 3: PR은 항상 `main` 브랜치를 대상으로 생성 — 🟢 필수

- 모든 PR은 **`main` 브랜치를 base로 생성**한다 (`--base main`을 명시적으로 지정).
- PR은 기본적으로 **Draft(`--draft`)로 생성**하여 사람의 리뷰·승인을 전제로 한다.

```bash
# ✅ 올바른 예시
gh pr create --draft --base main \
  --title "feat: add new agent pattern" \
  --body "Summary of the change in English."
```

---

## 위반 시 처리

금지 명령어 실행이 감지되면 해당 에이전트의 작업을 **즉시 중단**하고 사용자에게 **보고**한 뒤,
필요 시 `git reset`·`git revert` 등으로 상태를 **복구**한다.

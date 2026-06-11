---
applyTo: "**/*.py"
---

# Python 프로젝트 공통 인스트럭션

## 가상환경 설정

- Python 3.14.5 기반 `.venv` 가상환경을 사용한다 (`.venv/`는 `.gitignore`로 버전 관리에서 제외).
- 생성·활성화 후 가상환경 안에서 패키지를 설치한다:
  ```bash
  python -m venv .venv
  source .venv/bin/activate   # macOS/Linux (Windows: .venv\Scripts\activate)
  pip install -r requirements.txt
  ```

## 의존성 관리

- 의존성은 `requirements.txt`에 명시하고, 패키지를 추가하면 갱신한다.
- 재현성을 위해 버전을 명시한다: SDK·핵심 패키지는 `==` 또는 `>=x,<y`, 프리릴리스는 `>=` 최소 버전 지정도 허용.

## 코드 컨벤션

- 한국어 주석과 docstring을 사용한다.
- 모듈 최상단에 모듈 설명 docstring을 작성한다.
- 함수에는 Args/Returns를 포함한 Google 스타일 docstring을 작성한다.
- 타입 힌트를 적극 활용한다.
- import 순서: 표준 라이브러리 → 서드파티 → 로컬 모듈 (각 그룹 사이에 빈 줄).
- 모든 에이전트/IO 호출은 `async/await`로 작성하고, 진입점은
  `if __name__ == "__main__": asyncio.run(main())` 형태를 따른다.
- 예외는 `try/except`로 감싸고 사용자 친화적인 한국어 메시지를 출력한다.

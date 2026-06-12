---
applyTo: "**/*.py"
---

# Azure 인증 / 시크릿 컨벤션

## 인증

- 로컬 개발은 `AzureCliCredential`(`azure-identity`)을 사용하고 `az login` 세션으로 인증한다(배포 환경은 `DefaultAzureCredential`/Managed Identity).
- 인증 객체와 `FoundryChatClient`는 재사용하고 요청마다 새로 만들지 않는다.
- API 키 기반 대신 passwordless 인증을 우선한다.

## 시크릿 / 보안

- 환경변수(`PROJECT_ENDPOINT`, `MODEL_DEPLOYMENT_NAME`)는 `.env`에서 `load_dotenv()`로 로드하고 코드에 하드코딩하지 않는다. `.env`는 커밋 금지(`.env.example`만 공유).
- 필수 환경변수는 누락 시 즉시 실패시키거나 명확한 한국어 안내 후 종료한다.
- 로그·출력에 토큰·키 등 민감정보를 남기지 않는다.

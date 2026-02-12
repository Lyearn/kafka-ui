# Repository Guidelines

## Project Structure & Module Organization
This repository is a multi-module project:
- `kafka-ui-api`: Spring Boot backend (`src/main/java`, `src/test/java`)
- `kafka-ui-contract`: shared API contract and generated clients
- `kafka-ui-serde-api`: serialization/deserialization API module
- `kafka-ui-react-app`: React + TypeScript frontend (`src/components`, `src/lib`, `src/generated-sources`)
- `kafka-ui-e2e-checks`: end-to-end UI checks (Maven/TestNG suites)
- `documentation/`, `etc/`, `.github/workflows/`: docs, style config, CI

## Build, Test, and Development Commands
- `./mvnw -B -V -ntp verify`: build and run backend/module checks.
- `./mvnw clean install`: build all modules.
- `./mvnw clean install -DskipTests`: build all modules without tests.
- `./mvnw -pl kafka-ui-api test`: run backend tests for API module only.
- `./mvnw spring-boot:run -pl kafka-ui-api`: run backend locally.
- `./mvnw spring-boot:run -pl kafka-ui-api -Dspring-boot.run.profiles=local`: run backend with local profile.
- `cd kafka-ui-react-app && pnpm install --frozen-lockfile`: install frontend deps.
- `cd kafka-ui-react-app && pnpm gen:sources`: regenerate OpenAPI TS clients.
- `cd kafka-ui-react-app && pnpm dev`: run frontend locally (Vite).
- `cd kafka-ui-react-app && pnpm build`: build frontend.
- `cd kafka-ui-react-app && pnpm lint && pnpm test`: run frontend lint/tests.
- `cd kafka-ui-react-app && pnpm lint:CI && pnpm test:CI`: CI-equivalent frontend lint/tests.
- `./mvnw -f kafka-ui-e2e-checks test -Pprod`: run e2e checks.
- `docker-compose -f ./documentation/compose/kafka-ui.yaml up`: run local Kafka + UI stack.

## Node.js Version Handling
- If a specific Node.js version is required, install it with `nvm install <version>`.
- Activate it with `nvm use <version>`.
- After UAT verification, notify the user.
- Do not uninstall Node.js versions automatically; the user will uninstall the version manually.

## Coding Style & Naming Conventions
- Follow `.editorconfig`: UTF-8, LF, trailing newline, default 4 spaces; Java uses 2-space indentation.
- Java style is enforced via `etc/checkstyle.xml`.
- Frontend uses ESLint + Prettier (`kafka-ui-react-app/.eslintrc.json`, `.prettierrc`).
- Prefer `camelCase` for variables/query params, `PascalCase` for React components, and lowercase plural REST paths (hyphen-separated segments).

## Testing Guidelines
- Backend: JUnit 5 + Testcontainers under `kafka-ui-api/src/test/java`.
- Frontend: Jest + Testing Library, tests named `*.spec.ts` / `*.spec.tsx` in `__test__` or `__tests__`.
- Run affected module tests locally before opening a PR; keep CI checks green.

## Commit & Pull Request Guidelines
- Match existing commit style: concise imperative subject, often scoped (`BE: ...`, `FE: ...`) and issue reference like `(#4427)`.
- Use GitHub closing keywords in commits/PR description when applicable.
- PRs must include: clear summary, testing notes, linked issues, matching labels, and docs/env var updates when behavior changes.
- Complete `.github/PULL_REQUEST_TEMPLATE.md` checklist to satisfy PR automation.

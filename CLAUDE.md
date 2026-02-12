# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UI for Apache Kafka is a free, open-source web UI to monitor and manage Apache Kafka clusters. Built with Spring Boot (reactive WebFlux) backend and React/TypeScript frontend.

**Key Technologies:**
- Backend: Java 17, Spring Boot 3.1.3, Spring WebFlux (reactive), Kafka clients 3.5.0
- Frontend: React 18, TypeScript, Vite, Redux Toolkit, TanStack React Query
- Build: Maven (backend), pnpm (frontend)
- Testing: JUnit 5, Testcontainers (backend), Jest (frontend), Selenide/TestNG (E2E)

## Build & Development Commands

### Backend (from project root)

```bash
# Build all modules
./mvnw clean install

# Build without tests
./mvnw clean install -DskipTests

# Run tests for specific module
./mvnw test -pl kafka-ui-api

# Run the application (requires Kafka cluster configured)
./mvnw spring-boot:run -pl kafka-ui-api

# Run with specific profile
./mvnw spring-boot:run -pl kafka-ui-api -Dspring-boot.run.profiles=local

# Check code style
./mvnw checkstyle:check
```

### Frontend (from kafka-ui-react-app/)

```bash
# Install dependencies
pnpm install

# Generate API clients from OpenAPI spec (required before first run)
pnpm gen:sources

# Start development server
pnpm dev

# Build for production
pnpm build

# Run tests
pnpm test

# Run tests with coverage
pnpm test:coverage

# Lint code
pnpm lint

# Fix linting issues
pnpm lint:fix

# Type check
pnpm tsc
```

### Docker Development

```bash
# Start local Kafka cluster with UI
docker-compose -f ./documentation/compose/kafka-ui.yaml up

# For frontend development with proxied backend
# Set VITE_DEV_PROXY in kafka-ui-react-app/.env.local
# then run: pnpm dev
```

### E2E Tests (from kafka-ui-e2e-checks/)

```bash
# Run E2E tests
./mvnw test
```

## Node.js Version Handling

- If a specific Node.js version is required, install it with `nvm install <version>`.
- Activate it with `nvm use <version>`.
- After UAT verification, notify the user.
- Do not uninstall Node.js versions automatically; the user will uninstall the version manually.

## Project Structure

### Maven Modules

- **kafka-ui-contract**: OpenAPI spec (`src/main/resources/swagger/kafka-ui-api.yaml`) and generated API interfaces/DTOs
  - Generates server-side API interfaces for backend
  - Generates TypeScript client for frontend (via pnpm gen:sources)

- **kafka-ui-api**: Main Spring Boot application
  - Entry point: `KafkaUiApplication.java`
  - Controllers: Implement generated API interfaces from kafka-ui-contract
  - Services: Business logic for Kafka operations
  - Emitters: Reactive message streaming components
  - Serdes: Built-in serialization/deserialization implementations

- **kafka-ui-serde-api**: Plugin API for custom serialization/deserialization
  - Interface for extending with custom serdes (Avro, Protobuf, etc.)

- **kafka-ui-e2e-checks**: End-to-end UI tests using Selenide and TestNG

- **kafka-ui-react-app**: React frontend application
  - Generated API clients in `src/generated-sources/` (from OpenAPI spec)
  - State management: Redux Toolkit + TanStack React Query
  - UI components in `src/components/`
  - Route configuration in `src/`

### Key Backend Packages

- `com.provectus.kafka.ui.controller`: REST API controllers (implement generated interfaces)
- `com.provectus.kafka.ui.service`: Business logic layer
- `com.provectus.kafka.ui.service.rbac`: Role-based access control
- `com.provectus.kafka.ui.service.metrics`: Kafka metrics collection
- `com.provectus.kafka.ui.emitter`: Reactive Kafka message emitters
- `com.provectus.kafka.ui.serdes`: Message serialization/deserialization
- `com.provectus.kafka.ui.config`: Spring configuration including auth, LDAP, OAuth
- `com.provectus.kafka.ui.model`: Domain models
- `com.provectus.kafka.ui.mapper`: MapStruct mappers for DTO conversions

## API Contract Workflow

The project uses code-first OpenAPI approach:

1. OpenAPI spec is manually maintained at `kafka-ui-contract/src/main/resources/swagger/kafka-ui-api.yaml`
2. Maven build generates:
   - Java interfaces and DTOs for backend (in kafka-ui-contract module)
   - TypeScript client for frontend (consumed by pnpm gen:sources)
3. Backend controllers implement the generated interfaces
4. Frontend uses generated TypeScript client for API calls

**When modifying APIs:**
1. Update the OpenAPI YAML spec in kafka-ui-contract
2. Run `./mvnw clean install` to regenerate Java code
3. Run `pnpm gen:sources` in kafka-ui-react-app to regenerate TypeScript client
4. Update controller implementations and frontend code

## Architecture Notes

### Backend

- **Reactive**: Uses Spring WebFlux with Project Reactor (Mono/Flux) for non-blocking operations
- **Multi-cluster**: Supports managing multiple Kafka clusters via configuration
- **Dynamic config**: Supports runtime configuration via `DYNAMIC_CONFIG_ENABLED` environment variable
- **Serdes**: Pluggable serialization system supporting Avro, Protobuf, JSON Schema, and custom implementations
- **RBAC**: Fine-grained role-based access control for resources
- **Auth**: OAuth2 (GitHub/GitLab/Google), LDAP support

### Frontend

- **React Query**: Used for server state management (API calls, caching)
- **Redux Toolkit**: Used for client state management
- **Code Generation**: TypeScript API client auto-generated from OpenAPI spec
- **Styling**: Styled-components and SASS
- **Forms**: React Hook Form with Yup validation
- **Routing**: React Router v6

## Testing

### Backend Tests

- Unit tests: Standard JUnit 5 tests in `src/test/java`
- Integration tests: Use Testcontainers to spin up Kafka clusters
- Base class: `AbstractIntegrationTest` provides common setup for integration tests
- Run specific test: `./mvnw test -Dtest=ClassName#methodName -pl kafka-ui-api`

### Frontend Tests

- Jest with React Testing Library
- Tests colocated with components
- Run in watch mode: `pnpm test`
- Coverage report: `pnpm test:coverage`

### E2E Tests

- Selenide for browser automation
- TestNG for test orchestration
- Uses docker-compose to spin up full environment
- Located in kafka-ui-e2e-checks module

## Code Style

### Backend

- Checkstyle configuration: `etc/checkstyle/checkstyle.xml`
- Can be imported into IntelliJ IDEA via Checkstyle plugin
- Enforced during build with `./mvnw checkstyle:check`

### Frontend

- ESLint with Airbnb config and TypeScript support
- Prettier for formatting
- Configuration in `.eslintrc.json` and `.prettierrc`

## Naming Conventions

- **REST paths**: lowercase, plural nouns, hyphens for multi-word segments (e.g., `/api/clusters/{clusterName}/consumer-groups`)
- **Query parameters**: camelCase
- **Model names**: camelCase, plural nouns
- **Java**: Standard Java conventions (PascalCase classes, camelCase methods/fields)
- **TypeScript**: PascalCase for components/types, camelCase for functions/variables

## Configuration

Application configuration via:
- `application.yaml` in kafka-ui-api/src/main/resources
- Environment variables (see docs.kafka-ui.provectus.io for full list)
- Dynamic config file when `DYNAMIC_CONFIG_ENABLED=true`

## Groovy Script Execution

Groovy script executions can be controlled via the environment variable:
- `filtering.groovy.enabled`: Enable/disable groovy script executions (security feature)

## Important Files

- `pom.xml`: Root Maven configuration with dependency versions
- `kafka-ui-contract/src/main/resources/swagger/kafka-ui-api.yaml`: OpenAPI specification
- `kafka-ui-api/src/main/java/com/provectus/kafka/ui/KafkaUiApplication.java`: Application entry point
- `kafka-ui-react-app/package.json`: Frontend dependencies and scripts
- `kafka-ui-react-app/vite.config.ts`: Vite build configuration
- `documentation/compose/`: Docker compose examples for various setups

## Common Development Tasks

**Adding a new API endpoint:**
1. Define endpoint in `kafka-ui-contract/src/main/resources/swagger/kafka-ui-api.yaml`
2. Regenerate code: `./mvnw clean install -pl kafka-ui-contract`
3. Implement in appropriate controller in kafka-ui-api
4. Add service layer logic
5. Regenerate frontend client: `cd kafka-ui-react-app && pnpm gen:sources`
6. Use generated client in frontend

**Adding a new custom serde:**
1. Implement `com.provectus.kafka.ui.serde.api.Serde` interface from kafka-ui-serde-api
2. Register in service loader or via configuration
3. Package as plugin JAR if external

**Running single backend test:**
```bash
./mvnw test -Dtest=TestClassName -pl kafka-ui-api
```

**Debugging frontend API calls:**
- API client is in `kafka-ui-react-app/src/generated-sources/`
- Check browser DevTools Network tab
- Backend API base path: `/api`

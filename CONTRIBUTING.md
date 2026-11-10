# Contributing & Architecture Guidelines - EventPulse

This document serves as the technical guide for the EventPulse project. It also acts as a context file for AI coding assistants (such as Claude, GitHub Copilot, or Cursor) to ensure that any generated code strictly follows the project's architectural decisions, preventing technical debt and inconsistencies.

## 1. Mandatory Tech Stack
*   **Backend:** Java 17 + Spring Boot 3.x (Spring Web, Data JPA, Security).
*   **Database:** PostgreSQL 16 managed via Flyway (Migrations).
*   **Frontend:** React (Vite) + TypeScript + Tailwind CSS.
*   **Infrastructure:** Docker and Docker Compose for local development.

## 2. Design Patterns & Business Rules
To ensure maintainability and clean code, the following rules are non-negotiable:
*   **Payments (Strategy Pattern):** All payment logic must be abstracted by the `PaymentStrategy` interface. Coupling the Stripe SDK directly inside Services is forbidden.
*   **State Machine (Template Method):** Event status transitions (`PublishTransition`, `CancelTransition`, `FinishTransition`) must extend the abstract class `EventStatusTransition`. Do not use `if/else` chains in Controllers for this.
*   **Financial Values:** Never use `double` or `float` for money. All monetary values must be handled as integers (`Long` or `BIGINT`) representing cents.
*   **Layers:** Controllers must not access Repositories directly. Business rules live exclusively in the `Service` layer.
*   **DTOs:** Use Java `record` for API request/response payloads. Never return JPA Entities (`@Entity`) directly in JSON responses.

## 3. Guidelines for AI Assistants
If you are an AI agent working in this repository, follow these directives strictly:
*   **No placeholders:** Never insert `// TODO` or `// implement later`. Provide complete, production-ready code.
*   **Tests:** Every new feature must come with tests. Use JUnit 5 + Mockito for the backend and Vitest for the frontend. Never disable tests (`@Disabled`).
*   **Logging:** `System.out.println()` is forbidden. Use the SLF4J logging API.
*   **Security:** No passwords, API keys, or secrets should be written in the code. Use environment variables defined in `.env.example`.
*   **Frontend UI:** Do not use hardcoded colors (e.g., `#E85A7A`) in React components. Use Tailwind CSS theme variables. Native browser `alert()`, `confirm()`, or `prompt()` are forbidden; use Toast/Modal components instead.

## 4. Validation Commands
Before committing any changes, ensure the project compiles and tests pass:
*   **Backend:** `cd backend && ./mvnw clean verify`
*   **Frontend:** `cd frontend && npm run test`
*   **Full environment:** `docker compose up --build`

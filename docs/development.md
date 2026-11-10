# Guia de Desenvolvimento - EventPulse

## 1. Configuração de Ambiente (.env)

Para rodar o projeto, você precisará configurar as variáveis de ambiente baseadas no arquivo `.env.example`:
- `DB_PASSWORD`: Senha do banco PostgreSQL.
- `JWT_SECRET`: Gerar com `openssl rand -base64 64`.
- `STRIPE_SECRET_KEY` e `STRIPE_PUBLISHABLE_KEY`: Chaves de teste do dashboard do Stripe.
- `STRIPE_WEBHOOK_SECRET`: Gerado pelo comando `stripe listen --forward-to localhost:8080/api/webhooks/stripe`.

## 2. Estrutura de Pastas

### Backend (`backend/src/main/java/br/ufpb/eventpulse/`)
- `config/`: SecurityConfig, StripeConfig, CorsConfig.
- `controller/`: Endpoints REST.
- `dto/`: Records de Request/Response.
- `domain/`: Entidades JPA e Enums.
- `repository/`: Interfaces Spring Data.
- `service/`: Regras de negócio.
- `payment/`: Strategy e implementação Stripe.
- `event/statemachine/`: Template Method e subclasses de transição.
- `integration/viacep/`: Client da API ViaCEP.
- `exception/`: GlobalExceptionHandler.

### Frontend (`frontend/src/`)
- `api/`: Clients Axios.
- `components/`: UI base e layout.
- `pages/`: Telas do sistema.
- `contexts/`: AuthContext, ThemeContext.

## 3. Ordem de Implementação (Roadmap de Fases)

| Fase | Entregável |
|------|-----------|
| **0** | Setup Spring Boot + Docker Compose (Postgres) + Healthcheck. |
| **1** | Entidades JPA + Migrations Flyway V1. |
| **2** | Auth (Register/Login/JWT filter). |
| **3** | CRUD de Organization/Event + Template Method de status. |
| **4** | Client ViaCEP + Endpoint. |
| **5** | Inscrição gratuita (com validação de vagas). |
| **6** | Stripe Checkout + Webhook (Strategy) + idempotência. |
| **7** | Frontend (Auth -> Listagem -> Checkout -> Dashboard). |
| **8** | CI (GitHub Actions) + Testes de Integração (Testcontainers). |
| **9** | (Opcional) Deploy DigitalOcean. |

## 4. Especificação do Frontend (Cores e Dark Mode)

**Contrato para a IA de UI/UX:**
1. Todas as cores devem ser referenciadas via variáveis CSS (`--color-primary`, `--color-bg`, `--color-surface`, `--color-text`). Proibido hexadecimais hardcoded nos componentes.
2. A troca de tema claro/escuro deve ser controlada pela classe `dark` no elemento `<html>` (Tailwind CSS).
3. Cores Primárias (Rosa): `#E85A7A` (Light) / `#F2A3B3` (Dark - maior luminosidade).
4. Botões, cards e inputs precisam ter conjuntos de variáveis para ambos os modos.

## 5. Testes (Casos Críticos)

- **Unitários Backend:** Cobrir `EventStatusTransitionTest` (todas as transições da máquina de estados) e `RegistrationServiceTest` (evento lotado, duplicado, evento gratuito).
- **Integração Backend:** Auth (Register -> Login -> Rota protegida) e Webhook (payload do Stripe validando a inscrição).
- **Frontend:** Vitest + Testing Library para renderização de lista de eventos e validação do botão de checkout.

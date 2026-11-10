# Arquitetura e Especificações Técnicas - EventPulse

## 1. Arquitetura do Sistema

O EventPulse adota uma **Arquitetura em Camadas (Layered Architecture)** combinada com o padrão **Strategy**, sem a introdução de microsserviços ou filas de mensageria. O objetivo é manter a simplicidade adequada para um projeto individual, sem sacrificar a robustez e a separação de responsabilidades.

### 1.1 Fluxo de Requisição
Frontend (React SPA) -> REST API (JSON) -> Spring Boot (Controller -> Service -> Repository) -> PostgreSQL.

## 2. Modelo de Dados (Entidades Principais)

*Decisões fixas:* UUID como chave primária (evita enumeração); Preços em centavos (BIGINT); `ddl-auto: validate` com Flyway gerenciando o schema.

- **User**: `id`, `name`, `email`, `password_hash`, `role` (ORGANIZER / ATTENDEE).
- **Organization**: `id`, `owner_id` (FK User), `name`, `slug`.
- **Event**: `id`, `organization_id` (FK), `title`, `description`, `starts_at`, `ends_at`, endereço (via CEP), `price_cents`, `capacity`, `status` (DRAFT, PUBLISHED, CANCELLED, FINISHED).
- **Registration**: `id`, `event_id`, `attendee_id`, `status` (PENDING_PAYMENT, CONFIRMED, CANCELLED), `stripe_session_id`, `amount_cents`, `confirmed_at`.

**Regras de Negócio:**
- Inscrição gratuita (`price_cents = 0`): status vira `CONFIRMED` imediatamente.
- Inscrição paga: fica `PENDING_PAYMENT` até o webhook do Stripe confirmar.
- Capacidade: bloqueia inscrição quando atingir o limite (usando lock otimista).

## 3. Padrões de Projeto (Design Patterns)

### 3.1 Strategy (Pagamento)
**Cenário:** Isolar a integração com o Stripe.
**Implementação:** `PaymentStrategy` (interface) e `StripePaymentStrategy` (implementação). O `RegistrationService` depende da interface, não do SDK do Stripe diretamente. Se amanhã precisarmos de outro gateway, basta criar outra classe que implemente a interface.

### 3.2 Template Method (Transição de Status do Evento)
**Cenário:** Mudanças de status (publicar, cancelar, finalizar) compartilham validações comuns, mas têm regras específicas.
**Implementação:** Classe abstrata `EventStatusTransition` define o fluxo `execute()` (validar permissão, validar status atual, aplicar regra específica). As subclasses (`PublishTransition`, `CancelTransition`, `FinishTransition`) implementam as regras específicas de cada transição.

## 4. Contrato de API e Autenticação

### 4.1 Autenticação
- **JWT** assinado com HS256, expiração de 24h. Sem refresh token (simplicidade).
- Armazenado no `localStorage` do frontend (simples; ciente de que `httpOnly cookie` é mais seguro e anotado como melhoria futura).
- O Webhook do Stripe é público, mas valida a assinatura `Stripe-Signature`.

### 4.2 Formato de Erro Padronizado
Todos os erros da API retornam o seguinte JSON:

> { "timestamp": "2026-11-10T14:30:00Z", "status": 409, "error": "INVALID_STATUS_TRANSITION", "message": "Event cannot transition from DRAFT to FINISHED", "path": "/api/events/550e8400.../status" }

## 5. Respostas para Perguntas de Borda (Entrevista)

- **Q: E se o attendee fechar a aba no Stripe?**
  *R:* A Registration fica `PENDING_PAYMENT`. Documentado como limitação; solução futura seria um job de expiração ou endpoint para o attendee cancelar.
- **Q: E se o webhook chegar duplicado?**
  *R:* Idempotência por `stripe_session_id`. Se o status já for `CONFIRMED`, retorna 200 sem reprocessar.
- **Q: Evento cancelado com inscrições pagas?**
  *R:* Não reembolsa automaticamente. O Organizer decide; a Stripe Refund API fica mapeada como trabalho futuro.

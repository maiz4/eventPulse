# EventPulse

**EventPulse** é uma plataforma SaaS de gestão e inscrição em eventos. O sistema permite que organizadores publiquem eventos (gratuitos ou pagos) e que participantes realizem suas inscrições de forma integrada com pagamentos online.

Este projeto foi desenvolvido como parte do meu portfólio pessoal, com o objetivo de demonstrar domínio em engenharia de software full-stack, arquitetura de sistemas, integração com APIs e boas práticas de CI/CD.

## Stack Tecnológica

**Backend:**
* Java 17 + Spring Boot 3
* Spring Security (JWT) + Spring Data JPA
* PostgreSQL + Flyway (Migrations)
* Stripe API (Pagamentos)
* Docker & Docker Compose

**Frontend:**
* React (Vite) + TypeScript
* Tailwind CSS (com suporte a Light/Dark mode)

**CI/CD & Infra:**
* GitHub Actions (Testes e Build)
* ViaCEP (Integração para autocompletar endereços)

## Decisões Arquiteturais

Meu obejtivo com esse projeto foi desenvolver algo simples, utilizando conceitos vistos na disciplina de APS e engenharia de software e focado na clareza do código:

* **Backend:** Foi utilizado apenas Spring Boot.
* **Padrões de Projeto:**
  * **Strategy:** Utilizado para isolar a lógica de pagamentos (`PaymentStrategy`). Isso permite que o sistema suporte outros gateways no futuro sem alterar o serviço de inscrições.
  * **Template Method:** Utilizado para a máquina de estados do Evento (Rascunho -> Publicado -> Finalizado/Cancelado), centralizando as validações comuns e permitindo regras específicas por estado.
* **SaaS Multi-tenant Simplificado:** O sistema organiza os eventos por `Organization`, permitindo que um organizador gerencie seus próprios eventos de forma isolada.

## Como rodar o projeto localmente

### Pré-requisitos
* Docker e Docker Compose instalados
* Uma conta no Stripe (para pegar as chaves de API de teste)
* Stripe CLI (para testar webhooks localmente)

### Passo a passo

1. Clone o repositório:
   `git clone https://github.com/maiz4/eventpulse.git`
   `cd eventpulse`

2. Configure as variáveis de ambiente:
   `cp .env.example .env`
   *Edite o arquivo .env e insira suas chaves do Stripe (STRIPE_SECRET_KEY, STRIPE_PUBLISHABLE_KEY)*

3. Suba os containers (Banco de Dados, Backend e Frontend):
   `docker compose up --build`

4. Teste os Webhooks do Stripe (em um terminal separado):
   `stripe listen --forward-to localhost:8080/api/webhooks/stripe`
   *Copie o whsec_... gerado e coloque no seu .env na variável STRIPE_WEBHOOK_SECRET*
   
5. Acesse a aplicação:
   * Frontend: `http://localhost:3000`
   * API Backend: `http://localhost:8080/api`
   * Healthcheck: `http://localhost:8080/actuator/health`
     
## Documentação Adicional

Para manter o código limpo, as especificações técnicas detalhadas estão na pasta `/docs`:

* [Arquitetura e Modelo de Dados](docs/architecture.md)
* [Guia de Desenvolvimento e Estrutura](docs/development.md)
* [Especificação da API (OpenAPI)](openapi.yaml) *(em construção)*

## Testes

O projeto conta com testes automatizados para garantir a integridade dos fluxos críticos:
* **Backend:** Testes unitários (JUnit + Mockito) para as regras de transição de status e cálculo de vagas. Testes de integração com Testcontainers para os endpoints.
* **Frontend:** Testes de componentes e validação de formulários com Vitest.

Para rodar os testes do backend:
`cd backend && ./mvnw clean test`

---
*Desenvolvido por Maiza Gomes*

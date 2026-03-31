# 🛡️ Projeto Argus Gateway
## Enterprise Multi-tenant Cloud Infrastructure

![Logo](argus_gateway_logo_1774929525375.png)

*The unblinking sentinel of your microservices ecosystem.*

[![Java 21](https://img.shields.io/badge/Java-21-orange?style=flat-square&logo=openjdk)](https://openjdk.org/)
[![Spring Boot 3.2](https://img.shields.io/badge/Spring_Boot-3.2-green?style=flat-square&logo=springboot)](https://spring.io/projects/spring-boot)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue?style=flat-square&logo=docker)](https://www.docker.com/)

---

## 🔍 Visão Geral

O **ARGUS Gateway** é uma infraestrutura de segurança distribuída projetada para aplicações SaaS modernas que exigem isolamento crítico de dados. Ele não apenas controla quem entra, mas garante que **nenhum dado vaze entre clientes (Tenants)** nas camadas mais profundas do sistema.

### 🏆 Diferenciais de Especialista

- **Zero-Trusted Data**: Isolamento nativo via **PostgreSQL Row-level Security (RLS)**.
- **Dynamic Governance**: Políticas de acesso via **Open Policy Agent (OPA)** (Rego).
- **Identity First**: Centralizado em **Keycloak** (OIDC/OAuth2).
- **High Performance**: Rate-limiting distribuído com **Redis**.

---

## 🏗️ Arquitetura do Sistema

```mermaid
graph TD
    classDef secure stroke:#00ffff,stroke-width:2px;
    classDef actor fill:#222,stroke:#fff,color:#fff;

    User((User)):::actor -->|JWT Request| Gateway[ARGUS Gateway]:::secure
    Gateway -->|1. Auth Check| Keycloak[Keycloak IAM]
    Gateway -->|2. Authorize?| OPA[Open Policy Agent]:::secure
    OPA -->|Allow/Deny| Gateway
    Gateway -->|3. Forward| ProductService[Product Service]
    ProductService -->|4. Set Session| Postgres[(PostgreSQL Vault)]:::secure
    Postgres -->|RLS Enforcement| RowLevel[Row Level Security]
```

---

## 🔐 Camadas de Blindagem (O "Cofre")

| Camada | Tecnologia | Função Principal |
| :--- | :--- | :--- |
| **Borda** | Spring Cloud Gateway | Filtro de entrada, auditoria e roteamento. |
| **Identidade** | Keycloak | Autenticação RSA256 e gestão de Claims de Tenant. |
| **Decisão** | OPA (Rego) | Polícia de acesso baseada em atributos (ABAC). |
| **Persistência** | Postgres RLS | Garantia matemática de que um Tenant nunca vê dados de outro. |

---

## 📄 Architectural Decision Records (ADR)

> [!IMPORTANT]
> ### Por que Row-Level Security (RLS)?
> A abordagem tradicional de isolamento de dados é frágil e propensa a erros humanos.

| Abordagem | Implementação | Risco |
| :--- | :--- | :--- |
| **Tradicional** | Filtros no código Java (`WHERE tenant_id = ?`) | **ALTO.** Um desenvolvedor pode esquecer o filtro em uma nova query. |
| **Expert (RLS)** | Políticas nativas no Banco de Dados | **ZERO.** O banco isola os dados mesmo se o código falhar. |

#### Como funciona no Postgres

```sql
-- Exemplo da nossa política de segurança
CREATE POLICY tenant_isolation_policy ON products
    USING (tenant_id = current_setting('app.current_tenant'));
```

---

## 🚀 Como Executar (Modo Showcase)

Basta um comando para subir todo o ecossistema pronto para ser testado:

```bash
# Clone e entre na pasta do projeto
# Suba todo o ecossistema (Gateway, Auth, DB, OPA)
docker compose up -d --build
```

---

## 🛠️ Stack Tecnológica

| Camada | Tecnologia | Ícone |
| :--- | :--- | :--- |
| **Backend** | Java 21, Spring Boot 3.2 | ![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white) |
| **Gateway** | Spring Cloud Gateway | ![Spring](https://img.shields.io/badge/Spring-6DB33F?style=for-the-badge&logo=spring&logoColor=white) |
| **Segurança** | Keycloak 24 & OPA | ![Keycloak](https://img.shields.io/badge/Keycloak-A10000?style=for-the-badge&logo=keycloak&logoColor=white) |
| **Database** | PostgreSQL 16 (RLS) | ![Postgres](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white) |
| **Cache** | Redis 7 | ![Redis](https://img.shields.io/badge/redis-%23DD0031.svg?style=for-the-badge&logo=redis&logoColor=white) |

---

## 🔒 Security Showcase
O diferencial deste projeto é a **Defesa em Profundidade**. Você pode testar o isolamento tentando acessar dados de um Tenant A com um Token do Tenant B. O sistema bloqueará a requisição em **três níveis**:
1. **Gateway:** O filtro OPA valida o `tenant_id` no JWT.
2. **Service:** O `TenantFilter` isola o contexto da thread.
3. **Banco de Dados:** O **RLS** garante que a query só retorne o que pertence ao Tenant logado.

---

Mantido com 🛡️ pela equipe **ARGUS Sentinel**.

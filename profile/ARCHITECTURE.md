
# 🏗️ Diretrizes de Arquitetura — uliving

Este documento consolida práticas para manter uma base escalável e de fácil manutenção, inspirada em **Clean Architecture**, **DDD (quando aplicável)** e princípios SOLID.

---

## 🎯 Objetivos

- Isolar regras de negócio de detalhes de framework
- Diminuir acoplamento e aumentar testabilidade
- Facilitar evolução, refatoração e onboarding
- Suportar múltiplos canais (web, mobile, integrações)

---

## 🧱 Clean Architecture (visão geral)

Organize o código para que **dependências apontem para dentro**:

- **Domain** → regras de negócio puras (entidades, value objects, políticas)
- **Application** → casos de uso (orquestração do domínio)
- **Infrastructure** → banco, mensageria, SDKs, HTTP clients, ORM
- **Presentation** → controllers, resolvers, UI, adaptadores de entrada

> Regra: **Domain não depende de NestJS/Angular/Next/Flutter/ORM.**

---

## 🔁 Fluxo de dependências

- `presentation → application → domain`
- `infrastructure → application/domain` (via interfaces/ports)

### Dependency Inversion (exemplo)

- Application define uma interface `UserRepository`
- Infrastructure implementa `PrismaUserRepository` (ou outro)
- Injeção de dependência liga tudo sem o domínio conhecer o detalhe

---

## 🧩 Ports & Adapters

Pense em integrações externas como “adapters”:

- **Port**: contrato (interface) definido internamente
- **Adapter**: implementação concreta (SDK, HTTP, DB, fila)

Benefícios:

- Troca de provedor com menor impacto
- Testes com fakes/mocks
- Melhor separação de responsabilidades

---

## 🧠 Domínio primeiro

- Regras de negócio devem estar no **domínio** ou em **casos de uso**
- Controller não deve conter regra de negócio (só adaptar input/output)
- ORM/DTO não devem “vazar” para o domínio

---

## 🧵 Eventos e mensageria (RabbitMQ)

Quando usar:

- Processos assíncronos
- Integrações entre serviços
- Redução de acoplamento entre fluxos

Boas práticas:

- Definir contratos claros (schema / versionamento)
- Implementar retries e DLQ quando necessário
- Idempotência para consumers

---

## 🗄️ Persistência (Cloud SQL / Mongo Atlas)

- Separar modelos de persistência (ORM schemas) das entidades de domínio
- Migrações versionadas e revisadas
- Índices e consultas críticos revisados (impacto em custo e performance)

---

## 🧪 Testabilidade por design

- Domínio e casos de uso devem ser testáveis sem infraestrutura real
- Infra pode ser testada com integração (localstack/containers quando fizer sentido)
- Use in-memory adapters para testes rápidos

---

## 📡 Observabilidade

- Logs estruturados e rastreáveis por request/trace
- Captura de erros e performance (Sentry)
- Métricas/alertas para fluxos críticos

---

## 📁 Sugestão de estrutura (NestJS)

Exemplo (adaptável por serviço):

- `src/domain/**`
- `src/application/**`
- `src/infrastructure/**`
- `src/presentation/**`

O NestJS normalmente vive em `presentation` e parte de `infrastructure` (DI, módulos, providers).

---

## 🧭 Padrões de decisão

Antes de escolher uma abordagem, responda:

1. Isso melhora a **experiência do estudante** ou a **eficiência operacional**?
2. Isso reduz risco de incidentes e facilita rollback?
3. Isso é observável, testável e sustentável?

Se a resposta for “não”, simplifique.

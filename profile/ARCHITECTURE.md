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

- Repositories de banco estendem o repository base do ORM e são injetados diretamente.
- Não crie interface/token por entidade apenas por cerimônia arquitetural.
- Use interface/port quando existir uma fronteira substituível real: múltiplas implementações, SDK,
  API externa, fila, storage, relógio ou gerador de identificador.
- Uma possibilidade futura não justifica abstração sem uso atual.

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

---

## Padrão vigente para back-ends NestJS

Estas regras complementam e, em caso de conflito, prevalecem sobre os exemplos genéricos anteriores:

- Crie um use case quando houver regra, múltiplas etapas, transação, efeito externo, evento ou
  reutilização independente do transporte. CRUD simples pode permanecer na fachada do módulo.
- Modules fazem somente composição de dependências.
- CRUD, paginação, ordenação e filtros genéricos permanecem nos repositories base compartilhados;
  queries específicas ficam no repository concreto e regra de negócio não pertence ao repository.
- Cada entidade/schema de persistência e cada repository concreto ficam em arquivos próprios. Em
  TypeORM, entidades de persistência terminam em `OrmEntity`.
- Barrels `index.ts` são proibidos para repositories. Importe cada repository diretamente pelo
  caminho do arquivo para manter explícita a origem da dependência.
- Decorators TypeORM ficam em linhas próprias, imediatamente acima do atributo, e propriedades ORM
  declaram explicitamente o modificador `public`.
- Listagens CRUD usam `findAllApi`; não recrie paginação, ordenação e filtros genéricos quando o
  repository base atender ao caso.
- Recursos CRUD reutilizam o controller base e seus endpoints de paginação e consulta por ID. Não o
  use quando endpoints herdados impedirem autorização, identidade ou visibilidade obrigatórias;
  justifique a exceção junto ao controller.
- Todo serviço expõe `GET /health` como rota pública, fora do prefixo global e sem depender de banco
  ou integrações externas.
- Eventos públicos incluem `eventId`, `occurredAt`, versão e payload tipado. Consumers são
  idempotentes, seguros para reentrega e preservam correlation ID quando disponível.
- Toda regra, transição, comportamento condicional e correção de bug exige teste. Testes unitários
  ficam em `src/modules/<módulo>/tests/` e espelham o caminho do arquivo testado.
- A estrutura em camadas é direção para módulos novos ou refatorações autorizadas. Não mova arquivos
  em massa junto com mudança funcional.
- Atualize `README.md`, `docs/**`, `.env.example` e o `AGENTS.md` quando mudar contrato, configuração,
  evento ou fluxo operacional.
- Todo módulo mantém `src/modules/<módulo>/README.md` com descrição objetiva de responsabilidade,
  entradas, saídas, regras, dependências, persistência, eventos e efeitos externos.
- Use Mermaid quando fluxos, sequências, estados, integrações ou dependências ficarem mais claros com
  diagrama; não use quando uma lista curta for suficiente e mantenha o diagrama sincronizado.
- Antes de entregar, execute `npm run check`, revise `git diff` e execute `git diff --check`.
- Não execute migrations, seeds, deploys, consumers ou scripts operacionais sem autorização explícita.

---

## Evolução de módulos e fronteiras de microserviços

### Estrutura evolutiva

- Todo projeto possui uma organização raiz explícita dentro de `src`: `src/core/` para componentes
  compartilhados/transversais e `src/modules/<módulo>/` para contextos de negócio.
- Essa regra também vale para o repositório chamado `core`: nele, o código compartilhado da aplicação
  fica sob `src/core/`; o nome do repositório não elimina essa pasta pai.
- `src/core/` pode conter `application`, `domain` e `infrastructure` compartilhados pelo serviço, mas
  não recebe regras pertencentes a um módulo específico.
- Não crie estruturas paralelas como `src/infra`, `src/common` ou `src/shared` para contornar essa
  organização. Código realmente transversal deve ser classificado dentro de `src/core/`.
- Código novo em módulos antigos segue a estrutura pragmática de Clean Architecture:
  `application`, `domain`, `infrastructure`, `presentation` e `tests`.
- Não perpetue uma organização antiga apenas porque o módulo ainda não foi migrado.
- Não mova arquivos antigos sem necessidade. Migre gradualmente quando forem alterados de forma
  relevante e mantenha a mudança revisável, sem misturar reorganização massiva com regra nova.
- Use cases ficam em `application`; regras e invariantes em `domain`; ORM, filas e providers em
  `infrastructure`; controllers, DTOs e subscribers de entrada em `presentation`.

### Comunicação entre módulos

- Um módulo nunca injeta nem chama o repository de outro módulo.
- Comunicação interna ocorre pelo service ou fachada pública exportada pelo módulo proprietário.
- Apenas o módulo proprietário conhece seus repositories e detalhes de persistência.
- Não use imports profundos para contornar a API pública do módulo.
- Se houver ciclo entre módulos, reveja responsabilidades; não use `forwardRef` como solução padrão.

### Desacoplamento entre microserviços

- Cada microserviço mantém suas regras, persistência e decisões dentro da própria fronteira.
- Não compartilhe banco, entidades ORM, repositories nem relações de banco entre serviços.
- Integrações usam contratos HTTP/eventos versionados, IDs externos e snapshots mínimos quando
  necessários. O serviço proprietário continua sendo a fonte da verdade.
- Prefira consistência eventual e idempotência a acoplamento transacional distribuído.
- Quanto mais uma regra puder ser resolvida dentro de um único serviço, menor deve ser o acoplamento
  necessário com os demais.

### Identificação por `code`

- Todo registro relevante de negócio compartilhado ou referenciado entre microserviços possui um
  `code` estável, único no contexto e independente do ID interno do banco.
- O mesmo objeto de negócio usa o mesmo `code` em todos os serviços que mantêm referência ou
  projeção dele. Por exemplo, um cliente com `code` `xxxxx` conserva esse valor no `core`, no
  `usupport` e nos demais consumidores.
- IDs internos (`id`, `_id`) não são contratos entre serviços. Use `code` em eventos, integrações,
  correlação, idempotência e reconciliação.
- O serviço proprietário gera o `code`; consumidores apenas o preservam. Não gere outro código para
  representar o mesmo registro e não altere um `code` já publicado sem estratégia de migração.
- Defina unicidade e índice para `code` na persistência e valide duplicidade nas fronteiras.

### API única e roteamento

- Embora implementado como microserviços, o back-end é apresentado ao cliente como uma única API.
- Em desenvolvimento local, o front-end acessa os serviços por proxy, preservando os mesmos paths
  públicos usados em produção.
- Em produção, Kubernetes e Ingress Controller roteiam cada requisição ao serviço responsável pelo
  path. O front-end não conhece endereço interno, porta, pod ou mecanismo de descoberta dos serviços.
- Paths públicos são contratos da plataforma. Mudança de prefixo ou responsabilidade exige análise
  de compatibilidade no proxy local, no Ingress e nos consumidores antes do cutover.

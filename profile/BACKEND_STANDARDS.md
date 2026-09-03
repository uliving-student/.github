# Padrão global de back-ends — uliving

Este documento é a fonte canônica para desenvolvimento e manutenção dos back-ends da uliving. Deve
ser lido antes do `AGENTS.md` local de cada repositório.

Link canônico:
<https://github.com/uliving-student/.github/blob/main/profile/BACKEND_STANDARDS.md>

Os `AGENTS.md` locais mantêm somente contexto, riscos e exceções específicas do serviço. Em caso de
conflito, este padrão prevalece. Exceção local exige justificativa, escopo, prazo ou critério de
remoção e, quando afetar outros serviços, `TODO(JIRA)` para rollout.

## Princípios

- Prefira a menor mudança que resolva completamente o problema.
- Clean Architecture é aplicada de forma pragmática; separação de responsabilidades é obrigatória,
  abstração sem benefício concreto não é.
- Regras ficam o mais desacopladas possível dentro do serviço proprietário.
- Preserve contratos públicos e planeje compatibilidade antes de mudanças incompatíveis.
- Não faça refatoração ampla fora do escopo da tarefa.

## Antes de alterar

1. Leia este documento, o `AGENTS.md` local, o README do módulo e o código afetado por completo.
2. Verifique `git status` e preserve mudanças fora da tarefa.
3. Procure um padrão semelhante no repositório.
4. Identifique contratos HTTP/RPC, eventos, migrations, persistência e efeitos operacionais.
5. Não adicione dependência sem justificar por que plataforma e dependências atuais não bastam.

## Organização do projeto

Todo projeto possui:

```text
src/
├── core/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
└── modules/
    └── <modulo>/
        ├── application/
        │   ├── services/
        │   └── use-cases/
        ├── domain/
        ├── infrastructure/
        ├── presentation/
        ├── tests/
        ├── <modulo>.service.ts
        └── <modulo>.module.ts
```

- `src/core/` contém componentes realmente compartilhados e transversais do serviço.
- A regra também vale para o repositório chamado `core`; o nome do projeto não elimina `src/core/`.
- Contextos e regras de negócio ficam em `src/modules/<módulo>/`.
- Não crie novas raízes paralelas como `src/infra`, `src/common` ou `src/shared`.
- A raiz do módulo é reservada ao module NestJS e à fachada pública quando ela existir.
- Modules fazem somente composição de dependências.

### Evolução de pastas antigas

- Código novo segue a estrutura acima mesmo quando o módulo ainda usa pastas antigas.
- Arquivos diretamente envolvidos e significativamente alterados podem ser migrados.
- Não mova ou renomeie pastas, arquivos e módulos fora do escopo atual.
- Migração ampla ocorre em tarefa e PR próprios.
- Não misture reorganização massiva com mudança funcional.

## Responsabilidades das camadas

- `domain`: entidades, estados, invariantes e regras puras, sem NestJS, ORM ou transporte.
- `application`: casos de uso, orquestração e portas canônicas.
- `infrastructure`: ORM, repositories, adapters, SDKs, filas, storage e clients externos.
- `presentation`: controllers, DTOs, subscribers e tradução entre transporte e aplicação.
- Fachada do módulo: contrato público interno e operações CRUD simples.

Não coloque regra de negócio em controller, DTO, migration, schema/entidade ORM, entity hook,
repository, module ou configuração.

Crie use case quando houver regra de negócio, múltiplas etapas, transação, efeito externo, evento ou
reutilização independente do transporte. CRUD simples pode permanecer na fachada.

## Direção das dependências

```text
domain <- application <- infrastructure/presentation
```

- `domain` não importa `application`, `infrastructure` ou `presentation`.
- `application` não importa `infrastructure` ou `presentation`.
- `infrastructure` implementa portas da aplicação quando houver fronteira substituível real.
- `presentation` converte HTTP, filas e RPC em inputs da aplicação e traduz resultados/erros.
- O composition root (`*.module.ts`) liga tokens e portas às implementações concretas.
- Casos de uso não selecionam adapters nem dependem de DTOs de transporte.
- Evite `forwardRef`; ciclos indicam responsabilidade ou fronteira mal definida.

## Comunicação entre módulos

- Nunca injete, importe ou chame repository pertencente a outro módulo.
- Módulos comunicam-se pelo service ou fachada pública exportada pelo módulo proprietário.
- Repository e persistência permanecem privados ao módulo proprietário.
- Não use imports profundos para contornar a API pública do módulo.

## Repositories e persistência

- Repositories TypeORM estendem `RepositoryMysqlBase<T>`; repositories MongoDB estendem
  `RepositoryMongoBase<T>`.
- Injete o repository concreto diretamente no service do próprio módulo.
- Não crie interface/token por entidade. Use porta para múltiplas implementações ou fronteiras reais:
  API/SDK externo, fila, storage, relógio, gerador de ID ou provider substituível.
- CRUD, paginação, ordenação e filtros genéricos permanecem no repository base.
- Listagens CRUD usam `findAllApi`; não recrie o contrato com QueryBuilder ou parsing manual.
- Queries, projeções e agregações específicas ficam no repository concreto.
- Regra de negócio não pertence ao repository.
- Cada repository fica em `<recurso>.repository.ts` próprio.
- Barrels `index.ts` são proibidos para repositories; use import direto pelo arquivo.
- Evite N+1, `select *` conceitual, populate/relações desnecessárias e consultas sem limite/índice.

### TypeORM

- Classes de persistência terminam em `OrmEntity` e ficam em `<nome>-orm.entity.ts` próprio.
- Entidade ORM não é entidade de domínio nem DTO HTTP.
- Decorators ficam em linhas próprias imediatamente acima do atributo.
- Propriedades ORM declaram explicitamente `public`.
- Busca obrigatória usa `findOneOrFailBy`; `findOneBy` é reservado para ausência válida.
- `EntityNotFoundFilter` global converte `EntityNotFoundError`; não repita conversão nos módulos.
- Escritas relacionadas usam uma transação e o mesmo `EntityManager`/`QueryRunner`.
- Nunca altere migration aplicada; crie uma nova migration reversível.

### MongoDB

- Injete `Model<T>` com `@InjectModel` no repository concreto.
- Use projection e `.lean()` quando documento Mongoose completo não for necessário.
- Valide IDs externos e prefira operadores atômicos.
- Escritas atômicas relacionadas compartilham uma única `ClientSession` quando houver suporte.
- Mudanças de schema preservam compatibilidade com documentos antigos e rollout gradual.

## Controllers, DTOs e paginação

- Recursos com listagem/consulta por ID reutilizam o controller base e seus `GET /` e `GET /:id`.
- O service expõe `findAll(filter): Promise<[T[], number]>` e `findOne(id)`.
- Adapte no service o retorno do repository base ao contrato esperado pelo controller.
- Não replique parsing, `skip`, `take`, ordenação ou metadados no controller concreto.
- Não use o controller base se endpoints herdados impedirem autorização, identidade ou visibilidade
  obrigatórias; justifique a exceção junto ao controller.
- DTOs de entrada usam `class-validator` e representam corretamente opcionalidade.
- Não exponha entidade ORM nem objeto de SDK externo como contrato HTTP.
- Use configuração centralizada (`ConfigService`, `AppConfig` ou equivalente), não `process.env`
  espalhado.

## Integrações externas e ports/adapters

Casos de uso não conhecem SDKs, clients HTTP, autenticação, URLs, configuração, payloads, status ou
nomes de campos específicos de fornecedores.

```text
Use case
  -> porta canônica da application
  -> adapter da infrastructure
  -> provider externo
```

- Portas usam linguagem canônica do sistema; formatos do fornecedor ficam no adapter.
- Cada provider traduz seu formato para o contrato canônico.
- Factory/registry no composition root seleciona adapters; não use condicionais de fornecedor no
  caso de uso.
- Provider sem implementação operacional falha explicitamente como não suportado.
- Defina timeout e falha explícita; não faça retry cego de efeito não idempotente.

### Efeitos externos não repetíveis

1. Registre localmente intenção ou estágio.
2. Execute o efeito externo uma única vez.
3. Persista imediatamente o identificador externo.
4. Execute efeitos locais secundários de forma idempotente.

- Falha depois de sucesso remoto não libera repetição cega.
- Falha ambígua permanece reconciliável e não volta automaticamente ao estado inicial.
- Retomada usa o identificador persistido e repete apenas etapas seguras.
- Locks/leases liberam somente estágios cuja repetição seja comprovadamente segura.

## Microserviços e identificação por `code`

- Serviços não compartilham banco, entidade ORM, repository ou relação de banco.
- O serviço proprietário é a fonte da verdade; consumidores guardam IDs externos e snapshots mínimos.
- Comunicação usa HTTP/RPC ou eventos versionados, idempotentes e compatíveis.
- Prefira consistência eventual a transação distribuída.
- Todo registro relevante referenciado entre serviços possui `code` estável, único no contexto e
  independente de `id`/`_id` interno.
- O mesmo objeto usa o mesmo `code` em todos os serviços.
- O proprietário gera o `code`; consumidores o preservam.
- Use `code` em eventos, integrações, idempotência, correlação e reconciliação.
- Defina índice/unicidade e não altere `code` publicado sem estratégia de migração.

## API única e roteamento

- O front-end enxerga os microserviços como uma única API.
- Localmente, o proxy do front-end roteia pelos mesmos paths públicos.
- Em produção, Kubernetes e Ingress Controller roteiam por path.
- O cliente não conhece endereço interno, porta, pod ou descoberta de serviço.
- Path público é contrato; mudança exige compatibilidade coordenada no serviço, proxy, Ingress e
  consumidores antes do cutover.

## Eventos e consumers

- Eventos públicos incluem versão, `eventId`, `occurredAt` e payload tipado.
- Consumers são idempotentes, toleram reentrega e não dependem de ordem global.
- Preserve correlation ID, origem e timestamps necessários à investigação.
- Use chave externa estável para deduplicação.
- Não descarte falha silenciosamente; registre contexto sanitizado e estratégia de reprocessamento.

## Bootstrap e health check

- Exponha `GET /health` público, fora do prefixo global e sem autenticação.
- O health check não depende de banco nem integração externa para responder ao probe básico.
- Exclua explicitamente `GET /health` do prefixo global quando necessário.

## TypeScript, estilo e IDE

- Use aspas simples, ponto e vírgula, tabs de largura 2, linha de até 100 caracteres e LF.
- Não use `any` em código novo; prefira tipo explícito ou `unknown` com narrowing.
- Não use assertion para silenciar o compilador; valide dados externos em runtime.
- Use `async` somente quando necessário e evite `return await` sem motivo.
- Prefira nomes do domínio a `data`, `item`, `manager` ou `helper` genéricos.
- Comentários explicam decisões e riscos; não repetem o código.
- Não desabilite TypeScript/ESLint globalmente. Exceção local exige justificativa.
- Arquivo novo fica sem warnings e trecho alterado não aumenta dívida existente.
- Projetos mantêm `.editorconfig`, Prettier, ESLint flat config e configurações/recomendações de IDE.

## Testes

- Toda regra, transição, condição e correção de bug exige teste; bug fix inclui regressão que falha
  sem a correção.
- Mudança apenas documental/configuração sem lógica pode dispensar teste, com justificativa no PR.
- Cubra comportamento observável, não detalhe privado.
- Mockeie somente fronteiras externas; nunca a unidade sob teste.
- Cubra sucesso, validação, autorização, duplicidade, idempotência e falha externa quando aplicáveis.
- Para efeitos remotos, cubra timeout, resposta perdida, retry, concorrência e falha após sucesso.
- Testes não dependem de rede, ordem, relógio/timezone real ou estado compartilhado.
- Testes unitários ficam em `src/modules/<módulo>/tests/`, espelhando o caminho do código e usando
  `.spec.ts`. E2E permanece em `test/`.
- Migre testes legados gradualmente quando forem significativamente alterados.

## Documentação

- Todo módulo mantém sua documentação em um `README.md` localizado diretamente na raiz do próprio
  módulo: `src/modules/<módulo>/README.md`.
- Não coloque essa documentação dentro de `application`, `domain`, `infrastructure`, `presentation`
  ou `tests`, nem crie uma pasta `docs` dentro do módulo apenas para esse conteúdo.
- O README do módulo deve ser objetivo e conter, conforme aplicável: responsabilidade e limites,
  entradas e saídas, casos de uso e regras principais, dependências com outros módulos, persistência,
  endpoints, eventos consumidos/publicados, integrações e efeitos externos.
- Atualize o README do módulo na mesma alteração sempre que seu comportamento, contrato, fluxo ou
  dependência mudar.
- Módulos novos já devem nascer com esse README. Em módulos antigos sem documentação, crie-o quando o
  módulo receber desenvolvimento novo ou uma alteração relevante; não é necessário documentar todos
  os módulos não relacionados à tarefa atual.
- Atualize `README.md`, `docs/**`, `.env.example` e documentação operacional quando mudar contrato,
  configuração, evento, ambiente ou fluxo.
- Use Mermaid quando fluxo, sequência, estado, integração ou dependência ficar mais claro com
  diagrama. Não use quando lista curta for suficiente e mantenha o desenho sincronizado.

### Evolução do padrão

- Regra global é alterada somente neste documento, nunca copiada para todos os `AGENTS.md`.
- Regra específica permanece no `AGENTS.md` do serviço.
- Mudança global sem rollout completo recebe `TODO(JIRA)` com motivo, repositórios afetados e critério
  de remoção. Nunca invente ID.
- Oriente o usuário a abrir o card; depois substitua por identificador real, como `TODO(ULT-123)`.

## Segurança e observabilidade

- Valide autenticação, autorização, audiência, unidade e transições no servidor.
- Nunca registre ou documente segredo, token, senha, documento, dado pessoal completo, credencial ou
  payload sensível.
- Não use dados reais em fixtures, testes, documentação ou mensagens de erro.
- Logs são estruturados, sanitizados e correlacionáveis.
- Preserve métricas e contexto necessários a diagnóstico sem expor dados sensíveis.
- Mantenha dependências atualizadas e trate vulnerabilidades conforme risco.

## Validação obrigatória

A validação é proporcional ao risco e ao alcance da alteração. Durante o desenvolvimento e antes de
entregar uma mudança localizada, execute no mínimo:

```bash
git diff --check
```

- revise o `git diff` completo da tarefa;
- execute os testes diretamente relacionados aos arquivos, casos de uso e contratos alterados;
- execute lint nos arquivos alterados, quando o projeto oferecer comando ou suporte para esse
  escopo;
- execute `typecheck` ou `build` quando a alteração afetar código compilado. Não é necessário rodar
  ambos localmente quando os dois validarem a mesma fronteira de compilação;
- informe comandos não executados, warnings e falhas preexistentes.

Execute lint, testes e build completos localmente quando a mudança afetar componentes transversais,
configuração de build/teste, dependências, contratos públicos, schema/migrations, autenticação,
autorização, eventos, concorrência ou vários módulos; execute também quando não houver CI confiável
cobrindo esses gates.

Correções e mudanças restritas a um módulo não exigem a suíte completa local se os testes focados, a
checagem de tipos/compilação aplicável e o CI cobrirem o restante. Não use `--runInBand` por padrão;
reserve execução serial para testes que comprovadamente dependam dela ou para diagnosticar
instabilidade.

Antes do merge, o CI deve validar no mínimo typecheck, lint, suíte completa de testes e build. Prefira
`npm run check` quando ele agrupar esses gates. Se o CI não executar algum deles, a lacuna deve ser
coberta localmente e registrada no PR.

Quando possível, automatize fronteiras arquiteturais no CI; documentação orienta, gate impede
integração da violação.

## Limites operacionais

- Não altere API pública, schema, migration, evento ou integração sem solicitação explícita.
- Não edite `.env`, chaves ou credenciais nem revele conteúdos.
- Não execute migration, seed, deploy, consumer real, importação ou sincronização automaticamente.
- Não faça limpeza/refatoração ampla, commit ou push sem autorização.
- Não toque pastas/módulos fora do escopo atual.

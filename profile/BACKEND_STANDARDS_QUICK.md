# Padrão de back-ends — resumo operacional

Use este resumo no trabalho diário. O
[padrão completo](https://github.com/uliving-student/.github/blob/main/profile/BACKEND_STANDARDS.md)
continua sendo a fonte canônica e prevalece em caso de dúvida ou conflito.

Leia o padrão completo ao iniciar trabalho no repositório e quando a alteração envolver arquitetura,
módulos, persistência, contrato público, evento, migration, segurança, rebase/merge ou baseline.
Na retomada da mesma tarefa, não releia documentos que não mudaram.

## Antes de alterar

1. Leia este resumo, o `AGENTS.md` local, o README e os arquivos afetados.
2. Verifique `git status` e preserve mudanças fora da tarefa.
3. No fluxo trunk-based, parta de `main`, mantenha a branch curta e abra um único PR para `main`.
4. Identifique contratos, persistência, eventos e efeitos externos envolvidos.
5. Após rebase ou merge, confira imports, modules, entidades ORM e baselines antes de continuar.

## Fronteiras obrigatórias

- `domain <- application <- infrastructure/presentation`.
- Domain não depende de NestJS, ORM, transporte ou camadas externas.
- Application não importa infrastructure nem presentation.
- Nunca importe, injete, registre ou chame repository pertencente a outro módulo.
- Não importe entidade ORM ou infraestrutura de outro módulo, nem apenas para tipagem.
- Módulos se comunicam pela fachada/service público do proprietário.
- `*.module.ts` compõe dependências; não contém regra e não registra entidade externa.
- Não aumente baseline para apenas fazer um teste passar.

## Persistência e TypeORM

- Entidade ORM fica em infrastructure e não é entidade de domínio nem DTO.
- Relação física legada externa usa o nome TypeORM registrado e um contrato estrutural local mínimo.
- A interface TypeScript não resolve a relação em runtime e não autoriza escrita no agregado externo.
- Se muitos consumidores precisarem da mesma projeção, amplie o contrato público do proprietário.
- Regra de negócio não fica em repository, entidade ORM, hook, DTO ou migration.
- Nunca altere migration aplicada nem execute migration ou seed automaticamente.

## Contratos, efeitos e segurança

- Preserve compatibilidade de HTTP/RPC, eventos e dados persistidos.
- Integrações externas ficam atrás de ports/adapters e usam contratos canônicos.
- Efeitos não repetíveis exigem idempotência, ID externo persistido e retomada segura.
- Não exponha nem registre segredo, token, documento ou dado pessoal completo.
- Não execute deploy, consumer, importação, sincronização ou efeito externo automaticamente.

## Validação

- Execute testes focados e lint nos arquivos alterados.
- Execute typecheck ou build conforme o alcance; use a suíte completa para mudanças transversais.
- Execute o teste arquitetural depois de rebase/merge e antes do merge.
- Revise o diff completo e execute `git diff --check`.
- Informe warnings, falhas preexistentes e comandos não executados.

Se o resumo ou o padrão completo obrigatório não estiver acessível, não altere código: informe o
bloqueio e solicite acesso ou uma cópia atualizada.

# Rollout das proteções arquiteturais nos back-ends

Este checklist operacional complementa o
[padrão global](https://github.com/uliving-student/.github/blob/main/profile/BACKEND_STANDARDS.md).
Ele não redefine regras arquiteturais.

## Objetivo

Garantir que pessoas e agentes de código recebam as mesmas instruções e que violações sejam
bloqueadas pelo CI antes do merge.

## Adoção obrigatória por repositório

1. Criar ou revisar `AGENTS.md` na raiz:
   - exigir o resumo operacional antes de qualquer alteração;
   - exigir o padrão completo no primeiro trabalho ou nos cenários de maior risco;
   - manter apenas responsabilidade, riscos e exceções locais;
   - impedir alteração de código quando o padrão estiver inacessível.
2. Criar ou revisar `CLAUDE.md` na raiz:
   - apontar para o resumo, o padrão completo e o `AGENTS.md` local;
   - exigir nova leitura após rebase, merge ou troca de branch, mas não em retomada simples;
   - proibir aumento de baseline para apenas fazer o teste passar.
3. Criar um comando estável `test:architecture` no `package.json`.
4. Executar `test:architecture` em etapa própria do CI, antes da suíte completa.
5. Tornar o workflow de validação obrigatório na proteção da branch principal.
6. Confirmar que o template de PR inclui as verificações arquiteturais.

## Cobertura mínima do teste arquitetural

O teste deve falhar quando surgir:

- import, injeção ou chamada de repository pertencente a outro módulo;
- import de infraestrutura ou entidade ORM de outro módulo, inclusive `import type`;
- registro de entidade externa em `TypeOrmModule.forFeature`;
- export de repository por um módulo;
- dependência `domain -> application/infrastructure/presentation`;
- dependência `application -> infrastructure/presentation`;
- crescimento não autorizado de baseline arquitetural.

## Tratamento do legado

- Registre a dívida atual em baseline explícito e revisável.
- O baseline é uma catraca: alterações normais podem mantê-lo ou reduzi-lo.
- Não atualize o baseline com novas violações.
- Exceção temporária exige justificativa, responsável, prazo ou critério de remoção e `TODO(JIRA)`
  quando houver rollout entre repositórios.

## Validação após rebase ou merge

1. Compare alterações recebidas em imports, `*.module.ts`, entidades ORM e baselines.
2. Resolva conflitos preservando a alternativa mais desacoplada.
3. Execute `npm run test:architecture` antes de continuar a implementação.
4. Execute typecheck e testes focados dos módulos afetados.
5. Registre no PR regressões vindas da branch-base, mesmo quando corrigidas na mesma branch.

## Critério de conclusão do rollout

Um backend está protegido somente quando os dois arquivos de instrução estão na raiz, o teste
arquitetural cobre as regras mínimas, o CI executa esse teste explicitamente e o workflow é obrigatório
na branch principal. Documentação sem gate de CI não conclui a adoção.

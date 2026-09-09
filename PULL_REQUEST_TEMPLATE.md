## Ticket

- Jira: ULT-___

## Contexto

<!-- Qual problema esta alteração resolve e por que ela é necessária? -->

## O que foi feito

-

## Como testar

1.

## Evidências

<!-- Prints, GIFs, requests/responses ou logs sanitizados, quando aplicável. -->

## Riscos e mitigação

<!-- Considere API, banco, migrations, segurança, rollback, feature flag e integrações. -->

## Checklist

- [ ] Li o padrão global de back-ends e as instruções locais antes de alterar o código.
- [ ] Após o último rebase/merge da branch-base, executei novamente os testes arquiteturais aplicáveis.
- [ ] Não injetei/importei repository, entidade ORM ou infraestrutura pertencente a outro módulo.
- [ ] Nenhum baseline arquitetural aumentou; qualquer exceção está justificada, tem responsável e
      critério de remoção.
- [ ] Testes necessários foram adicionados ou atualizados; caso contrário, justifiquei a ausência.
- [ ] Lint, typecheck, testes e build aplicáveis foram executados.
- [ ] A documentação foi atualizada quando necessário.
- [ ] Impactos no padrão global possuem TODO e card Jira, ou uma justificativa de não aplicabilidade.
- [ ] Não foram incluídos segredos nem dados pessoais, financeiros ou de produção.
- [ ] Migrations novas foram revisadas e nenhuma migration já aplicada foi alterada.
- [ ] Alterações incompatíveis de API ou eventos foram identificadas e comunicadas.
- [ ] O plano de rollback foi considerado para mudanças de maior risco.

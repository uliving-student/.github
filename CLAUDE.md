# Instruções obrigatórias para Claude

Antes de alterar código de qualquer backend da uliving, leia o resumo operacional:

<https://github.com/uliving-student/.github/blob/main/profile/BACKEND_STANDARDS_QUICK.md>

Leia também o [padrão completo](https://github.com/uliving-student/.github/blob/main/profile/BACKEND_STANDARDS.md)
no primeiro trabalho no repositório e em mudanças de arquitetura, módulos, persistência, contratos,
eventos, migrations, segurança, rebase/merge ou baseline arquitetural.

Em seguida, leia o `CLAUDE.md` e o `AGENTS.md` existentes na raiz do backend afetado. Instruções de
prompt não substituem o padrão global.

Se um documento obrigatório não estiver acessível, não altere o código. Informe o bloqueio e
solicite acesso ou uma cópia atualizada.

Depois de rebase, merge ou troca de branch, releia o resumo e as seções completas relacionadas ao
diff. Na retomada da mesma tarefa, releia somente se as instruções ou a branch-base tiverem mudado.

Antes de implementar mudança de alto impacto em regra de negócio, pergunte explicitamente ao usuário
se deve usar feature flag e aguarde a decisão.

Este arquivo protege trabalhos feitos diretamente no repositório `.github`. Para carregamento
automático em outro backend, esse backend também deve manter um `CLAUDE.md` na própria raiz apontando
para o padrão canônico.

# Instruções para agentes — repositório organizacional

Antes de alterar qualquer backend da uliving, leia o resumo operacional:

<https://github.com/uliving-student/.github/blob/main/profile/BACKEND_STANDARDS_QUICK.md>

Leia também o [padrão completo](https://github.com/uliving-student/.github/blob/main/profile/BACKEND_STANDARDS.md)
no primeiro trabalho no repositório e em mudanças de arquitetura, módulos, persistência, contratos,
eventos, migrations, segurança, rebase/merge ou baseline arquitetural.

Depois leia o `AGENTS.md` local do repositório afetado. O padrão global prevalece; o arquivo local
mantém somente responsabilidades, riscos e exceções específicas do serviço.

Se um documento obrigatório não estiver acessível, não altere código de backend. Informe o bloqueio
e peça acesso ou uma cópia atualizada.

Após rebase, merge ou troca de branch, releia o resumo e as seções completas relacionadas ao diff.
Na retomada da mesma tarefa, releia somente se as instruções ou a branch-base tiverem mudado.

Antes de implementar mudança de alto impacto em regra de negócio, pergunte explicitamente ao usuário
se deve usar feature flag e aguarde a decisão.

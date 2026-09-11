# 🤝 Contribuindo — uliving

Este guia descreve como colaborar nos repositórios da uliving.

---

## ✅ Antes de começar

- Verifique se existe ticket no Jira (ex.: `ULT-123`)
- Alinhe escopo e critérios de aceitação
- Confirme impacto em outras squads/sistemas

---

## 🌿 Branching

Siga o padrão descrito em [ENGINEERING.md](ENGINEERING.md#-padrão-de-branch-jira).

O fluxo é trunk-based: crie uma branch curta a partir de `main` e abra um único PR de volta para
`main`. Staging é ambiente, não branch de integração.

---

## 🔀 Pull Requests

Siga o processo descrito em [ENGINEERING.md](ENGINEERING.md#-pull-request-pr).

---

## 🧪 Qualidade

- Rode testes e linters localmente quando possível
- Garanta que pipelines estejam verdes
- Documente mudanças que impactem operação/suporte

---

## 🧾 Definition of Done

Consulte [ENGINEERING.md](ENGINEERING.md#-definition-of-done-dod).

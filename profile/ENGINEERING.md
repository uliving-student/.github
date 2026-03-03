
# 🧭 Guia de Engenharia — uliving

Este documento define padrões mínimos para manter **qualidade, previsibilidade e velocidade** no desenvolvimento.

> Regra de ouro: **se não está testável, revisável e observável, não está pronto.**

---

## 🌿 Padrão de Branch (Jira)

Toda branch deve estar vinculada a um ticket do Jira.

### Formato

`<tipo>/ULI-<id>-<descricao-curta>`

### Exemplos

- `feature/ULI-245-criar-endpoint-reservas`
- `fix/ULI-310-corrigir-bug-checkout`
- `refactor/ULI-522-refatorar-servico-pagamentos`
- `chore/ULI-400-ajustar-eslint`

### Tipos permitidos

- `feature` → nova funcionalidade
- `fix` → correção de bug
- `hotfix` → correção urgente em produção
- `refactor` → refatoração sem mudança de comportamento esperado
- `chore` → tarefas técnicas/infra/ajustes
- `docs` → documentação
- `test` → ajustes de testes

---

## 🧾 Commits (recomendado)

Recomendamos **Conventional Commits** para facilitar changelog e automações:

`<tipo>(escopo opcional): mensagem`

Exemplos:

- `feat(reservas): criar endpoint de consulta`
- `fix(checkout): corrigir cálculo de desconto`
- `chore(ci): ajustar pipeline`

Tipos comuns: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`.

---

## 🔀 Pull Request (PR)

Todo código deve passar por PR antes de ir para a branch principal.

### Requisitos mínimos

- Link do ticket Jira (ex.: `ULI-123`)
- Descrição clara do problema e da solução
- Build/linters OK
- Testes atualizados (ou justificativa explícita quando não aplicável)
- 1+ aprovação (ou política do time)
- Sem segredos no código (tokens/keys)

### Checklist de qualidade

- [ ] Código legível (nomes bons, funções pequenas)
- [ ] Sem duplicação desnecessária
- [ ] Tratamento de erros consistente
- [ ] Logs/observabilidade adequados (quando aplicável)
- [ ] Migrações versionadas (quando aplicável)
- [ ] Impacto em performance e custo considerado (quando aplicável)

### Como escrever uma boa descrição de PR

Inclua:

1. **Contexto:** o que motivou a mudança
2. **O que foi feito:** bullets objetivos
3. **Como testar:** passo a passo
4. **Evidências:** prints/gifs/logs quando for UI
5. **Riscos e mitigação:** migrações, rollback, flags

> Dica: PR pequeno é PR rápido.

---

## 🧼 Clean Code (padrões mínimos)

### Nomes intencionais

- Variáveis e funções devem dizer **o que** fazem, não **como**
- Evite abreviações obscuras
- Prefira `calculateMonthlyInvoice()` a `calcInv()`

### Funções pequenas

- Uma função deve ter **um motivo** para mudar (SRP)
- Se ficou difícil de nomear, provavelmente faz coisa demais

### Evite estados implícitos

- Prefira passar dados como parâmetros
- Evite dependências globais e efeitos colaterais escondidos

### DRY com cuidado

- Não abstraia cedo demais
- Extraia duplicações quando houver **padrão claro**, não só coincidência

### Erros são parte do fluxo

- Trate erros explicitamente
- Crie mensagens úteis para debug
- Em APIs: respostas consistentes (status + payload)

---

## 🧪 Testes

### O que testamos

- **Unitários:** regras de negócio e validações
- **Integração:** endpoints, repositórios, filas, jobs
- **Contrato:** integrações com serviços externos quando fizer sentido

### Princípios

- Teste deve ser **determinístico**
- Evite testes frágeis de UI; priorize testes de comportamento
- Dê preferência a testes no **domínio/casos de uso**

---

## 🔐 Segurança (mínimo)

- Nunca commitar secrets (use variáveis de ambiente/secret manager)
- Validar inputs (DTOs/validators)
- Sanitizar logs (não logar dados sensíveis)
- Dependências atualizadas (patches de segurança)

---

## 📦 Versionamento e Releases (sugestão)

- SemVer quando aplicável (principalmente bibliotecas)
- Changelog automatizado via Conventional Commits (opcional)
- Deploy com tags/versões rastreáveis

---

## ✅ Definition of Done (DoD)

Uma tarefa só é “done” quando:

- Implementação concluída e revisada
- Testes adequados e pipeline verde
- Observabilidade básica (logs/alerts quando necessário)
- Documentação atualizada (quando necessário)
- Ticket Jira com evidências de validação (quando aplicável)

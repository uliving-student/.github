---

# 🧭 Diretrizes de Engenharia de Software

Nosso Departamento de Tecnologia segue padrões bem definidos para garantir qualidade, escalabilidade e manutenibilidade do código.

---

## 🌿 Padrão de Branch (Jira)

Todas as branches devem estar vinculadas a um ticket do Jira.

### 📌 Formato

tipo/ULI-123-descricao-curta

### 📍 Exemplos

feature/ULI-245-criar-endpoint-reservas  
fix/ULI-310-corrigir-bug-checkout  
chore/ULI-400-ajustar-config-eslint  

### 🎯 Tipos permitidos

- feature → nova funcionalidade  
- fix → correção de bug  
- chore → tarefas técnicas  
- refactor → refatoração  
- hotfix → correção urgente em produção  

---

## 🔀 Pull Requests

Todo código deve passar por Pull Request antes de ir para a branch principal.

### ✅ Requisitos para PR

- Branch atualizada com main
- Build passando
- Testes atualizados
- Code review aprovado (mínimo 1 aprovador)
- Descrição clara do que foi feito
- Link do ticket Jira

### 📝 Template de PR deve conter:

- Contexto
- O que foi feito
- Como testar
- Evidências (prints se necessário)

---

## 🧼 Clean Code

Seguimos os princípios de Clean Code:

- Nomes descritivos e intencionais
- Funções pequenas e com responsabilidade única
- Evitar comentários desnecessários (o código deve ser autoexplicativo)
- Baixo acoplamento
- Alta coesão
- Evitar duplicação (DRY)
- Tratamento adequado de erros

---

## 🏗️ Clean Architecture

Adotamos princípios de arquitetura limpa para manter o domínio isolado da infraestrutura.

### 📐 Camadas

- Domain → regras de negócio
- Application → casos de uso
- Infrastructure → banco, APIs externas, frameworks
- Presentation → controllers / UI

### 📌 Regras importantes

- O domínio não depende de frameworks
- Dependências apontam sempre para dentro
- Inversão de dependência (Dependency Inversion Principle)
- Separação clara entre regra de negócio e implementação técnica

---

## 🧪 Testes

- Testes unitários para regras de negócio
- Testes de integração para fluxos críticos
- Cobertura mínima recomendada: 70%

---

## 🚀 Cultura de Engenharia

- Código é responsabilidade coletiva
- Melhorar o código é parte do trabalho
- Refatoração é contínua
- Documentação faz parte da entrega
- Qualidade > velocidade

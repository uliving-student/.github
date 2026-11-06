# Pacotes privados do GitHub (`@uliving-student/*`)

Como instalar os pacotes privados sem colocar token no repositório, e o que fazer quando um token
vaza. Ticket: ULT-2390.

## Como funciona

- Os pacotes `@uliving-student/*` ficam no GitHub Packages. O `.npmrc` de cada serviço aponta o
  escopo para `https://npm.pkg.github.com` e lê o token da variável de ambiente `NODE_AUTH_TOKEN`:

  ```
  @uliving-student:registry=https://npm.pkg.github.com
  //npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
  ```

  O arquivo não tem segredo, então pode ficar no repositório. **O token nunca vai dentro dele.**
- No CI, o `npm ci` recebe `NODE_AUTH_TOKEN` a partir de um secret do GitHub Actions.
- Na imagem Docker, o token entra como secret do BuildKit, nunca como `ARG` ou `ENV`. O README de
  cada serviço tem a seção "Autenticação no GitHub Packages".

## Instalar na sua máquina

Defina a variável só no terminal em que for instalar:

```bash
# Bash / Git Bash
export NODE_AUTH_TOKEN="$(gh auth token)"
npm ci
```

```powershell
# PowerShell
$env:NODE_AUTH_TOKEN = (gh auth token)
npm ci
```

- Precisa de `gh auth login` com acesso à organização. Se o `gh auth token` receber `401`, use um
  token pessoal clássico com o escopo `read:packages` e guarde-o no gerenciador de segredos ou no
  perfil do seu shell, **nunca em arquivo de repositório**.
- Com `NODE_AUTH_TOKEN` vazio, o npm responde `E401 Unauthorized` ao buscar um pacote privado.
- O `.npmrc` do projeto vem antes do `~/.npmrc` na ordem de precedência do npm. Por isso, nos
  serviços que leem `${NODE_AUTH_TOKEN}`, defina a variável em vez de editar arquivo.

## O que não fazer

- Não cole token em `.npmrc`, `.env`, README, código, teste, script, mensagem de commit ou de PR.
- Não troque `${NODE_AUTH_TOKEN}` pelo valor "só para funcionar". Antes de `git add -A`, confira
  `git status` e `git diff .npmrc`.
- Não use `ARG` ou `ENV` para o token no Dockerfile: ele fica gravado nas camadas da imagem.

## A trava automática

Cada serviço tem `scripts/check-no-secrets.sh` e o workflow **Secret guard**, que roda em pull
request e em push na `main` e verifica:

1. os arquivos versionados;
2. as linhas **adicionadas** nos commits do PR. Assim pega o token que entra num commit e sai no
   seguinte: apagar o arquivo depois não tira o token do histórico.

Padrões: token do GitHub (`ghp_`, `gho_`, `ghu_`, `ghs_`, `ghr_` com 36 ou mais caracteres),
`github_pat_`, `npm_` e `_authToken=` com valor literal. O `_authToken=${NODE_AUTH_TOKEN}` passa. A
verificação nunca imprime o valor encontrado: só o arquivo e a linha.

Para rodar localmente, de dentro do repositório:

```bash
bash scripts/check-no-secrets.sh
bash scripts/check-no-secrets.sh --range origin/main..HEAD
```

Exemplo falso em teste (nunca credencial real): ponha `secret-guard:allow` na linha.

Limites: a trava pega padrões conhecidos, não qualquer segredo, e só avisa depois do push. Ela não
substitui o secret scanning do GitHub nem revisão de PR. Recomenda-se marcar **Secret guard** como
check obrigatório da `main`.

## Se um token vazou

1. **Revogue agora** (GitHub → Settings → Developer settings → Personal access tokens). Se o commit
   ainda não foi enviado, desfaça o commit; se foi enviado, considere o token comprometido.
2. Gere um token novo e atualize o secret do CI, se for o token usado por ele.
3. Apagar o arquivo ou reescrever o histórico **não basta**: clones, forks e caches já podem ter o
   valor. Avise o time.
4. Procure o valor nos logs só por contagem (por exemplo, `gcloud logging read`), sem copiá-lo para
   chat, card ou PR.

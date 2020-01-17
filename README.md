# changelog-cicd
![Status Badge do Gerador de CHANGELOG](https://github.com/klauskpm/changelog-cicd/workflows/Gerador%20de%20CHANGELOG/badge.svg)

Repositório com passo-a-passo de como gerar um CHANGELOG automaticamente.

Esse passo-a-passo é a versão muito resumida do artigo: [Como gerar CHANGELOG automaticamente]()

## Gerando um CHANGELOG automaticamente

### 1) Instale as dependências

```sh
npm install --global commitizen

# dentro do seu projeto
npm install --save-dev husky @commitlint/cli @commitlint/config-conventional standard-version
```

### 2) Configure o commitizen

```sh
# dentro do seu projeto
commitizen init cz-conventional-changelog --save-dev --save-exact
```

### 3) Crie o arquivo `commitlint.config.js`

```js
module.exports = {
    extends: ['@commitlint/config-conventional']
};
```

### 4) Altere o seu `package.json`

```json
{
  ...,
  // opcional
  "repository": {
    "url": "git@gitlab.com:meu-usuario/meu-repo.git"
  },
  ...,
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "scripts": {
    "release": "standard-version"
  }
}
```

### 5) Crie o arquivo `.versionrc`

```json
{
  "types": [
    {"type": "feat", "section": "Funcionalidades"},
    {"type": "fix", "section": "Errors Corrigidos"},
    {"type": "chore", "hidden": true},
    {"type": "docs", "hidden": true},
    {"type": "style", "hidden": true},
    {"type": "refactor", "hidden": true},
    {"type": "perf", "section": "Melhorias de Performance"},
    {"type": "test", "hidden": true}
  ],
  "releaseCommitMessageFormat": "chore(release): {{currentTag}} [skip ci]"
}
```

### 5.1) Faça uma release inicial antes de automatizar (OPCIONAL)

Caso queira, pode criar uma release com a versão atual do projeto, antes de adicionar um processo de automatização.

```sh
npm run release -- --first-release
```

### 6) Crie o `.github/workflows/gerador-de-changelog.yml`

```yml
name: Gerador de CHANGELOG

# só executa no push de commit da branch master
on:
  push:
    branches:
      - master

jobs:
  build:
    # só deixa executar se o último commit não conter 'skip ci' na mensagem
    if: "!contains(github.event.head_commit.message, 'skip ci')"

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v1

    # configura o GITHUB_TOKEN
    # https://help.github.com/en/actions/automating-your-workflow-with-github-actions/authenticating-with-the-github_token
    - name: Configura o GitHub token
      uses: fregante/setup-git-token@v1
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
        name: "Gerador de changelog"
        email: "changelog@users.noreply.github.com"

    # instala a versão 11.x do Node.js
    - name: Instala o Node.js 11.x
      uses: actions/setup-node@v1
      with:
        node-version: 11.x

    # instala as dependências, vai para a master
    # executa o standard-version, e envia o commit e tag
    - name: Gera o CHANGELOG
      run: |
        npm ci
        git checkout master
        npm run release
        git push origin master --follow-tags
```

### 7) Adicione a badge do workflow no `README.md`

```md
https://github.com/<OWNER>/<REPOSITORY>/workflows/<WORKFLOW_NAME>/badge.svg
```

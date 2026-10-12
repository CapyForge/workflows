# CapyForge — workflows

Workflows reutilizáveis do GitHub Actions para os projetos da **CapyForge**. Os repositórios dos projetos (criados a partir do [template Spring Boot](https://github.com/CapyForge/template-spring-boot)) não guardam a lógica de CI: só arquivos curtos que chamam estes workflows na versão principal `v1`. Uma correção publicada aqui chega a todos os projetos de uma vez.

## Workflows

| Workflow | O que faz | Permissões que quem chama concede |
|---|---|---|
| `source-branch.yml` | A `main` só recebe `release/X.Y.Z` e `hotfix/X.Y.Z`. Em PR para a `main`, confere a branch de origem e, se falhar, comenta no PR explicando (input `comment`, padrão `true`). Em push na `main`, confere que o commit veio de um PR de release ou hotfix. Em push em `release/*` ou `hotfix/*`, confere o nome da branch. Fora disso, o job é pulado. | `contents: read`, `pull-requests: write` |
| `maven-verify.yml` | `./mvnw verify` com Eclipse Temurin (input `java-version`, padrão `25`). | `contents: read` |
| `release-tag.yml` | No merge de `release/X.Y.Z` ou `hotfix/X.Y.Z` na `main`, cria a tag `vX.Y.Z` e uma Release do GitHub (só no primeiro merge). Com `major-tag: true`, move também a tag `vX`. | `contents: write` |
| `backport.yml` | No mesmo merge, abre o PR `backport/X.Y.Z` → `develop` e aprova o CI dele. Exige a opção "Allow GitHub Actions to create and approve pull requests". | `contents: write`, `pull-requests: write`, `actions: write` |

Os checks aparecem como `<job de quem chama> / <job do workflow>`. No template, por exemplo: `source-branch / check` e `verify / maven`.

## Exemplo de uso

```yaml
name: CI
on:
  push:
    branches: [main, develop, 'release/**', 'hotfix/**']
  pull_request:
permissions:
  contents: read
jobs:
  source-branch:
    uses: CapyForge/workflows/.github/workflows/source-branch.yml@v1
    permissions:
      contents: read
      pull-requests: write
  verify:
    needs: source-branch
    if: ${{ !failure() && !cancelled() }}
    uses: CapyForge/workflows/.github/workflows/maven-verify.yml@v1
```

## Versões

- Cada versão é uma tag `vX.Y.Z` com uma Release do GitHub. A tag principal (`v1`) acompanha a versão mais recente da mesma linha.
- Correções e melhorias compatíveis entram na `v1` e chegam a todos os projetos. Mudanças incompatíveis (nome de job, inputs, permissões) vão para uma nova versão principal (`v2`), e cada projeto migra quando quiser.

## Contribuição

Fluxo de branches da CapyForge: PRs para a `develop` (squash); `release/X.Y.Z` e `hotfix/X.Y.Z` para a `main` (merge commit). O merge na `main` cria a tag, a Release, move a `v1` e abre o backport. Decisões no repositório [`CapyForge/docs`](https://github.com/CapyForge/docs).

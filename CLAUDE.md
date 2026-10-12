# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`CapyForge/workflows` is a **public** repository of reusable GitHub Actions workflows for **CapyForge** projects (a managed PaaS for Java + Spring Boot apps). Customer repositories, scaffolded from `CapyForge/template-spring-boot`, keep only thin caller workflows that reference these files at the major tag `@v1`, so a fix here reaches every project at once (Decision 32 in the docs repo). Because every customer CI depends on it, this repository is a public contract: within a major version, never rename jobs or inputs, change required permissions or break callers; incompatible changes go to a new major (`v2`).

## The documentation repository is the source of truth

Decisions live in **`E:\Dev\capyforge\docs`** (GitHub `CapyForge/docs`): start with its `CLAUDE.md`; the branch flow is Decision 31 and the centralized CI is Decision 32 in `capyforge-decisoes-pendentes-e-proximos-passos.md`; template details in `architecture/vertical-slice.md` §4. Run `git -C E:/Dev/capyforge/docs pull` first. If a change here alters anything approved, present it to the user first and record it in the docs repo after approval.

## Workflows

- `source-branch.yml` (job `check`): only `release/X.Y.Z`/`hotfix/X.Y.Z` reach `main`; PR check + one pt-BR comment on failure (input `comment`), merge-commit check on push to `main` via the commits→pulls API, name check on pushes to `release/*`/`hotfix/*`; skipped otherwise. Callers gate their build on it with `needs` + `if: ${{ !failure() && !cancelled() }}`.
- `maven-verify.yml` (job `maven`): `./mvnw verify`, input `java-version` (default `25`).
- `release-tag.yml` (job `tag`): tag `vX.Y.Z` + GitHub Release on merge into `main`; input `major-tag` moves `vX` (this repo uses it).
- `backport.yml` (job `backport`): opens `backport/X.Y.Z` → `develop` and approves its `action_required` CI runs.
- Reusable workflows never declare permissions; callers grant them (README table). Check names are `<caller job> / <called job>`; the template's required checks are `source-branch / check` and `verify / maven`.
- Own CI: `ci.yml` (local `source-branch` + `actionlint`); own releases: `release.yml` (local `release-tag` with `major-tag: true` + `backport`). Nested `uses: ./...` resolves to this repository at the same commit.

## Working agreements

- Talk to the user in Portuguese (pt-BR); code, identifiers and commit messages in English; README in pt-BR.
- Branch flow (Decision 31): `develop` default, squash into `develop`; only `release/X.Y.Z`/`hotfix/X.Y.Z` into `main` by merge commit; merge only via PR. Commit or push only when asked; conventional commits with a bullet body and the attribution line requested by the harness.
- Do not record version numbers in docs: versions are the GitHub Releases/tags.
- Lint locally: `MSYS_NO_PATHCONV=1 docker run --rm -v "E:/Dev/capyforge/workflows:/repo" -w /repo rhysd/actionlint:1.7.12 -color`.
- Changes reach customers only through the major tag: test on a template PR pointing to a branch ref before releasing.
- Environment: Windows 11; Bash tool is Git Bash; `jq` not installed; GitHub CLI at `C:\Program Files\GitHub CLI\gh.exe`.

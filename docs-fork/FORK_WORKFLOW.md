---
title: "Fork Workflow"
description: "Workflow de fork: sincronização com upstream e gerenciamento de modificações pessoais"
lastUpdated: 2026-06-09
---

# Fork Workflow

Este documento descreve o fluxo de trabalho para gerenciar um fork do OmniRoute com o
repositório original (`diegosouzapw/OmniRoute`), permitindo:

- Manter `main` sempre sincronizado com o `upstream/main`
- Trabalhar em `develop` com modificações próprias
- Enviar contribuições seletivas via PR para o upstream
- Manter alterações pessoais/estratégicas isoladas no fork

## Arquitetura de Branches

```
upstream/main (repositório original: diegosouzapw/OmniRoute)
    │
    │  git pull (via main local)
    │
    ▼
main (local) ─── tracking upstream/main
    │
    │  git merge main (para atualizar develop com a base mais recente)
    │
    ▼
develop (seu fork: ViFigueiredo/OmniRoute)
    ├── features/ (branches de feature para PR ao upstream)
    ├── configs/ (branches com configurações pessoais)
    └── modificações diretas (pequenos ajustes locais)
```

### Branches

| Branch     | Tracking               | Finalidade                                        |
| ---------- | ---------------------- | ------------------------------------------------- |
| `main`     | `upstream/main`        | Espelha o repositório original                    |
| `develop`  | `origin/develop`       | Base de trabalho para todas as suas modificações  |

### Remotes

| Remote     | URL                                              | Uso                        |
| ---------- | ------------------------------------------------ | -------------------------- |
| `origin`   | `https://github.com/ViFigueiredo/OmniRoute.git`  | Seu fork (leitura/escrita) |
| `upstream` | `https://github.com/diegosouzapw/OmniRoute.git`  | Original (leitura)         |

## Convenções

- `main` **nunca** recebe commits diretos. Tudo que está em `main` veio do `upstream/main`.
- `develop` é onde você trabalha. Recebe merge de `main` para se manter atualizado.
- Branches de feature para contribuição devem ser criadas a partir de `main`, não de `develop`.
- Modificações pessoais/estratégicas ficam no `develop` ou em branches derivadas dele, e **nunca são enviadas ao upstream**.

## Rotina de Sincronização

Sempre que for iniciar um trabalho, siga estas verificações:

```bash
# 1. Verifique se upstream/main teve atualizações
git fetch upstream

# 2. Se sim, sincronize main local
git checkout main
git pull upstream main          # fast-forward para o estado mais recente

# 3. Atualize develop com a nova base
git checkout develop
git merge main                  # traz as novidades do upstream para develop
```

Após a sincronização, verifique o que está pendente em develop:

```bash
git log main..develop --oneline
```

Isso mostra apenas os commits que existem em develop mas não em main (suas modificações).

## Verificação de Estado

Use essa checagem para ter uma visão clara do estado atual:

```bash
# 1. upstream/main atualizou?
git log main..upstream/main --oneline
# Se retornar commits → está desatualizado, precisa sincronizar

# 2. O que em develop já foi PRado e aceito no upstream?
git log upstream/main --oneline | grep -f <(git log main..develop --oneline --format="%s")
# Commits que aparecem aqui são seus PRs já aceitos

# 3. O que em develop ainda é só seu (não está no upstream)?
git log upstream/main..develop --oneline --no-merges
# Esses são seus commits pendentes (PRáveis ou pessoais)
```

## Como Contribuir com PR ao Upstream

Sempre que quiser contribuir uma modificação de volta ao repositório original:

```bash
# 1. Parta de main (base limpa e atualizada)
git checkout main
git pull upstream main

# 2. Crie uma branch específica para a contribuição
git checkout -b feature/minha-nova-feature

# 3. Faça os commits normalmente
git add .
git commit -m "feat: descrição da feature"

# 4. Envie para seu fork
git push origin feature/minha-nova-feature

# 5. Abra um Pull Request no GitHub
#    head: ViFigueiredo/feature/minha-nova-feature
#    base: diegosouzapw/main
```

Depois que o PR for aceito no upstream:

```bash
# Atualize main (agora o commit já faz parte do upstream/main)
git checkout main
git pull upstream main

# Atualize develop
git checkout develop
git merge main

# Apague a branch de feature (opcional, já que o commit está no histórico)
git branch -D feature/minha-nova-feature
```

> **A branch de feature pode ser deletada** porque o commit já está no `main` via `upstream/main`.
> Em `develop`, o mesmo commit pode chegar por duas vias: via `git merge main` (se já foi
> aceito no upstream) ou via merge direto da branch de feature. O git lida com isso
> naturalmente sem duplicar.

## Como Manter Modificações Pessoais

Para modificações que **não devem ir para o upstream** (configurações pessoais, scripts
deploy, secrets locais, estratégias próprias):

### Opção 1: Diretamente no develop (recomendado para ajustes pequenos)

```bash
git checkout develop
# Faça suas alterações pessoais
git add .
git commit -m "chore: minha config pessoal"
```

### Opção 2: Branch separada derivada do develop (recomendado para mudanças maiores)

```bash
git checkout develop
git checkout -b configs/minha-estrategia
# Faça suas alterações
git add .
git commit -m "chore: estratégia pessoal X"
```

Depois, mantenha a branch atualizada com develop:

```bash
git checkout configs/minha-estrategia
git merge develop
```

### Cuidados ao fazer merge de main

Quando você faz `git merge main` em `develop`, só sobem os commits do upstream.
Suas modificações pessoais permanecem intactas. Use `git log main..develop`
para ver o que é exclusivamente seu.

## Resumo do Checklist

| Ação                                           | Comando                                                        |
| ---------------------------------------------- | -------------------------------------------------------------- |
| Verificar se upstream atualizou                | `git log main..upstream/main --oneline`                        |
| Sincronizar main com upstream                  | `git checkout main && git pull upstream main`                  |
| Atualizar develop com a base mais recente      | `git checkout develop && git merge main`                       |
| Ver o que é só seu no develop                  | `git log main..develop --oneline`                              |
| Ver o que é só seu no develop (sem merges)     | `git log upstream/main..develop --oneline --no-merges`         |
| Enviar develop para o fork                     | `git push origin develop`                                      |
| Criar branch para contribuição                 | `git checkout -b feature/nome` (a partir de `main`)            |
| Abrir PR                                        | `gh pr create --repo diegosouzapw/OmniRoute --head seu:branch` |
| Apagar branch de feature após PR aceito        | `git branch -D feature/nome`                                   |

## Exemplo Prático

```bash
# Cenário: você trabalhou em develop, upstream lançou v3.8.19

# 1. Verificar
git log main..upstream/main --oneline
# 87f4a2e Release v3.8.19  ← tem novidade!

# 2. Sincronizar
git checkout main
git pull upstream main          # main agora está em v3.8.19

# 3. Atualizar develop
git checkout develop
git merge main                  # develop agora tem v3.8.19 + suas modificações

# 4. Ver o que é seu
git log main..develop --oneline --no-merges
# abc1234 feat: minha melhoria local  ← só seu
# def5678 chore: config de deploy     ← só seu

# 5. Se quiser contribuir abc1234 ao upstream:
git checkout main
git checkout -b pr/minha-melhoria
git cherry-pick abc1234          # pega só esse commit
git push origin pr/minha-melhoria
# Abre PR no GitHub
```

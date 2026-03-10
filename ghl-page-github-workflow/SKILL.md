---
name: ghl-page-github-workflow
description: Use when the task is to create a GHL landing page, connect it to GitHub, open a PR, and apply Claude code review suggestions to the final code.
---

# GHL Page → GitHub → PR → Review → Apply

## Overview

Workflow completo para criar uma landing page GHL, publicar no GitHub e aplicar revisão automatizada via Claude Code Review antes de finalizar o código.

## Fluxo em 3 Etapas

```
[1. CRIAR PÁGINA] → [2. CONECTAR GITHUB] → [3. PR + REVIEW + APLICAR]
```

---

## Etapa 1 — Criar a Página com ghl-landing-page

**Invoke obrigatório:**
```
Skill: ghl-landing-page
```

Siga todas as instruções da skill antes de avançar. Somente prossiga para a Etapa 2 quando o arquivo `index.html` estiver completo e funcional localmente.

**Checklist antes de avançar:**
- [ ] `index.html` único criado (sem build tools)
- [ ] `.ghl-wrapper` com full-width breakout presente
- [ ] Animações e interações implementadas
- [ ] Testado no navegador local

---

## Etapa 2 — Conectar ao GitHub com /install-github-app

Após a página criada, execute **na ordem exata**:

```
1. Inicialize o repositório:
   git init && git checkout -b main

2. Crie o repositório no GitHub e faça o push inicial (main com README):
   gh repo create <nome-repo> --public --source=. --remote=origin --push

3. Execute o comando:
   /install-github-app
```

**Aguarde a confirmação:**
```
GitHub Actions setup complete!
```

Somente avance para a Etapa 3 após essa confirmação. Se falhar, verifique permissões do `gh auth status` e tente novamente.

---

## Etapa 3 — PR + Claude Review + Aplicar Mudanças

### 3a. Criar branch de feature e abrir o PR

```bash
# Branch com a página
git checkout -b feat/landing-page
git add index.html
git commit -m "feat: add landing page"
git push -u origin feat/landing-page

# Abrir PR para main
gh pr create --title "feat: landing page" --base main --head feat/landing-page --body "..."
```

### 3b. Aguardar o Claude Code Review

O GitHub Action roda automaticamente ao abrir o PR. Aguarde os comentários de revisão aparecerem no PR antes de continuar.

Para verificar:
```bash
gh pr checks <número-do-pr>
```

### 3c. Aplicar as mudanças sugeridas

Leia cada comentário do review e aplique no `index.html`:

```bash
gh pr view <número-do-pr> --comments
```

Após aplicar:
```bash
git add index.html
git commit -m "apply: claude review suggestions"
git push
```

O PR será atualizado automaticamente e o review rodará novamente para confirmar.

---

## Erros Comuns

| Situação | Solução |
|----------|---------|
| `/install-github-app` falha com "Failed to access repository" | Repositório precisa existir no GitHub antes. Execute o `gh repo create` primeiro. |
| GitHub Action não roda | Verifique se o `.github/workflows/` foi criado pelo `/install-github-app` e está commitado em `main`. |
| PR sem comentários do review | O Action pode estar em fila. Aguarde ou verifique `gh run list`. |
| Review sugere mudança já feita | Ignore e faça push — o review re-roda e confirma. |

---

## Quick Reference

```
ghl-landing-page skill  →  index.html pronto
/install-github-app     →  GitHub Actions configurado
feat branch + PR        →  review automático dispara
gh pr view --comments   →  lê sugestões
git commit + push       →  aplica e re-verifica
```

---
name: repo-proprio-i16br
description: "Repositório próprio do portal (v3.0) — GitHub I16BR/portal-erp, privado; como autenticar e empurrar"
metadata:
  node_type: reference
  type: reference
---

**Criado em 09/07/2026.** O portal passou a viver num repositório **nosso**, não no do cliente.

- **Organização:** `I16BR` (github.com/I16BR) — display "I16BR", razão social "I16 CONSULTING",
  e-mail de contato `mafra@i16br.com`, tipo "business or institution". Dona: a conta pessoal
  **`framafrai16`**. Único membro. Ninguém convidado (Junior/Ricardo NÃO têm acesso).
- **Repositório:** `I16BR/portal-erp` — **privado**. Nome genérico de propósito (não amarra ao cliente).
- **Branch `main`** = o que era `feature/despesas-cotacao`, com a **história completa (429 commits)**,
  dos quais 33 estavam além do `origin/main` do repo antigo. Publicada em `5075c76`.

**Como empurrar (do MacBook):**
- Remoto `i16br` = `git@github-i16br:I16BR/portal-erp.git`. O `origin`
  (`MafraFiori/DAVI_CAMINHOES`, do Ricardo/Junior) continua configurado e **não recebe nada** —
  ver [[equipe-e-governanca-portal]].
- Autenticação por **chave SSH dedicada** `~/.ssh/i16br_github` (ed25519, sem passphrase), com alias
  `Host github-i16br` no `~/.ssh/config` (`IdentitiesOnly yes`) para não misturar com as outras chaves
  nem com a conta `frmafra@gmail.com`. A pública está registrada em Settings > SSH keys de `framafrai16`.
- Identidade local só deste repo: `user.email = mafra@i16br.com`. (Os 8 commits de 09/07 ficaram
  assinados como `mafra@ccskf.net`, que era o config anterior.)

**Gotcha de rede (09/07):** o Mac não tinha DNS próprio (usava o roteador 192.168.15.1, que recusava
consultas de forma intermitente) → `Could not resolve hostname github.com` no meio de operações git.
Corrigido com `sudo networksetup -setdnsservers Wi-Fi 1.1.1.1 8.8.8.8` + flush. Ainda falha de vez em
quando; se um comando git der erro de DNS, **repetir** antes de concluir que algo quebrou.

**Backup adicional:** `git bundle` da história completa em `~/backups/portal-davi-*.bundle` (verificado
com `git bundle verify` = "complete history") e cópia em `~/backup-thales/bundles/` no VPS 161.

**Ainda pendente:** o commit `324c496` do `origin/main` (25/06, i18n do Ricardo) **não está** nesta
história — foi decisão consciente. Se quiser trazê-lo, `git fetch origin` é leitura e não expõe nada.
Falta também fazer a imagem Docker ser construída a partir deste repo — ver [[container-161-camada-efemera]].

**Também na org I16BR:** `I16BR/portal-servicos` = portal Mafra Consult (LS Printers / CNC), publicado 09/07 a partir do VPS 161. Deploy keys estão BLOQUEADAS por política da org (a opção nem aparece em Member privileges) → o push sai do Mac, nunca do servidor.

---
name: ambiente-mac-migracao
description: "Ambiente do MacBook (Francisco/Mafra) — máquina de trabalho oficial desde 14/07/2026; paths, deploy, histórico Mac↔Windows"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

**Migração Windows → MacBook feita em 09/07/2026** (o notebook Windows estava travando). O Mafra usa um **MacBook Pro** (`mafra@Mac`, macOS 26.5.1, IP local 192.168.15.171 na rede Wi-Fi "LUMAFRA"). Claude Code já instalado e usado no Mac (node via nvm — só carrega no Terminal interativo, não em SSH não-login).

**Paths no Mac:**
- **Projeto/código:** `~/portal-davi` (branch `feature/despesas-cotacao`), trazido com todo o trabalho de 09/07 (inclui a tela NotasServico e o backend NFS-e). node_modules NÃO veio → rodar `npm install` só se for executar local; para DEPLOY não precisa.
- **Memória (auto-load):** `~/.claude/projects/-Users-mafra-portal-davi/memory/` → por isso **abrir o Claude dentro de `~/portal-davi`** (cwd = /Users/mafra/portal-davi → esse slug).
- **Chave do VPS 161:** `~/.ssh/davi_vps_teste` (perm 600). O `deploy-161.sh` usa `KEY=~/.ssh/davi_vps_teste` → funciona nativo no Mac, sem ajuste. Deploy Mac→161 testado OK.
- **Google Drive:** `~/Library/CloudStorage/GoogleDrive-frmafra@gmail.com` (mesma conta do Windows → sincroniza).
- Backup da memória também em `~/davi-memoria/`.

**Como usar (no Mac):** Terminal → `cd ~/portal-davi` → `claude`. A memória carrega sozinha. Deploy: `bash deploy-161.sh webapp/...` igual no Windows (agora até mais fácil, ssh/scp nativos).

**Sequela do tar (corrigida 09/07):** a migração restaurou **216 diretórios do `.git`** (e outros do projeto) **sem bit de escrita do dono** (`dr-xr-xr-x`) → o repo ficou somente-leitura e nenhum `git commit` funcionava ("Unable to create .git/index.lock"). Corrigido com `find . -type d ! -perm -u+w -exec chmod u+w {} +`. Se aparecer de novo em outra cópia por tar, é isso.

**Trabalho de 09/07 commitado (local):** 8 commits temáticos sobre `39c04c3` na branch `feature/despesas-cotacao` — segurança/.env, Mosaico, Pendencias, Devolução+Nota de Crédito, DRE+Razão, NotasServico, portal/wiki/deploy, e o lote de correções Compras/Estoque/Financeiro. `.gitignore` passou a excluir `/data/` (NF-e/NFS-e reais, `tickets.db`, permissões) e `.claude/`. Nenhum segredo entrou no histórico. Ver [[container-161-camada-efemera]] e [[equipe-e-governanca-portal]].

## Histórico da máquina de trabalho (houve vai-e-volta)
- **09/07 — migrou pro Mac.** Regra: usar UMA máquina (o Mac).
- **10/07 — REVERTIDO pro Windows.** Motivo registrado: o Claude do Mac editou o container do 161 direto (`docker cp`) sem passar pelo fluxo, e havia divergência. O Windows (`D:\portal-davi`) virou fonte única temporária e foi sincronizado (mosaico.js sha `ca069765367d58c2`).
- **14/07 — REVERTIDO DE NOVO pro Mac (decisão do Mafra, VALE ESTA).** 👉 **Este MacBook é a máquina de trabalho e a fonte única da verdade. Windows sai de cena de vez.**

## Regra vigente (a partir de 14/07/2026)
- **Mac = fonte única.** Tudo que escreve (código e memória) sai daqui. Nada de trabalhar/deployar pelo Windows.
- **Tudo versionado no GitHub próprio do Mafra — conta `frmafra-bit`:**
  - **Memória** → repo `frmafra-bit/infra-geral` (chave SSH `~/.ssh/frmafra_bit_github`, host `github-frmafrabit`). É o repo que já existia (populado pelo Windows até 10/07); a partir de 14/07 só o Mac empurra.
  - **Código do portal** → a mover para um repo em `frmafra-bit` (host `github-frmafrabit`). Ver [[repo-proprio-i16br]] (destino anterior era I16BR/portal-erp; consolidar em frmafra-bit).
- **NÃO empurrar** para `MafraFiori/DAVI_CAMINHOES` (repo do Ricardo/Junior) — ver [[equipe-e-governanca-portal]].
- **Windows:** ❌ **NÃO EXISTE MAIS (14/07/2026).** A máquina Windows foi **formatada e virou produto de laboratório** — não trabalha mais no projeto. Logo o Mac é a **ÚNICA** máquina e divergência ficou impossível. A chave `davi-memoria-windows` na conta `frmafra-bit` está **órfã** (a máquina dela não existe) — pode apagar quando quiser.
- **Nunca colocar credencial em código** — usar `.env` (ver [[integracao-qive]]).
- Deploy 161: Frontend → `bash deploy-161.sh webapp/...` (bumpa version.json). Backend → `docker cp` + `docker restart davi_caminhoes` (NÃO bumpa version.json).
- Acesso Windows→Mac por SSH (se precisar): chave `mac_migracao` autorizada no `~/.ssh/authorized_keys` do Mac; Mac em `mafra@192.168.15.171` / `MacBook-Pro-de-Francisco.local` (rede LUMAFRA).

Ver [[clone-thales-vps]] (também há um clone no 161) e [[container-161-camada-efemera]].

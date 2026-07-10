---
name: ambiente-mac-migracao
description: "Ambiente do MacBook (Francisco/Mafra) — projeto migrado do Windows em 09/07/2026; paths, deploy, como usar"
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

**REVERTIDO EM 10/07/2026 — o MAC ESTÁ "DE FÉRIAS"; a máquina de trabalho voltou a ser o WINDOWS (`D:\portal-davi`).** Decisão do Mafra. Motivo: divergência e o Claude do Mac editou o container do 161 direto (`docker cp`) sem passar pelo fluxo. **Regras agora:**
- **Windows = fonte única da verdade.** Não trabalhar/deployar pelo Mac.
- Frontend → `bash deploy-161.sh webapp/...` (bumpa version.json). Backend → `docker cp` + `docker restart davi_caminhoes` (NÃO bumpa version.json).
- **Nunca colocar credencial em código** — usar `.env` (ver [[integracao-qive]]).
- O Windows foi sincronizado em 10/07 com a correção que o Mac fez (mosaico.js sha `ca069765367d58c2`).
- **Trabalho do Mac ainda NÃO auditado localmente** (o Mac estava dormindo/fora da rede). GitHub e VPS foram auditados: nada pushado, `deploy-161.sh` não rodou, só o `mosaico.js` do container foi trocado (a correção do `.env`). Se o Mac voltar, conferir git local dele antes de descartar.
- Acesso Windows→Mac por SSH: chave `mac_migracao` (autorizada no `~/.ssh/authorized_keys` do Mac); Mac em `mafra@192.168.15.171` / `MacBook-Pro-de-Francisco.local` (rede LUMAFRA). Cópia da memória no Mac em `~/.claude/projects/-Users-mafra-portal-davi/memory/` (pode estar defasada).

Ver [[clone-thales-vps]] (também há um clone no 161).

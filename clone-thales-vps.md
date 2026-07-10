---
name: clone-thales-vps
description: "Clone do Thales (Claude Code) rodando no VPS 161 — estado, acesso, memórias e terminal web"
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Objetivo (Mafra, 08/07/2026): ter o Thales acessível de qualquer dispositivo pelo navegador, para melhorar o portal a partir dos erros, e não depender do Windows local (que já travou 2x → medo de perder dados/memórias).

**Backups (contra travar o Windows):**
- Google Drive: `G:\Meu Drive\Projeto SAP Davi TerraMix\_BACKUP_THALES\<data>\` → `memory\` (24 memórias) + `portal-davi_<data>.zip` (código, sem node_modules/.git).
- VPS 161: memórias em `~/davi_caminhoes/.thales-memory/` (24 arquivos). Servidor também tem `rclone` (sync Drive) e `~/claude-backup`.

**Estado do clone no VPS 161 (161.97.185.151, user administrador, SSH porta 22 — NÃO 3003):**
- `claude` 2.1.201 já instalado (`/usr/bin/claude`) e **autenticado** (`~/.claude/.credentials.json`). Já usado (history.jsonl, vários projects: portal-cnc, portal-mafra, etc.).
- Workspace anterior: `~/claude-workspace/` (memórias de OUTROS projetos + index.html). Muitos scripts em `~/*.sh` (backup_portal, setup_terminal=tmux, start_work, etc.).
- Acesso hoje: SSH + tmux (`~/start_work.sh` = `tmux new -A -s infra`).

**Terminal no navegador (pendente — precisa de sudo/senha, que o Thales não manuseia):**
- Script pronto no servidor: `~/setup_thales_web.sh` (instala ttyd + HTTPS auto-assinado + senha + systemd `thales-web` na porta 7681; abre `claude` numa sessão tmux `thales`).
- Mafra roda (senha do terminal fica com ele): `THALES_USER=mafra THALES_PASS='senha-forte' sudo -E bash ~/setup_thales_web.sh`
- Depois acessa: `https://161.97.185.151:7681` (aviso de cert auto-assinado → Avançado > Prosseguir).

**IMPORTANTE p/ SSH via ferramentas Windows:** usar OpenSSH do PowerShell (`& ssh -i $env:USERPROFILE\.ssh\davi_vps_teste ...`); o git-bash desta máquina estava quebrado ("bash.exe not found"). Ver deploy em [[deploy-e-detector-versao-portal]] e governança [[equipe-e-governanca-portal]].

**Acesso pelo celular (Termux) — funcionando desde 08/07/2026:**
- Chave dedicada `thales_celular` (ed25519) instalada no authorized_keys do 161; par salvo em `%USERPROFILE%\.ssh\thales_celular` e no Drive `G:\Meu Drive\Projeto SAP Davi TerraMix\_THALES_CELULAR\` (+ LEIA-ME com passo a passo Termux).
- No servidor: `~/bin/thales` = `tmux new -A -s thales "claude; bash -l"` (sessão persistente); alias no celular: `thales`.

**Avisos no Telegram com data/hora — funcionando:**
- `~/bin/tg "msg"` envia ao chat 6734074491 via bot @Daviportalalertas_bot (mesmo token do vigia, em ~/vigia/vigia.py) com carimbo `dd/mm/aaaa hh:mm:ss`.
- Hooks do Claude no servidor (`~/.claude/settings.json`): Stop → "✅ Terminei a tarefa"; Notification → "🔔 Preciso da sua atenção". Criado do zero (não existia settings.json).

**Pendente:** repo Git privado versionando memórias (escolha do Mafra) — falta auth GitHub. Sincronizar memórias locais↔VPS periodicamente. Terminal web (ttyd na 7681) segue aguardando `sudo ufw allow 7681/tcp`.

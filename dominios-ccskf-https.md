---
name: dominios-ccskf-https
description: Portais dos clientes em <cliente>.ccskf.com.br com HTTPS no VPS 161; davi.ccskf.com.br -> porta 3003 (portal do time)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 92eba152-918b-4363-93e2-2ee7ca5ad176
---

**Padrão de acesso dos portais (montado no VPS Contabo 161, `161.97.185.151`):** cada cliente em `https://<cliente>.ccskf.com.br`. DNS do domínio `ccskf.com.br` é gerido no **KingHost** (painel Gerenciar DNS); os registros A dos clientes apontam pro 161. Reverse proxy = **nginx** na frente de apps Node/Express; HTTPS via **Let's Encrypt/certbot** (renova sozinho).

**⚠️ Existem DOIS "portais Davi" no 161 — não confundir:**
- **Porta 3003** = container docker `davi_caminhoes` (tenant SBO_HOMOLOG) = **o portal UI5 com TODO o trabalho** (Compras, Estoque, Vendas, Fiscal, Contábil). É o que o time (Rafaela/Rute) usa. `<title>Portal SAP</title>`.
- **Porta 3050** = app multi-tenant `portal-mafra`/`server.js` (pm2 `portal-cnc`) que serve **CNC (SBO_CNCGRUPO_H) e LS Printers (SBO_FLOWS4)**, diferenciados pelo header `X-Tenant`. Login em `/portal/login`, `<title>Portal de Serviços</title>`. É outra arquitetura de portal (genérica), NÃO tem os módulos do Davi.

**Decisão 14/07/2026:** `davi.ccskf.com.br` aponta pro **3003** (portal real do time), NÃO pro 3050. Alguém tinha pré-configurado o bloco nginx do davi pra 3050/SBO_DAVI — foi corrigido pra 3003.

**Config nginx:** arquivo `/etc/nginx/sites-enabled/portal-mafra` (é o ATIVO; `sites-available/portal-mafra` divergiu e **não é carregado** — `nginx.conf` só inclui `sites-enabled/*` e `conf.d/*`). Blocos: cnc/ls (3050, 443 ssl), davi (3003, agora com 443 ssl via certbot). Backup pré-mudança em `/home/administrador/portal-mafra.bak-14jul`.

**⚠️ Gotcha do glob:** `include /etc/nginx/sites-enabled/*` pega QUALQUER arquivo na pasta. Não deixar backup (`*.bak`) dentro de `sites-enabled/` — o nginx carrega e duplica os `server` blocks ("conflicting server name"). Guardar backups FORA da pasta.

**Como adicionar um cliente novo (receita):** (1) registro A `<cliente>.ccskf.com.br -> IP` no DNS KingHost; (2) bloco `server` no `sites-enabled/portal-mafra` (server_name + proxy_pass pra porta do app); (3) `sudo nginx -t` (validar antes!); (4) `sudo systemctl reload nginx`; (5) `sudo certbot --nginx -d <cliente>.ccskf.com.br --non-interactive --agree-tos --redirect`.

**Acesso:** SSH `administrador@161.97.185.151` com `~/.ssh/davi_vps_teste`. **sudo pede senha** (a senha do Linux `Luma110703` — a MESMA marcada pra rotacionar, ver [[estado-portal-receb-junho2026]]). Ver também [[deploy-e-detector-versao-portal]] e [[arquitetura-dados-portal]].

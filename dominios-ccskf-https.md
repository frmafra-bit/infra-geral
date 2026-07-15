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

**⚠️⚠️ ARMADILHA CRÍTICA (14/07, quebrou Compras): o portal Davi precisa de DOIS proxies, não um.** O front do app fala com a **Service Layer via `/b1s/v1/` (URL relativa)** e com o backend Node via `/api/` + `/` (ambos no :3003). O `:3003` (container) serve o portal e `/api/`, mas **NÃO proxia `/b1s/` (dá 404)**. O proxy `/b1s/` → `https://b1.ativy.com:17162/b1s/` vivia no nginx **:8080** (config `davi-teste`). Ao apontar `davi.ccskf.com.br` só pro :3003, o Mosaico abria (usa `/api/mosaico/`) mas **TODO o SAP dava "Load failed"** (SL → 404). **FIX:** adicionar no bloco davi 443 um `location /b1s/ { proxy_pass https://b1.ativy.com:17162/b1s/; proxy_ssl_server_name on; proxy_set_header Host b1.ativy.com:17162; proxy_read_timeout 120s; }` + `client_max_body_size 50m;`. **Todo vhost novo do portal Davi precisa desses DOIS locations (`/` →:3003 e `/b1s/` →SL).** Testar vhost TLS específico com `curl --resolve host:443:127.0.0.1` (senão cai no vhost default e engana). Backup `/home/administrador/portal-mafra.bak-b1s`.

**⚠️ Gotcha do glob:** `include /etc/nginx/sites-enabled/*` pega QUALQUER arquivo na pasta. Não deixar backup (`*.bak`) dentro de `sites-enabled/` — o nginx carrega e duplica os `server` blocks ("conflicting server name"). Guardar backups FORA da pasta.

**Como adicionar um cliente novo (receita):** (1) registro A `<cliente>.ccskf.com.br -> IP` no DNS KingHost; (2) bloco `server` no `sites-enabled/portal-mafra` (server_name + proxy_pass pra porta do app); (3) `sudo nginx -t` (validar antes!); (4) `sudo systemctl reload nginx`; (5) `sudo certbot --nginx -d <cliente>.ccskf.com.br --non-interactive --agree-tos --redirect`.

**Acesso:** SSH `administrador@161.97.185.151` com `~/.ssh/davi_vps_teste`. **sudo pede senha** (a senha do Linux `Luma110703` — a MESMA marcada pra rotacionar, ver [[estado-portal-receb-junho2026]]). Ver também [[deploy-e-detector-versao-portal]] e [[arquitetura-dados-portal]].

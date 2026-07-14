---
name: container-161-camada-efemera
description: "PERIGO — o portal no VPS 161 roda em camada gravável do container; recriar o container apaga o trabalho"
metadata:
  node_type: memory
  type: project
---

**Descoberto em 09/07/2026, ao ligar as credenciais do Qive por `.env`.**

O container `davi_caminhoes` no VPS 161 tem **227 arquivos que existem SÓ na camada gravável**
(resultado dos deploys por `docker cp`). A **imagem `davi_caminhoes-davi_caminhoes` NÃO contém**
os apps novos — conferido: `Mosaico`, `Pendencias` e `NotasServico` não existem nela.

**Consequência:** `docker compose up -d`, `docker compose build`, `docker rm`, ou qualquer coisa que
**recrie** o container **APAGA todo o trabalho de 09/07 do servidor**. Só os bind-mounts sobrevivem
(`./xml` e `./data` — é por isso que o cache do Qive e o `tickets.db` estão a salvo).

**O que é seguro:** `docker restart davi_caminhoes` (preserva a camada gravável e faz o Node reler o
`/app/.env`). **O que NÃO é seguro:** recriar. Antes de qualquer recreate, ou se reconstruir a imagem,
**redeployar os 227 arquivos** — hoje a fonte deles é o git local do Mac (ver [[ambiente-mac-migracao]]).

**Credenciais do Qive (mudança 09/07):** saíram de literal em `backend/routes/mosaico.js` e passaram a
`process.env.QIVE_API_ID` / `QIVE_API_KEY`. Existem em **dois lugares** no 161:
`~/davi_caminhoes/.env` (host, lido pelo `env_file` do compose num futuro recreate) e `/app/.env`
(dentro do container, lido pelo `dotenv` do `server.js`, cujo WorkingDir é `/app`). `qiveGet` falha com
mensagem clara se faltarem. Validado: Qive responde HTTP 200 e `POST /api/mosaico/sync` = `success:true`.
Backup do estado anterior em `~/backup-thales/<timestamp>/` no 161.

**`deploy-161.sh` só serve para `webapp/`** (monta o destino com `rel=${f#webapp/}`). Arquivo de
**backend** precisa de `docker cp` manual + `docker restart`. Ver [[deploy-e-detector-versao-portal]].

**A cura definitiva** é a imagem passar a ser construída a partir do git (o repo próprio v3.0 —
ver [[equipe-e-governanca-portal]]), para o servidor deixar de ser a única cópia de um sistema
que não se consegue reconstruir.

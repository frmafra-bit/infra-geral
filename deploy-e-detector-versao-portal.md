---
name: deploy-e-detector-versao-portal
description: Como fazer deploy no VPS de teste e o detector de nova versão (anti-cache SPA) do portal Davi
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Deploy do portal no VPS de **teste (161)** — o time testa aqui, **nada vai pra produção (davicaminhoes.i16br.com / 213) sem ok do Mafra**. Ver [[equipe-e-governanca-portal]].

**Script de deploy:** `D:/portal-davi/deploy-161.sh <arquivo dentro de webapp/...> [mais...]` — faz `scp` + `docker cp` pro container `davi_caminhoes:/app/webapp/<relpath>` e **bumpa o `version.json`** (timestamp UTC). Chave: `~/.ssh/davi_vps_teste`; host `administrador@161.97.185.151`; **SSH na porta 22** (a 3003 é a porta HTTP do portal/Express — confirmado 04/07/2026, tentar SSH na 3003 dá "Connection closed"/HTTP 400). Clone local em `D:/portal-davi`.

**Detector de nova versão (implementado 01/07/2026):** o portal é uma SPA — o usuário carrega o JS na memória ao abrir e, sem recarregar a página, **continua rodando o código antigo mesmo após deploy** (foi a causa de vários "bugs fantasma": recurso já existia/corrigido, mas o navegador rodava versão de dias atrás). O servidor já manda `Cache-Control: no-cache, no-store` (server.js ~linha 105) — mas isso só afeta o PRÓXIMO fetch, e a SPA só refaz fetch num reload.
- `webapp/version.json` = `{"version":"<timestamp>"}`, servido com no-store.
- `Component.js` → `_initVersionChecker()`: lê a versão no boot, e a cada **90s** compara com o servidor; se mudou, `_promptReload()` abre MessageBox "Atualização disponível" → "Atualizar agora" faz `location.reload(true)`. "Depois" silencia até sair OUTRA versão.
- **REGRA:** todo deploy DEVE bumpar o `version.json` (o `deploy-161.sh` já faz). Sem bump, o aviso não dispara. Quando replicar em produção, levar Component.js + version.json + o bump no processo de deploy.
- **Novidades + COMO VALIDAR no aviso (01/07/2026):** o `version.json` tem `notes: [{m:"mudanca", v:"como validar"}, ...]`; o `_promptReload(notes)` exibe mudanca + "Como validar" no MessageBox (aceita string simples p/ retrocompat). Passar no deploy: `NOTES="mudanca 1::como validar 1|mudanca 2::como validar 2" bash deploy-161.sh <arquivos>` (`|` separa itens, `::` separa mudanca da validacao). O script monta o JSON localmente e envia. Caveat SPA: o formato novo só aparece a partir do deploy SEGUINTE ao que todos carregaram o Component com esse recurso.

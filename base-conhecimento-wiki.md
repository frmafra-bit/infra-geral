---
name: base-conhecimento-wiki
description: Wiki/base de conhecimento de problemas→soluções do portal (link + como manter)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

**Base de Conhecimento (wiki) do portal** — criada 02/07/2026 a pedido do CEO. Registro auditável de cada ocorrência: data/hora recebido→resolvido, solicitante, área, problema, diagnóstico, solução, status. Serve de referência p/ novos funcionários e auditoria (lições aprendidas por data).

- **Arquivo-fonte:** `webapp/wiki/index.html` (HTML autocontido; os registros ficam no array `DADOS` no `<script>`). Tem busca + filtro por área/solicitante.
- **Link (homolog):** http://161.97.185.151:3003/wiki/ (e :8080/wiki/).
- **Como atualizar:** editar o array `DADOS` (adicionar novo objeto {id,data,rec,res,solic,area,status,prob,diag,sol}) e deployar: `bash deploy-161.sh webapp/wiki/index.html`.
- **REGRA sugerida:** a cada entrega/correção nova, acrescentar um registro aqui (fecha o ciclo com o Detector de Versão, que já mostra "o que mudou + como validar"). Ver [[deploy-e-detector-versao-portal]], [[estado-portal-julho2026-compras-estoque]].

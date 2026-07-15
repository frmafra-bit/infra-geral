---
name: padrao-export-perso-listas
description: Padrão de Excel/PDF/escolher-colunas nas listas do portal (molde SaidaMercadoria) + gotcha do TablePersoController
metadata: 
  node_type: memory
  type: reference
  originSessionId: 92eba152-918b-4363-93e2-2ee7ca5ad176
---

**Item C.3 do backlog Compras** — padronizar Excel/PDF/escolher-colunas em TODAS as listas (molde = **SaidaMercadoria** `sMercadoria.controller.js`). Receita por lista:
- No controller: importar `sap/m/TablePersoController`; copiar as funções `onConfigColunas` (perso via localStorage, LS_KEY único por lista), `_mapaColunas` (ids+labels+property, `numero:true` p/ valores), `_colunasExport` (genérico, respeita colunas visíveis), `_dadosExport` (mapeia o modelo da tela), `onExportExcel` (CSV `;` + BOM `﻿`), `onExportPdf` (janela `window.open` + `print()`).
- Na view: `Table id="table"`; **id estático em TODAS as colunas** (ver gotcha); 3 botões na toolbar (`onConfigColunas`/`onExportExcel`/`onExportPdf`).

**⚠️ GOTCHA (achado testando a UI 14/07):** o `TablePersoController` exige **id estático em TODAS as colunas, inclusive as de botão/ação** (ex.: coluna "Copiar"). Se UMA coluna ficar sem id, o UI5 avisa *"Suffix __column0 ... must be static. Otherwise personalization can not be persisted"* e **o diálogo de Colunas NÃO abre**. Corrigido dando `id="colCopiar"` à coluna sem id.

**Piloto entregue e VALIDADO por UI (Playwright) 14/07:** `PedidoCompras/ConsultaPC` — login→menu Compras→Pedido de Compra→busca "Iniciar" (botão da FilterBar, não "Ir")→Excel baixa CSV, PDF abre impressão, diálogo Colunas abre com selecionar/reordenar. Botão de busca das listas = **"Iniciar"**. Após login aparece MessageBox "Usuário logado com sucesso" (dar OK).

**Estado das listas (14/07):** SaidaMercadoria=completa (molde). PedidoCompras=completa (piloto). Falta: SolicitacaoCompras (add colunas), NFEntrada (add Excel/PDF), RecebMercadoria/CotacaoCompras/EntradaMercadoria/TransferenciaEstoque/Devoluções/MediaCompras (tudo). Ver [[busca-cliente-8-copias]], [[estado-portal-julho2026-compras-estoque]].

**Testes de UI:** Playwright instalado no scratchpad (usa Chrome do Mac via `channel:'chrome'`); portal por IP `http://161.97.185.151:3003/` (evita DNS flaky do Mac). Ver [[fluxo-completo-compra-venda-teste]] p/ teste de sistema via SL.

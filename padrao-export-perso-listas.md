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

**Testes de UI:** Playwright instalado no scratchpad (usa Chrome do Mac via `channel:'chrome'`). ⚠️ **Telas que puxam da Service Layer (Compras/Vendas/quase tudo) NÃO funcionam por `http://161...:3003`** (o :3003 não proxia `/b1s/` → 404). Testar pela **`https://davi.ccskf.com.br`** lançando o Chrome com `args:['--host-resolver-rules=MAP davi.ccskf.com.br 161.97.185.151','--ignore-certificate-errors']` + `ignoreHTTPSErrors:true` → resolve o host direto pro IP (contorna DNS flaky do Mac E o :3003 sem /b1s/). Login: campos por placeholder Usuário/Senha, botão "Entrar", depois dispensar MessageBox "Usuário logado com sucesso" (OK). Botão de busca das listas = "Iniciar". **Drag-and-drop nativo do UI5** (HTML5 DnD) NÃO dispara com `dragTo`; usar sequência manual `mouse.move/down/move(steps)/up`. Ver [[fluxo-completo-compra-venda-teste]], [[dominios-ccskf-https]], [[mosaico-vendas-kanban]].

**Kanban de Faturamento (MosaicoVendas) — DRAG-AND-DROP entregue 15/07:** arrastar um card pra coluna vizinha à direita avança a etapa (Cotação→Pedido→Entrega→NF). Implementação: `dnd:DragInfo`/`dnd:DropInfo` (xmlns `sap.ui.core.dnd`) em `dragDropConfig` de cada `List`, **pareados por `groupName`** (G_cot/G_ped/G_ent) → o UI5 só deixa soltar na coluna certa (adjacência garantida sem código). O `drop="onDropCard"` reusa as ações existentes `onGerarPedido/Entrega/Nota` (passa o card arrastado como `getSource()`); mantém o diálogo total/parcial. Testado por UI: arrastar Cotação→Empenho abre "Gerar Empenho". Botões continuam funcionando.

---
name: mapa-relacoes-vendas
description: "Mapa de Relações estendido p/ cadeia de vendas + gotchas (NodeId, BaseType devolução)"
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Portal Davi — `Apps/MapaRelacoes/Controller/MapaRelacoes.controller.js` agora trata a cadeia de **vendas** além de compras (modo detectado por BaseType em `_onObjectMatched`; `_modoVenda`).

Faixas de venda: Cotação de Venda → Pedido de Venda → Entrega → NF de Saída → Contas a receber → Pedido de Devolução → Devolução → Nota de Crédito. Botão "Mapa de Relações" em todos os detalhes de venda.

ObjTypes/entitysets venda: Cotação 23=Quotations, Pedido 17=Orders, Entrega 15=DeliveryNotes, NF 13=Invoices, Contas a Receber 24=IncomingPayments, Pedido de Devolução 234000032=ReturnRequest, Devolução 16=Returns, Nota de Crédito 14=CreditNotes.

**Nota de Crédito (módulo novo `Apps/NotaCredito`):** modelo contábil do SAP p/ devolução — camadas espelhadas: Entrega↔Devolução (estoque), NF↔Nota de Crédito (financeiro+estoque). Devolução (Returns) só p/ mercadoria NÃO faturada; venda faturada estorna com Nota de Crédito (CreditNotes). Pedido de Devolução (ReturnRequest) não posta nada (autorização/RMA). O Contas a Receber vem da NF, não do Pedido de Devolução. Portal: botão "Gerar Nota de Crédito" na NFSaidaDetalhe (`onGerarNotaCredito` → POST CreditNotes com linhas BaseType 13/BaseEntry=NF, BPL 1, CFOP/TaxCode/WarehouseCode copiados da NF) + tela read-only NotaCreditoDetalhe (rota RouteNotaCreditoDetalhe). NC baseada em NF SÓ funciona se a NF não tiver Pedido de Devolução na linha (senão "base já fechada"). NC avulsa (sem base) sempre funciona.

**GOTCHA 1 (crítico):** cada tipo de documento tem numeração de DocEntry independente — Cotação 30, Pedido 30, Entrega 29 podem repetir o mesmo número. O controller identificava nó só por DocEntry → ao achar DocEntry repetido em outra faixa, descartava como duplicado e a cadeia parava. **Corrigido:** NodeId agora é composto `LineId + "_" + DocEntry`; deduplicação, ligações pai→filho (Children) e Origem usam o nid; nó guarda `DocEntry` à parte; `onNodePress` resolve o DocEntry real pelo nó (`oNodeSel.DocEntry`). `BuscaProximo`/`buscaTodosProximo`/`buscaContasPagar`/`buscaContasReceber` receberam `ParentNid`. Isso também consertou bug latente na cadeia de compras. **Why:** sem isso o mapa de vendas não mostra os próximos documentos.

**GOTCHA 2:** ao basear uma Devolução (Returns) num Pedido de Devolução, o BaseType na linha é **234000031** (não 234000032 — esse é o ObjType do documento em si). Mas nesta base o SL recusou Returns baseada em ReturnRequest ("cannot use this type of document as a base") e também recusou base em entrega já faturada ("base document has already been closed"). Na prática a Devolução nasce da **Entrega aberta** (BaseType 15). Vínculo RR→Return obrigatório seria parametrização SAP (Ativy).

Ver [[estorno-e-ver-lancamento-contabil]] (componente Ver Lançamento) e [[backlog-faturamento-vendas]].

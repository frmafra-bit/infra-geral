---
name: estado-portal-julho2026-compras-estoque
description: O que foi entregue/corrigido no portal (Compras e Estoque) em 01/07/2026 e o que ficou pendente
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Sessão intensa em **01/07/2026** — lote grande de correções de Compras e Estoque do portal, todas deployadas no **VPS teste 161** (ver [[deploy-e-detector-versao-portal]]). Nada em produção sem ok do Mafra. Ver [[projeto-migracao-sap-davi]], [[backlog-compras-portal]].

**COMPRAS — corrigido/entregue:**
- **DocTotal inflado ("1 item com valor total de todos"):** o SAP **honra** o DocTotal enviado (ver [[sap-sl-gotchas]]). Removido o envio de DocTotal fixo em IncluirPC, DetalhePC, CotacaoDetalhe, CotacaoAdd e no **`GravaPedidoCompra` do Mapa** (era a real origem: o botão "Criar Pedido" do Mapa mandava DocTotal + só os itens visíveis). Corrigidos ainda 2 typos no GravaPedidoCompra (`CostinCode2/3/4/5`→`CostingCode`, `DocumentsOwner -`→`=`).
- **Mapa de Cotações — itens sumiam:** o `$crossjoin(PurchaseQuotations,.../DocumentLines)` **faltava a junção** `PurchaseQuotations/DocEntry eq .../DocumentLines/DocEntry` → produto cartesiano → paginação(20) cortava itens. E usava `LineNum` no lugar do `BaseLine`. Corrigido (select passou a trazer BaseLine).
- **Mapa — despesas adicionais:** passou a mostrar TODAS as despesas (Seguro/Outros...), não só Frete; Valor Total do mapa somado pelos itens do mapa (não pelo DocTotal cheio da cotação).
- **Mapa — saldo da SC travava gerar pedido:** removida a trava por `saldoSC` (a SC zera ao ser cotada); vale o saldo da **cotação**.
- **Unidade de medida:** base vinha ZERADA (só "Manual"); populadas **54 unidades** nas 7 bases (ver [[catalogo-objetos-customizados-sap]]); campo passou a default "UN" ao escolher item.
- **Recebimento:** Total recalculado pelas linhas (não copiar DocTotal do pedido); botão **"Cancelar Recebimento"** (reusa `cancelarOuEstornarMovimento`).
- **Vincular Recebimento (ImportaXml):** coluna "Nota Fiscal" (NumAtCard); a NF passou a ser gerada **baseada no recebimento** (BaseType 20) → SAP fecha o recebimento e ele some da lista; **fallback**: se o XML divergir (qtd maior), gera NF sem vínculo + aviso.
- **PriceAccuracy (arredondamento) RESOLVIDO** — ver [[sap-sl-gotchas]].

**ESTOQUE — Saída de Mercadoria (lista sMercadoria):** colunas Usuário/Item/Projeto/CentroCusto/Frota/OS/Sub-contrato/Valor; **valor real** das baixas (custo médio via total do JournalEntry quando DocTotal=0 — `_injetaValorLinhas` aprimorado: mantém LineTotal exato e rateia só o residual); dimensões em **código E descrição** (colunas separadas); **escolher colunas** (sap.m.TablePersoController, salvo em localStorage); **exportar Excel (CSV ; WYSIWYG das colunas visíveis) e PDF**. "Ver Lançamento Contábil": valor da Movimentação de Estoque = custo médio.

**SOLICITAÇÃO:** Centro de Custo mostra **código + descrição** (propriedade `description` do Input; `CostingCodeName`).

**PENDENTE:** #4 Frete/IPI/Desconto sempre editável (cotação de solicitação) — não feito; padronizar Excel/PDF/escolher-colunas/cód+desc nas outras listas; subir tudo pra produção (com backup); Qive Fase 1. Ver [[integracao-qive]].

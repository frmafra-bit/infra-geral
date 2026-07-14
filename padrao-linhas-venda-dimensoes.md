---
name: padrao-linhas-venda-dimensoes
description: "Padrão das telas de faturamento: toggle Produto/Serviço + colunas de dimensão iguais às de compras"
metadata:
  type: project
---

**Padrão exigido pelo Mafra** para TODAS as telas de faturamento (Cotação/Licitação,
Pedido de Venda/Empenho, Entrega/Liberação, NF Saída): as linhas devem ser iguais às de
compras (documentos de marketing do SAP têm os mesmos campos em compra e venda).

**Toggle Produto x Serviço** (SegmentedButton no header da tabela, `selectedKey="{/<Doc>/_ModoVenda}"`,
`selectionChange="onModoVendaChange"`, valores `produto`/`servico`). Ao trocar, limpa as linhas.
- **Produto** → lupa Item (`ajudaPesqProdutos`, filtra `SalesItem eq 'Y'`), Quantidade, Preço, Depósito, Desconto %.
- **Serviço** → lupa **Conta Contábil de RECEITA** (`ChartOfAccounts` filtrado por
  `AccountType eq 'at_Revenues' and ActiveAccount eq 'tYES'`), valor (LineTotal) digitado direto.
  Em vendas é RECEITA; em compras é despesa. NUNCA trazer todas as contas.

**Colunas de dimensão** (mesmas de compras, gravam nos CostingCodes do SAP):
Projeto=`ProjectCode`, Centro de Custo/Sinistro=`CostingCode`, Frota/Pneu=`CostingCode2`,
OS=`CostingCode3`, Sub contratos=`CostingCode4`, + `CFOPCode`, `TaxCode`, `DiscountPercent`.

**Armadilha recorrente — array do modelo não inicializado:** vários lookups do Base fazem `push` num
array do modelo (`Projetos`, `CentroCusto`, `Frota`, `OrdemServico`, `SubContratos`, `CondPag`, `Produtos`,
`ContaContabil`, `TpImpostoCompras`) e ASSUMEM que ele existe → em telas que não inicializam dá
`undefined is not an object (...push)`. Correção preferida: o próprio `busca*` do Base inicializa o array
antes de preencher (feito em `BuscaCondPag` v2026-07-13T17:16). Caso contrário, inicializar no `criaModelo`
da tela. Ao criar/tocar num lookup novo, garantir o array.

**Imposto (TaxCode) entrada × saída:** venda = SAÍDA = **código começa com 5** (5101/5102… CFOP de
venda); compra = entrada = começa com 1. Filtro CORRETO em telas de venda: LOCAL
`startswith(Code,'5') and Inactive eq 'tNO'` no `SalesTaxCodes`. ⚠️ `ValidForAR eq 'tYES'` NÃO serve —
inclui 1101-001 (testado no SAP: 1101-001 tem ValidForAR=tYES). `SalesTaxCodes` NÃO tem campo `Category`.
Confirmado no SAP homolog: 21 códigos 5xxx. O `onFiltraTpImpostoCompras` do
Base é de COMPRAS (traz tudo). Mesma lógica das dimensões: os lookups do Base
que fazem `push` ao abrir precisam do array inicializado no modelo (`Projetos`, `CentroCusto`, `Frota`,
`OrdemServico`, `SubContratos`) senão dá "Erro - buscaX".

**Writeback:** os lookups do Base gravam em `data.Items[linha]` (path fixo `/Items`, split[2]).
NÃO serve para telas cujo modelo é `/<Doc>/DocumentLines` (split[3]). Solução (aplicada na Cotação):
reusar `ajudaPesq*`/`onFiltra*`/`busca*` do Base (abrem fragmento + preenchem lista, genéricos) e
**sobrescrever só os `onConfirmaDialog*` localmente** gravando em `<Doc>.DocumentLines[linha]`.
Imposto e Conta Contábil precisam ser 100% locais (o Base grava hardcoded em `NFEntrada`).

**Salvar:** montar payload limpo — o SAP rejeita o campo interno `_ModoVenda`. Setar
`DocType: servico ? 'dDocument_Service' : 'dDocument_Items'`. Linha serviço = `AccountCode`+`LineTotal`
(sem ItemCode/Quantity). Linha produto = `ItemCode`+`Quantity`+`UnitPrice`(=Price da tela)+`WarehouseCode`.

**Feito:** Cotação/Licitação (`CotacaoVendaAdicionar`) e **Pedido de Venda 100%** (`PedidoVendaAdicionar`
view+controller reescritos v2026-07-14T02:54Z): tela ÚNICA criar/ver/editar (rota
`PedidoVendaAdicionar/:DocEntry:/:mode:`), toggle Produto/Serviço, conta de receita, imposto 5xxx,
dimensões, Total por linha, células EDITÁVEIS (a view era cópia read-only da consulta = bug),
modo edição carrega `Orders(DocEntry)` em `/Pedido`. Card do Kanban (`onAbrirPedido`→`_abrirEdicao`),
lista (`onSelectionChange`) e "Novo" abrem esta tela. Aposentada a `PedidoVendaDetalhe` (tela de
consulta que não abria em runtime). LIÇÃO: a `PedidoVendaAdicionar.view` (30/jun) tinha a tabela
ligada em `/PedidoVenda/DocumentLines` mas o modelo é `/Pedido` → "Adicionar Itens não fazia nada".
Testado via SL: Pedido produto nº 66 e serviço nº 67 criados OK (homolog, podem ser removidos).
**Falta replicar:** Entrega (Liberação). NFSaida já tinha o toggle (falta conta de receita + 5xxx).
Ver [[busca-cliente-8-copias]], [[ciclo-vendas-setor-publico]], [[mosaico-vendas-kanban]].

**Fechar documento (Close) com justificativa (Pedido, deploy v2026-07-14T03:39):** botão "Fechar Pedido"
(visível só em edição + `DocumentStatus === 'bost_Open'`). Fluxo: pede justificativa OBRIGATÓRIA →
PATCH `Orders(DE)` com `Comments` = antigo + `\n[FECHAMENTO dd/mm/aaaa]: <motivo>` (grava ANTES, doc
fechado fica read-only) → POST `Orders(DE)/Close`. Testado no SL: PATCH 204 + Close 204 → bost_Close.
Padrão a repetir na Cotação (`Quotations/Close`) e Entrega. **Descrições das dimensões:** nome embaixo
do código na linha (`_ProjectName`/`_AccountName`/`_TaxName`/`_CostingName`/`_Costing2/3/4Name`, classe
CSS `.dimDesc`); resolvidos no load via `Projects`(Code/Name), `ChartOfAccounts`(Code/Name),
`SalesTaxCodes`(Code/Name), `DistributionRules`(FactorCode/FactorDescription). **Rolagem horizontal:**
`onAfterRendering` liga `wheel`→`scrollLeft` na `.tabelaItensScroll` (Windows/mouse comum) + CSS
`::-webkit-scrollbar` visível (Mac esconde overlay). Ver [[mapa-relacoes-vendas]].

**CICLO DE VENDAS COMPLETO (v2026-07-14T04:07):** Cotação, Pedido, Entrega e NF de Saída todos
padronizados (toggle Produto/Serviço, conta de receita at_Revenues, imposto saída 5xxx, dimensões com
nome, Mapa). Cada LISTA tem toggle Lista/Kanban (colunas Aberto/Fechado, saldo aberto, botão Mapa),
ordenada por DocNum desc, colunas Número/Cliente/Status/Valor, clique abre a tela completa em edição
(Cotação/Pedido/Entrega via Adicionar-edit; NF via Detalhe). BaseType do Mapa: Cotação 23, Pedido 17,
Entrega 15, NF 13. Entregas de PRODUTO baixam estoque (SAP exige saldo); serviço não. NFSaida usa
`table:Table` (grade, rola nativa) — sem ScrollContainer. Bug: `f:content` do DynamicPage aceita 1
filho só (Entrega lista quebrou até envolver em VBox); Cotação/Pedido usam VerticalLayout (aceita vários).

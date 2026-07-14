---
name: mosaico-vendas-kanban
description: "Kanban de Faturamento (ciclo de vendas) — app novo MosaicoVendas, espelho do Mosaico de compras; cadeia validada via SL"
metadata:
  type: project
---

**Construído 12/07/2026** — app novo `Apps/MosaicoVendas`, kanban do ciclo de VENDAS na língua de
contrato público ([[ciclo-vendas-setor-publico]]). Espelho do [[mosaico-kanban-fiscal|Mosaico de compras]].

## Fundação VALIDADA via Service Layer (SBO_HOMOLOG, 12/07) — ANTES de montar a UI

Cadeia de vendas tipo **Serviço** encadeia ponta a ponta (payloads reais):
- Cotação (Quotations) → Pedido base 23 → Entrega base 17 → NF de Saída base 15. Todos
  `DocType:"dDocument_Service"`, linha com `AccountCode` + `LineTotal` (SEM ItemCode/quantidade — a
  NFS-e de serviço não tem quantidade, ver [[faturamento-davi-2026-analise]]).
- **Gotchas confirmados:** exige `BPL_IDAssignedToInvoice:1` (senão "Specify an active branch [OQUT.BPLId]");
  a **NF de Saída exige `TaxCode`** na linha (senão "Row without tax") — usei `1101-001` (SalesTaxCodes).
- Cliente de teste: `C43465459000173` MUNICIPIO DE AMPARO (prefeitura já cadastrada no homolog).
  Conta receita: `3.01.01.01.02`. Docs de teste ficaram no homolog (Cotação 43→Pedido 88→Entrega 82→NF 56).

## Estrutura do app (deployado 161, v2026-07-13T00:13Z)

- `MosaicoVendas.view.xml` — 4 colunas: **Licitação/Cotação → Empenho/Pedido → Liberação/Entrega →
  Faturamento/NF de Saída** + alternância **Kanban ↔ Lista** (SegmentedButton /Visao) + "Nova
  Licitação/Cotação" (por ora navTo Cotação de Venda existente).
- `MosaicoVendas.controller.js` — herda Base; `_carregar()` puxa Quotations/Orders/DeliveryNotes
  abertos + Invoices via fetchApi; botões `onGerarPedido/onGerarEntrega/onGerarNota` usam `_gerar()`
  com os BaseType validados.
- Manifest: rota `RouteMosaicoVendas` + target `TargetMosaicoVendas`. Handler `MosaicoVendas()` no Base.
  Menu Faturamento > "🔴 NOVO — Faturamento (Kanban)".
- Namespace correto do projeto: **`portalsap.portalsap`** (NÃO sap.ui.demo.walkthrough).

## DEVOLUÇÃO, NÃO CANCELAMENTO (regra do Mafra 12/07 — espelho de compras)

No SAP não se cancela documento contabilizado; reverte-se pelo **documento inverso** (ver
[[mosaico-kanban-fiscal]], lição da NF 78/09-07). No kanban de vendas:
- **Entrega errada → Devolução de Venda** (`Returns`, base 15) — botão "Devolver" na col. Liberação.
- **NF de Saída errada → Nota de Crédito** (`CreditNotes`, base 13) — botão "Devolver (Nota de Crédito)".
- `_devolver()` herda linhas com Quantity/AccountCode/TaxCode e lança o inverso. NUNCA usa `/Cancel`.
- (Autocrítica: na limpeza dos testes eu chamei `/Cancel` e deu 400 nos 4 — era pra ter devolvido.)

## ENTREGUE 12/07 (3 pedidos do Mafra, deploy v2026-07-13T00:32Z)

1. **Card na Home** — aba Faturamento, tile "Faturamento (Kanban)" (press MosaicoVendas), 1º da aba.
   Espelha o padrão do tile_mosaico das compras.
2. **Clicar no doc abre o formulário** — cada card tem o nº como `<Link>`: onAbrirCotacao/Pedido/
   Entrega/Nota → navTo Route{Cotacao|Pedido}VendaDetalhe / RouteEntregaDetalhe / RouteNFSaidaDetalhe
   com {DocEntry}. As telas de detalhe de venda JÁ existiam.
3. **Criar em qualquer fase** — botão + no topo de cada coluna: onNovaCotacao/NovoPedido/NovaEntrega/
   NovaNota → navTo Route...Adicionar (Entrega usa fallback RouteEntrega).
4. **Mapa de Relações em TODAS as etapas** — botão árvore em cada card + na visão lista (onMapaLinha);
   navTo RouteMapaRelacoes/{DocEntry}/{BaseType} (23/17/15/13). Componente MapaRelacoes reaproveitado.

## ENTREGUE 12/07 parte 2 (saldo + filtro, deploy v2026-07-13T01:08Z)

- **Saldo aberto por card:** `_comSaldo()` soma `OpenAmount` das DocumentLines (R$ ainda não
  consumido); para Invoices usa DocTotal − PaidToDate. Card mostra Total + "Saldo aberto R$" (verde se
  >0) + situação (Aberto/Fechado/Cancelada-Devolvida via DocumentStatus/Cancelled).
- **Filtro de situação:** SegmentedButton /Situacao (aberto/fechado/todos) no topo; `onMudaSituacao`
  refiltra por `DocumentStatus eq bost_Open/bost_Close`. Default "aberto". Quando o saldo zera o doc
  some do "aberto" e aparece em "fechados".

## TELAS DE FATURAMENTO — falta padronizar (pedido Mafra 12/07, NÃO feito ainda)

As 4 telas de venda (Cotação, Pedido, Entrega, NF de Saída) estão MUITO atrás da compra. Comparação:
- Alvo (NF Entrada): Item, Descrição, Referência, Qtd, Preço, Projeto, Centro de Custo, Frota/Pneu, OS,
  Sub contratos, Desconto %/R$, CFOP, Código Imposto.
- CotacaoVenda: só Item/Desc/Qtd/Preço. PedidoVenda: só Item/Desc/Qtd. Entrega: Desc/Qtd/Preço.
  NFSaida: mais completa (tem dimensões + Sub, falta Referência/desconto/CFOP/imposto).

Pedidos do Mafra para TODAS as telas de faturamento:
1. Colunas de item completas (referência, dimensões, desconto, CFOP, imposto) — padronizar pela NFSaida
   (NÃO pela compra: dimensões de venda usam **DistributionRules**, não ProfitCenters — ver
   [[backlog-faturamento-vendas]]).
2. Cabeçalho busca **CLIENTE** (cCustomer), não fornecedor. NFSaida já faz (l.1105); garantir nas outras.
3. Linha com seletor **Item OU Serviço**: Item → busca produto (Items SalesItem eq 'Y'); Serviço →
   busca **conta contábil de RECEITA** (`ChartOfAccounts AccountType eq 'at_Revenues'`). ⚠️ hoje a busca
   de conta (`_carregaNomesContas`/AjudaPesqContaContabil) traz TODAS as contas — falta filtrar por receita.
4. Alinhar o botão Produto/Serviço ao **DocType** do SAP (hoje NFSaida usa gambiarra `_ModoVenda` =
   produto/servico, desligada do DocType — unificar).
Plano: fazer a **NFSaida** como molde validado primeiro, depois replicar em Cotação/Pedido/Entrega.

## Pendências (próximas etapas)

1. **Valor PARCIAL no avanço** — hoje `_gerar()` herda a linha inteira; falta permitir editar o valor
   (total OU parcial) antes de gerar, como o Mafra descreveu (empenho parcial → medição parcial).
2. **"Nova Cotação" completa** dentro do kanban (hoje só encaminha p/ tela existente).
3. **Modo Item** (produto/ativo/remessa) — o kanban hoje assume Serviço; suportar também Item
   (venda de produto, ativo imobilizado, remessa p/ conserto) — depende do CFOP da Rafaela
   ([[backlog-fiscal-rafaela]]).
4. **Selo "Devolvida"** no card após reverter (o de compras tem; falta persistir estado — vendas não
   tem cache tipo Qive, o estado teria que vir do próprio SAP ou de estado local).
5. **Validar na tela ao vivo** (a base homolog tem pouca venda; colunas podem vir vazias).

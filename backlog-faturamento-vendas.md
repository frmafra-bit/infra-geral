---
name: backlog-faturamento-vendas
description: "Auditoria do ciclo de vendas/faturamento (29/06) — lacunas vs padrão das compras; só diagnosticado, não implementado"
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Auditoria do **faturamento (ciclo de vendas)** feita em 29/06/2026 a pedido do Mafra, antes de evoluir.
Decisão dele: **só auditar agora, NÃO implementar** (avaliar depois). Doc completo: `D:\Davi caminhoes\Auditoria_Faturamento_Davi.md`.

Ciclo: Cotação de Venda → Pedido de Venda → Entrega → NF de Saída. Está **bem atrás** do padrão que aplicamos
em Compras/[[estado-portal-receb-junho2026]]. Lacunas vs regras firmadas ([[padrao-formulario-unico-processo]]):
1. **Dimensões incompletas:** NF de Saída expõe só Projeto + Centro de Custo (1); faltam CostingCode2..5. Os
   controllers de Pedido/NF têm handlers `onChangeCostingCode2..5` mas SEM colunas na tela (código morto).
   Cotação e Pedido de Venda não têm dimensão nenhuma.
2. **Desconto por linha:** ausente em Pedido e NF de Saída; na Cotação só `%` (consulta, read-only), falta o R$.
3. **Total da linha** não aparece na linha (só total do cabeçalho) — mesma correção do Recebimento.
4. **Sem opção Item x Serviço na NF de Saída** → não emite nota de serviço (mão de obra, locação, frete);
   é o espelho do "lançamento sem produto" que resolvemos no Contas a Receber.
5. **Padronização inclusão↔consulta** quebrada (Detalhe omite campos do Adicionar).

Integração já existe: NF de Saída (Invoices) é origem do Contas a Receber; Devolução de Venda (CreditNotes)
alimenta o abatimento (NC sinal −1). Mesma mecânica de colunas + value-helps `ajudaPesqProjetos`/
`ajudaPesqCentroCusto` que já usamos nas compras. Plano sugerido: NF de Saída primeiro, depois Pedido, Cotação, Entrega.

**INÍCIO do rework Faturamento (30/06):** NF de Saída-Adicionar tinha base quebrada — `changeTotal` e handlers de dimensão (onConfirmaDialogFrota/OS/Etapa/Gestor) referenciam `/Cotacao` que NÃO existe no criaModelo (só `/NFSaida`) → erro ao mudar qtd/preço, total nunca atualizava. **v1 deployada:** changeTotal reescrito (modelo /NFSaida, acumula, desconto-aware, seta /NFSaida/DocTotal) + parseNumericValue + onItemDiscountChange/onItemDiscountValorChange (Desconto %↔R$ por linha) + coluna Total de linha. Save remove DiscountValue/LineTotal das linhas (SAP recalcula). FALTA validar na tela (sem dados de venda na base). **v2 pendente:** Item/Serviço (precisa coluna AccountCode/conta razão, espelhar NF Entrada) + dimensões CC2-4 (value-helps ajudaPesqFrota=ProfitCenters, ajudaPesqOS=DistributionRules, ajudaPesqSubContratos — esta NÃO existe nem na NF Entrada, criar) escrevendo em /NFSaida.DocumentLines[linha].CostingCode2/3/4. Depois NFSaidaDetalhe + PedidoVenda + CotacaoVenda. Template = NFEntradaAdicionar (controller ~3000 linhas, métodos em 1165-1310 dimensões, 1352 changeTotal, 1512 desconto, 212 onDocTypeChange).

**NF de Saída Adicionar — v2 deployada (30/06):** dimensões CC2-4 (ajudaPesqFrota/OS/SubContratos) — DESCOBERTA: dimensões usam **DistributionRules** (FactorCode) por InWhichDimension, NÃO ProfitCenters (a NF Entrada usa ProfitCenters pra Frota = ERRADO, gera "Invalid distribution rule"). buscaFrota=DistributionRules dim2, buscaOS=dim3, buscaSubContratos=dim4; onConfirma escreve /NFSaida.DocumentLines[linha].CostingCode2/3/4. Item/Serviço: SegmentedButton selectedKey={/NFSaida/DocType} + onDocTypeChange (limpa linhas) + coluna Conta Contábil (ajudaPesqContaContabil, fragment ContaContabil existe) visível p/ Serviço, ItemCode/Depósito visíveis p/ Item. criaModelo DocType="dDocument_Items". FALTA validar na tela (sem dados de venda). PENDENTE: NFSaidaDetalhe (consulta read-only com mesmos campos), Pedido de Venda, Cotação de Venda, Entrega. Quando fizer Pedido/Cotação Venda, CORRIGIR também a fonte das dimensões para DistributionRules se estiverem com ProfitCenters.

---
name: ciclo-vendas-setor-publico
description: "Modelo do ciclo de vendas da Davi/Terra Mix — venda para poder público; cada objeto SAP tem nome/conceito de contrato público"
metadata:
  type: project
---

**Correção do Mafra (12/07/2026):** o ciclo de vendas do portal está com rótulos genéricos, mas a
Davi/Terra Mix **fatura contra PREFEITURAS** (ver [[integracao-qive]] — faturamento = NFS-e emitidas
contra Mogi/Suzano/Ribeirão Pires). Então cada objeto padrão do SAP tem um NOME e um CONCEITO de
contrato público. O motor (documento SAP) é o mesmo; muda o significado.

## O mapeamento (objeto SAP → conceito público)

| Objeto SAP | Rótulo hoje | Conceito correto |
|---|---|---|
| SalesQuotations (23) | Cotação de Venda | **Licitação / Pregão GANHA** (o ganho da venda) |
| Orders (17) | Pedido de Venda | **EMPENHO** — previsão de faturamento (total OU parcial), depende de aprovação |
| DeliveryNotes (15) | Entregas | **Liberação / Autorização p/ faturar** — baixa o empenho (total/parcial) |
| Invoices (13) | Notas Fiscais de Saída | **FATURAMENTO** (serviço ou produto), baseado na Entrega (total/parcial) |
| CreditNotes / Returns | Devolução | **Devolução de Entrega** + **Devolução de NF de Saída** |

## O fluxo (nas palavras do Mafra)

1. **Cotação** = o ganho da licitação/venda.
2. **Empenho** (= Pedido de Venda) = *previsão de faturamento*; foi liberado para faturar MAS depende de
   ser aprovado. Pode ser total ou parcial.
3. Quando **autorizado** → **Entrega**, que **baixa o Pedido de Venda (empenho)** total ou parcial.
4. **Nota** = faturamento, feito **com base na Entrega**, total ou parcial.

Cadeia: Licitação ganha → Empenho (previsão) → Entrega (baixa o empenho) → Nota (baseada na entrega),
com quantidade parcial em cada passo.

## Chave técnica

É o **ESPELHO do ciclo de compras** que já validamos ponta-a-ponta no Mosaico
(ver [[mosaico-kanban-fiscal]] e [[fluxo-completo-compra-venda-teste]]): Pedido→Recebimento→NF de Entrada,
com consumo parcial por `BaseType/BaseEntry/BaseLine` e o pedido fechando sozinho quando 100% consumido.
No lado de vendas: **Pedido de Venda (17) → Entrega base 17 → NF de Saída base Entrega (15)**.

**Estado atual (verificado 12/07):** ver [[backlog-faturamento-vendas]] — o ciclo de vendas foi só
AUDITADO, exceto o rework da NFSaida-Adicionar (30/06). O encadeamento forward com baixa parcial
(empenho→entrega→nota) **provavelmente NÃO está implementado** — os BaseType encontrados nos controllers
parecem auto-referência/reversão, não a criação encadeada. **CONFIRMAR na tela antes de estimar.**
App `Licitacoes` já EXISTE no portal (começado; era POC UDO @DAVI_LICIT — ver [[proximos-portais-davi]]).

## ESCOPO DECIDIDO (Mafra 12/07/2026) — a "saída sem sofrer"

**Insight do Mafra:** no SAP todo documento de marketing (saída OU compra) pode ser **por Item** ou
**por Serviço**. No modo **Serviço** não há ItemCode/quantidade — seleciona-se uma **conta contábil**
(`AccountCode`) e lança-se o **valor**. Isso ESPELHA exatamente a NFS-e emitida (valor fechado, sem
quantidade — ver [[faturamento-davi-2026-analise]]) e é como já se lança serviço em compras (a NF de
Entrada 69 / AO3 era `dDocument_Service`). **Já existe no código:** `NFSaidaAdicionar.controller.js`
tem `ajudaPesqContaContabil` (l.444), grava `AccountCode` (l.481), trata `dDocument_Service` (l.915) —
espelho de compras começado; falta a coluna na view e validar.

**Consequência:** dissolve o problema de "capturar quantidade". A **NF de Saída de serviço** = conta +
valor. O **saldo passa a ser por VALOR** (não por quantidade): contrato R$ teto → empenho R$ autorizado
→ nota R$ abate. O SAP controla saldo por valor nativo na cadeia de documentos de serviço.

**PRIORIDADE AGORA:** NF de Saída **por prestação de serviços** (faturar prefeitura). Fazer o modo
Serviço ficar sólido.

**NÃO DESCARTAR (o portal de vendas tem que suportar TAMBÉM o modo Item):** Pedido de Venda + NF de
Saída para **venda de produtos**, **ativo imobilizado**, **remessa para conserto** e **outras remessas**.
São operações tipo Item, com CFOP próprio (remessa não é venda — sai e volta). ⚠️ Remessas dependem do
**CFOP correto** = item pendente da Rafaela (ver [[backlog-fiscal-rafaela]]).

**DEPOIS (adiado):** medições / faturamento por medição (a quantidade real da planilha anexa). Só pensar
nisso se/quando precisarem do detalhe de quantidade dentro do sistema.

## O que falta decidir (Mafra)

- **Empenho**: guarda só o número do empenho, ou dados que puxam para a nota (dotação, valor autorizado)?
- **Relabel simples** (Cotação→Licitação, Entrega→Liberação) vs **modelagem** (empenho + licitação como
  estrutura própria). Duas são só nome; empenho e licitação exigem campo/UDO.
- Liga com [[proximos-portais-davi]] (Contratos & Projetos, Vanessa) e com a captura do faturamento
  (NFS-e emitida, [[integracao-qive]]) → Contas a Receber automático.

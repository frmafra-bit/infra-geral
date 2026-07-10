---
name: estorno-e-ver-lancamento-contabil
description: "Mecânica validada (29/06) p/ botão Ver Lançamento Contábil e Cancelar/Estornar movimentos — pedido do Mafra, NÃO implementado ainda"
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

**Melhoria 09/07** (`view/fragmentos/LancamentoContabil.fragment.xml` + `verLancamentoContabil` no Base): rodapé de TOTAIS nas colunas Débito/Crédito (`<Column><footer><ObjectNumber .../>` com `dados.TotalDebito/TotalCredito`, verde se `Balanceado` senão vermelho). Coluna "Parceiro de Negócio" só nas linhas de PARCEIRO — regra `ShortName !== AccountCode` (aí ShortName é o CardCode; conta contábil pura tem ShortName==AccountCode) — e com NOME do parceiro (1 busca `BusinessPartners?$select=CardCode,CardName&$filter=` pelos CardCodes distintos). Vale em TODAS as telas que usam o componente. Confirmado no JE 580 (2 linhas Fornecedores 2.01.01.01.01/CardCode + 1 transitória 1.01.02.01.99).

Pedido do Mafra (29/06): todas as telas que geram movimento contábil deveriam ter um botão **"Ver Lançamento
Contábil"** (igual ao Detalhes do [[estado-portal-receb-junho2026|Contas a Pagar]]), e faltam **Cancelar/Estornar**
(reverter contábil + custo + estoque) na Entrada de Mercadoria e na Requisição. Mecânica VALIDADA via Service Layer,
mas **NÃO implementado** — o Mafra dispensou as perguntas de escopo (aguardar próxima instrução).

**Linkagem documento → lançamento contábil (universal):** todo documento que posta contábil tem o campo
**`TransNum`** = o `JdtNum` do lançamento. Basta `GET JournalEntries(TransNum)` p/ montar o dialog de
contabilização em QUALQUER tela. Ex.: Goods Receipt DocEntry 80 → TransNum 189 = JE "Goods Receipt". Para
movimentos de estoque, mostrar tb a movimentação (item/depósito/qtd/entra-sai) já vem nas DocumentLines.

**Cancelar/Estornar (particularidade do SL):**
- **Entrada/Saída de Mercadoria** (`InventoryGenEntries`/`InventoryGenExits`): SL **NÃO expõe** `/Cancel` nem
  `/Close` (erro "Invalid entityset 'InventoryGenEntry'"). Estorno SAP-correto = **documento inverso**
  (Saída estorna Entrada e vice-versa, mesmo item/qtd/depósito) — reverte estoque + contábil. VALIDADO
  (GI DocEntry 96 estornou o GR 80). Campo de filial nesses docs = `BPL_IDAssignedToInvoice: 1` (não BPLID).
- **Documentos de marketing** (Quotations/Orders/Invoices...): `/Cancel` funciona (portal já usa
  `Quotations(DocEntry)/Cancel` em CotacaoVendaDetalhe e `PurchaseQuotations/Close` em RelMapaCotacoes).
- **StockTransfers**: `/Cancel` provavelmente suportado (deu 404 "no records" em DocEntry inexistente, não
  "invalid entityset") — confirmar com doc real.
- **Requisição** (endpoint plural `InventoryTransferRequests`, tipo interno 'StockTransfer'): cancelar = **`/Close`**
  → ✅ **FUNCIONA** (testei 30/06: criei DocEntry 18, `InventoryTransferRequests(18)/Close` → 204, virou bost_Close).
  `/Cancel` dá "action not supported" (requisição não tem cancelamento, tem fechamento). ATENÇÃO: é PLURAL com 's'
  (o singular `InventoryTransferRequest` dá "invalid entityset" — foi meu erro inicial). Não tem campo `Cancelled`,
  o status é DocumentStatus bost_Open/bost_Close. Provavelmente foi ISSO que o Tiago (frotas) fez.

Plano sugerido quando retomar: componente reutilizável "Ver Lançamento Contábil" (dialog via TransNum) +
ação Cancelar/Estornar por tipo (documento inverso p/ estoque; /Cancel p/ marketing; fechar p/ requisição).

**IMPLEMENTADO e deployado no VPS teste (30/06)** — conceito do Mafra ("mesmo endpoint + /Cancel"), com fallback:
- **Componente reutilizável no Base.controller.js** (disponível p/ TODAS as telas):
  `verLancamentoContabil(transNum, titulo, movimento)` abre fragment `view/fragmentos/LancamentoContabil.fragment.xml`
  (lançamento via `JournalEntries(TransNum)` + movimentação de estoque). E `cancelarOuEstornarMovimento(cfg)`:
  tenta a ação nativa `(entitySet)(docEntry)/Cancel|Close` (o conceito); se o SL não concluir, faz
  `_estornarPorDocumentoInverso` (POST no reverseEntitySet com as DocumentLines) revertendo estoque+contábil.
- **Entrada de Mercadoria** (eMercadoriaview): footer com "Ver Lançamento Contábil" (onVerLancamentoEntrada) e
  "Cancelar/Estornar" (onCancelarEntrada → InventoryGenEntries/Cancel, fallback InventoryGenExits).
- **Requisição** (DetalheSolicTransfEstoque): botão "Cancelar Requisição" (onCancelarRequisicao →
  InventoryTransferRequests(de)/Close, que JÁ funciona nativo).
- Pendente rollout: Saída de Mercadoria, Transferência, NFs, e o botão "Ver Lançamento" nas demais telas contábeis.
  Reaproveitar os métodos do Base (já prontos). node --check + XML OK; servido no nginx 8080.

---
name: fluxo-completo-compra-venda-teste
description: Receita p/ gerar fluxo completo compra→estoque→venda via SL + dados-mestre válidos (homolog)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Receita validada (SBO_HOMOLOG) para gerar ciclo completo make-to-stock via Service Layer, com projeto+dimensões+comprador. 20 fluxos criados em jun/2026 (DocNums NFv 13-32). Cada fluxo:

**Compra:** PurchaseRequests → 3× PurchaseQuotations (base 1470000113 na SC) → PurchaseOrders (base 540000006) → PurchaseDeliveryNotes/recebimento (base 22) → PurchaseInvoices/NF entrada (base 20) [alimenta estoque]. **Venda:** Orders/pedido → DeliveryNotes/remessa (base 17) → Invoices/NF venda (base 15) [gera financeiro].

Dados-mestre válidos:
- **Produtos estoque+compra+venda** (saldo 0): RV-0001..RV-0010 (areia/pedra/brita). Depósito "01".
- **Clientes**: C0001 (GABRIELA), C43465459000173 (AMPARO), C46319000000150 (GUARULHOS), C46523247000193 (DIADEMA), C46522959000198 (MAUA).
- **Fornecedores**: F00846804000106 (BRIDA), F47866934000174 (TICKET), F04323682000124 (CASA ANTENAS), F54890805000187, F03864289000185, F20938089000149.
- **Projetos** (OPRJ Code): 164/2021, 45/2021, 69/2022, 92/2022, 117/2023, 278/2022, 002/2022, 10/2024.
- **Dimensões** (linha): CostingCode=dim1, CostingCode2=dim2, CostingCode3=dim3, CostingCode4=dim4. Válidos testados: dim1 `01`/`CC0026`/`CC0025`; dim2 `DC1213`/`ALT9985`/`BDQ0I66` (NÃO usar `AKZ2F30` — "Invalid distribution rule", é regra de rateio múltiplo); dim3 `01012024`/`16`/`21`; **dim4 (Sub-Contrato) `10.001` (EEPS-CHICO) / `00001` (Sub-contrato 1)**. Transferências não aceitam dims.

**DRE por projeto/dimensão (amarrado ao razão):** projeto+dims são carimbados nas linhas do JournalEntry de cada documento. Conta **Receita `3.01.01.01.08`** (crédito, na NF de Venda/Invoices) e **CMV `5.02.01.01.01`** (débito, na Entrega/DeliveryNotes) — ambas carregam ProjectCode+CostingCode1-4. Estoque cred. `1.01.02.01.01`, AR `1.01.03.01.01`. Deduções/impostos ainda NÃO postam separado (parametrização fiscal pendente c/ Ativy) → DocTotal da NF = receita. DRE = Receita − Deduções − CMV = Resultado; agrega por qualquer eixo somando as linhas do JE. **Atenção custo médio:** CMV pode divergir do preço da última compra se o item tinha saldo anterior a outro custo (MovingAverage) — o razão é a verdade.
- **Comprador/solicitante**: campo `RequesterName` na PurchaseRequests (ex.: "RUTE SILVA", "LILIANE BARBOSA", "CINTIA TELES" — EmployeesInfo 2/5/4). ReqType válidos: 12=User,171=Employee (labels trocados no SL); mais simples só `RequesterName`.
- **CFOP/imposto**: compra `CFOPCode 0001`/`TaxCode 0001-001`; venda `5110`/`5110-001`. Filial sempre `BPL_IDAssignedToInvoice:1`.
- Docs de compra exigem header `RequriedDate` (grafia do SL) + linha `RequiredDate`, senão erro "Specify the required date [OINV.ReqDate]".

**Baixas/financeiro (fechamento do ciclo):** conta banco `1.01.01.02.02` (TransferAccount). Pagamento fornecedor = `VendorPayments` {DocType:"rSupplier", CardCode, BPLID:1, TransferAccount, TransferSum, PaymentInvoices:[{DocEntry, SumApplied, InvoiceType:"it_PurchaseInvoice"}]}. Recebimento cliente = `IncomingPayments` {DocType:"rCustomer", ..., InvoiceType:"it_Invoice"}. 20 NFs entrada pagas + 20 NFs venda recebidas (jun/2026) → notas fecham bost_Close. Desconto/juros/abatimento na baixa (JE de ajuste — validado): desconto concedido → JE {ShortName:cliente Credit=D}/{AccountCode 4.01.01.10.01 Debit=D}, aplica no IncomingPayment linha it_JournalEntry DocLine=Line_ID SumApplied=-D, TransferSum=T-D. Juros a pagar → JE {AccountCode 4.02.01.01.09 Debit=J}/{ShortName:fornec Credit=J}, aplica +J, TransferSum=T+J. Abatimento → NC avulsa aplicada no recebimento com SumApplied negativo e InvoiceType **`it_CredItnote`** (grafia exata do enum, valor 14 — NÃO "it_CreditNote"). Baixa parcial = SumApplied<DocTotal (fica bost_Open, PaidToDate parcial). Contas de controle: receber 1.01.03.01.01, pagar 2.01.01.01.01.

Estoque `InventoryPostings` (ajuste da contagem): a contagem trava o item ("already in another open document"); feche a contagem via `InventoryCountings(DE)/Close` e faça o InventoryPostings **standalone** (sem BaseDocument/BaseLine — vinculação por base falha "data not copied"). Linha: ItemCode/WarehouseCode/CountedQuantity/Price (sem "Counted").

**Estoque (processos + campos SL):** Entrada `InventoryGenEntries` (usa BPL_IDAssignedToInvoice:1, aceita dims/projeto na linha, UnitPrice=custo). Saída `InventoryGenExits`. Requisição transf. `InventoryTransferRequests` e Transferência `StockTransfers`: usam `StockTransferLines` (NÃO DocumentLines), `BPLID:1` (NÃO BPL_IDAssignedToInvoice), header FromWarehouse/ToWarehouse, linha WarehouseCode(destino)/FromWarehouseCode(origem); **NÃO aceitam CostingCode/dims** (transferência é estoque puro). StockTransfer base na requisição = BaseType 1250000001. Contagem `InventoryCountings` usa `BranchID:1` (NÃO BPLID nem BPL_IDAssignedToInvoice), linha Counted:"tYES"/CountedQuantity; ajuste via `InventoryPostings` (BaseDocument=contagem).

**Tela DRE Gerencial (módulo novo `Apps/DRE`, rota RouteDRE, menu Financeiro):** DRE por projeto + 4 dimensões puxando do razão em tempo real. Controller busca `JournalEntries?$filter=ReferenceDate ge/le` (paginado por nextLink, maxpagesize) e agrega client-side classificando por prefixo de conta: RECEITA `3.01` (sinal C), DEDUÇÃO `4.01.01.10` (D), CMV `5.02` (D), DESPESA `5.01.03` (D). DRE = Receita − Deduções − CMV = Lucro Bruto − Despesas = Result. Operacional. SegmentedButton troca o eixo (ProjectCode/CostingCode/2/3/4); clique na linha → drill-down por conta contábil. Crossjoin em JournalEntries NÃO funciona (linha não expõe JdtNum) → buscar por data e filtrar linhas no cliente. Despesas operacionais lançadas por JE (D conta 5.01.03.x com dims / C banco). Deduções só populam quando fiscal for parametrizado. Deployado no VPS teste; produção depende do Junior.

**IMPORTANTE (mapa):** a cadeia de compra e a de venda NÃO ficam ligadas por documento — encontram-se só no estoque/item. No Mapa de Relações aparecem como dois grafos separados. Para ligá-las seria preciso o assistente de procurement (SO→PO). Ver [[mapa-relacoes-vendas]] e [[auditoria-estoque-vendas]].

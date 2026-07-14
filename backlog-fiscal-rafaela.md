---
name: backlog-fiscal-rafaela
description: "Ajustes fiscais pedidos pela Rafaela (doc 08/07/2026) — diagnósticos, o que foi entregue e o que falta"
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Doc "Ajustes_SAP_Fiscal.docx" (Rafaela, Fiscal) — 13 itens. PDF do plano enviado a ela em 08/07 (Downloads/Ajustes_Fiscais_Plano_de_Acao_08-07-2026.pdf).

**ENTREGUE na Onda 1 (09/07, deployado no 161 e testado):**
- **#13 XML cruzando empresas**: causa dupla — (a) pasta xml/ única sem separação; (b) ImportaXml com URL HARDCODED da produção do Ricardo (davicaminhoes.i16br.com) → até o 161 gravava XML na produção. Fix: URLs relativas (6 ocorrências), upload grava `empresaGrupo` (CNPJ raiz do destinatário: 00700428=DAVI, 01363021=TERRAMIX, 23760674=OLIVEIRA) + fallback companyDb (localStorage portal_sap_dbo); /api/xml/list?companyDb= filtra por grupo (homolog+prod da mesma empresa veem o mesmo; arquivos sem grupo aparecem p/ todos). Testado com 2 XML sintéticos. ATENÇÃO: XMLs antigos ficaram no servidor de produção; pasta do 161 começa vazia.
- **#7 Parcelas do XML**: enviarParaSAP agora monta DocumentInstallments {DueDate, Percentage} das duplicatas (última fecha 100%) e DocDueDate = última parcela. Antes: DocDueDate=dhEmi, parcelas descartadas → tudo "à vista".
- **#2 ICMS ST**: validação da NFEntradaAdicionar somava só linha+TaxRate; agora inclui STAmount (novo campo "ICMS ST (R$)" no Resumo, handler onSTChange) + OutrasDespesas nas DUAS bases (validação e Percentage das parcelas) + no total exibido.
- **#10 Vencimento**: com parcelas, DocDueDate do cabeçalho é auto (última parcela); só obrigatório sem parcelas.
- **#11**: coluna "Nº NF Fornecedor" (NumAtCard) na lista NFEntrada (sem $select, campo já vinha).
- **#12**: status na NF Entrada agora "Em aberto (a pagar)"/"Quitada" (formatter local, não mexe no Base). Não era bug: NF fecha ao pagar.

**ENTREGUE na Onda 2 (09/07, madrugada — TUDO deployado e testado):**
- **#6+#5 Devolução de Compra**: app novo `Apps/DevolucaoCompra` (lista+adicionar+detalhe), rotas RouteDevolucaoCompra/Adicionar/Detalhe, Menu Compras > "Devolução de Compra", handler DevolucaoCompra no Base. Fluxo: busca PurchaseDeliveryNotes bost_Open → seleciona → linhas com RemainingOpenQuantity>0 (qtd editável, parcial ok) → POST PurchaseReturns {BaseType:20, BaseEntry, BaseLine, Quantity} (sem preço — herda da base). Testado ponta a ponta: estoque 286→276 e recebimento bost_Close. Devolução nº 1 (DocEntry 24) ficou como exemplo. IMPORTANTE deploy: app novo exige `docker exec mkdir -p` das pastas no container antes do deploy-161.sh.
- **#1**: quando a NF do XML sai pelo fallback sem base, o portal agora OFERECE encerrar o(s) recebimento(s) (POST PurchaseDeliveryNotes(id)/Close) — some da lista de vinculação.
- **#8 Frete no custo**: config SAP conferida (todas as 4 despesas: Stock=tYES, rateio aed_LineTotal) e mecânica PROVADA por teste real (receb 10×100 + frete 500 → custo médio 53,64→57,01; teste revertido via Cancel). É regra de PROCESSO: frete deve ir nas Despesas Adicionais do RECEBIMENTO (não da NF) p/ compor custo. Wiki id:36/37.
- RV-0001 ficou ativo (Valid tYES) — SL recusou desativar de volta (PATCH Valid:tNO retorna 204 mas não aplica; Frozen dá 400). Sem impacto.

**#4 ESTORNO (investigado 09/07):** em estado limpo o estorno da NF de Entrada FUNCIONA — testei via SL: PurchaseInvoices(DE)/Cancel retorna 204 (standalone E baseada em recebimento; ao cancelar a NF base 20 o recebimento reabre bost_Open). Logo o erro da Rafaela é caso ESPECÍFICO (NF já paga/encadeada com pagamento, ou outra tela). Proteção adicionada mesmo assim: NFEntradaDetalhe.onCancelarNFEntrada agora passa `reverseEntitySet:"PurchaseCreditNotes", baseType:18, doc` → se o Cancel falhar, o `cancelarOuEstornarMovimento` (Base) gera automaticamente a Nota de Crédito (base 18, Quantity explícita) revertendo estoque+financeiro. `_estornarPorDocumentoInverso` ganhou suporte a `oCfg.baseType` (reverso baseado no doc, fecha o original). AINDA precisa da Rafaela: QUAL documento + a MENSAGEM de erro exata (pra confirmar o caso dela).

**AINDA AGUARDANDO RAFAELA:** (#4) qual documento + mensagem exata do erro de estorno; (#9) qual fornecedor + CNPJ correto; (#3) qual campo falta no cadastro do Item.

**CFOP INTELIGENTE — ADIADO (decisão Mafra 09/07): NÃO implementar por dedução (5→1/6→2); esperar a RÉGUA da Rafaela.** O ciclo procure-to-pay completo foi validado via SL usando CFOP fallback 0001/TaxCode 0001-001 (funciona, mas não é o CFOP fiscal correto). Para fechar de vez o CFOP de entrada, PEDIR à Rafaela: (1) tabela de-para CFOP saída→entrada por tipo de operação (revenda, industrialização, uso/consumo, ativo), incluindo casos com ST/IPI; (2) a lista CURTA de CFOPs de entrada que ela usa de fato; (3) o TaxCode SAP casado com cada CFOP. Só então modelar (mexe em onGerarRecebimento/onConfirmaImposto do Mosaico + fallback das telas de entrada). Ver [[mosaico-kanban-fiscal]] (é o item 1 do "o que falta para fechar o fluxo"; item 2 = parametrização IPI/ICMS-ST no SAP, dependência Ativy).

**DOC NOVO DA RAFAELA (`Ajustes_SAP_Fiscal_Pendencias_Mafra.docx`, lido 09/07) — 3 itens, todos investigados:**
- **#9 fornecedor: CAUSA ACHADA.** `F03665112000150` = TINTAS ESTANCIA DAS CORES LTDA. O `CardCode` está
  certo; o **`FederalTaxID` tem 13 dígitos: `0366511000150` — falta o `2`** (correto `03665112000150`).
  Confirmado nas **4 bases** (SBO_HOMOLOG, SBO_DAVI_PRD, SBO_TERRAMIX, SBO_TERRAMIXHOMOLOG) — está em
  PRODUÇÃO, não é só teste. Explica o sintoma dela: Mosaico/ImportaXml casam fornecedor por
  `FederalTaxID`; o XML traz o CNPJ certo e o cadastro tem o errado → recebimento não aparece p/ vincular.
  Correção = 1 PATCH de campo. **Mafra decidiu NÃO corrigir ainda — fica pendência da Rafaela responder.**
- **#4 estorno: o número que ela deu NÃO EXISTE.** "N° NF SAP: 237" — procurado nas **7 bases** como
  `PurchaseInvoices.DocNum`, como `DocEntry`, como `NumAtCard`, como NF de saída (`Invoices`) e como
  recebimento (`PurchaseDeliveryNotes`). Zero resultados. No homolog a maior NF de Entrada é **87**.
  Falta ainda: **a mensagem de erro exata (print)** e **em qual base/tela** ela testou.
- **#3 campo do Item: item vazio.** Ela escreveu "não identificamos qual campo está ausente do Item" —
  devolveu a pergunta. Ou alguém reformula, ou encerra.

Credencial do Service Layer (usada nessa investigação, só leitura): `manager` / senha do Mafra em
`b1.ativy.com:17162/b1s/v1`. Não há credencial de serviço gravada em servidor nenhum — ver
[[agentes_runner_telegram]] se for criar tool de consulta SAP para o agente.

- **#Cleiton (13/07) — 2 bugs no Recebimento de Mercadoria, corrigidos:**
  (1) **Estorno "Código do Fornecedor está vazio":** o `_estornarPorDocumentoInverso` (Base) montava o
  PurchaseReturns/CreditNotes SEM `CardCode`. Adicionado `CardCode: doc.CardCode` no payload (deploy
  v2026-07-13T19:57). Vale p/ todos os estornos por documento inverso.
  (2) **Nº da NF trocava por número do sistema após salvar:** o campo "Número da Nota Fiscal" estava
  ligado a `SequenceSerial` (numeração fiscal que o SAP renumera). Religado a **`NumAtCard`** (convenção
  do portal p/ nº do fornecedor) na inclusão, consulta e save de RecebMercadoria (deploy v2026-07-13T20:00).
  Regra: nº da NF do fornecedor SEMPRE = NumAtCard, nunca SequenceSerial.

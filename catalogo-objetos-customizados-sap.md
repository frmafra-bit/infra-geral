---
name: catalogo-objetos-customizados-sap
description: Catálogo dos objetos customizados (UDT/UDO/UDF) e configs criados no SAP B1 do grupo Davi
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Inventário dos objetos customizados criados no SAP B1 (grupo Davi/Terra Mix). **REPLICADO E VERIFICADO EM TODAS AS 7 BASES (24/06/2026): SBO_HOMOLOG, SBO_DAVI_PRD, SBO_TERRAMIX, SBO_TERRAMIXHOMOLOG, SBO_ANGELICA, SBO_DALVES, SBO_OLIVEIRA — via script idempotente `replicar_todas_bases.py`.** Fonte: consulta direta ao Service Layer. Documento completo: `Catalogo_Objetos_Customizados_SAP.md` (Drive). Sempre que criar/alterar UDT/UDO/UDF ou config de cadastro, **atualizar este catálogo e replicar nas 7 bases**.

Gotchas da replicação: UDF em tabela de usuário exige TableName com prefixo **@** (ex.: `@DAVI_AMARR`); campo `db_Float` exige `SubType` (usei st_Price); UDO depende dos UDFs já existirem (criar UDT→UDF→UDO nessa ordem).

**UDT:** `DAVI_LICIT` (Licitações POC, NoObject); `DAVI_AMARR` (Amarração Frota, MasterData).
**UDO:** `DAVI_AMARR` (MasterData, endpoint `/DAVI_AMARR` sem prefixo U_).
**UDF:**
- `@DAVI_AMARR`: U_Placa Alfa(10), U_Contrato Alfa(20), U_SubContrato Alfa(20), U_Status Alfa(10)[ATIVO/INATIVO].
- `@DAVI_LICIT`: U_NumEdital, U_Orgao, U_Modalidade, U_ValorEstim(Float), U_DataSessao(Date), U_Status[ANALISE/PROPOSTA/DISPUTA/GANHOU/PERDEU].
- `OPRC` (centro de custo/sub-contrato): U_ProjetoSAP Alfa(20), U_ProjetoNome Alfa(100) — liga sub-contrato a [[projeto SAP]].
- `OPRQ` (Solicitação de Compras / PurchaseRequests): **U_Prioridade Alfa(1)[A=Alta/M=Média/B=Baixa, default M]** — REQ04 do [[backlog-compras-portal]].
**Config (não-UDF):** `ItemProperties` Característica 1="Original", 2="Similar" (REQ02 Tipo de Peça; via PATCH PropertyName; no item = Items.Properties1/2 tYES/tNO).
**Unidades de Medida (`UnitOfMeasurements`):** base vinha ZERADA (só "Manual"/AbsEntry -1) — por isso value-help de compras só mostrava "Manual". Em 01/07/2026 populadas **54 unidades** via POST `UnitOfMeasurements` (Code+Name): UN,PC,CJ,JG,KIT,PAR,DZ,CENTO,MIL,MG,G,KG,TON,AR,ML,L,M3,GL,MM,CM,M,KM,M2,POL,CX,PCT,FD,SC,LT,FR,BD,BB,TB,VD,AMP,BIS,ROL,BOB,RS,GF,BJ,SACH,BAR,CH,FL,PP,BLC,MET,H,DIA,MES,SERV,KWH,VB. **Replicado nas 7 bases em 01/07/2026** (autorização do CEO), confirmado por `$count`=55 em cada (SBO_HOMOLOG, SBO_DAVI_PRD, SBO_TERRAMIX, SBO_TERRAMIXHOMOLOG, SBO_ANGELICA, SBO_DALVES, SBO_OLIVEIRA). GOTCHA replicação: a checagem de existentes vem paginada em 20 (SL default) — usar `Prefer: odata.maxpagesize` ao ler p/ não tentar recriar e tomar 400 "já existe" (inofensivo, mas polui log). Itens continuam no grupo UoM "Manual" (aceitam texto livre); cadastrar grupo + amarrar UoM aos itens é projeto à parte. Ver [[backlog-compras-portal]].

Replicação p/ SBO_DAVI_PRD/SBO_TERRAMIX/etc. **pendente** de validação do Mafra na homolog.

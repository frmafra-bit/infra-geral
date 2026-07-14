---
name: faturamento-nfse-emitida-real
description: "Conteúdo real de 4 NFS-e emitidas pela Davi/Terra Mix contra prefeituras (Qive /nfse/emitted) — estrutura do faturamento público"
metadata:
  type: reference
---

**Levantado 12/07/2026** — puxadas 4 NFS-e EMITIDAS reais via `/v1/nfse/emitted` (Qive). Isto é o
FATURAMENTO da Davi/Terra Mix. Amostra fecha o modelo de vendas ([[ciclo-vendas-setor-publico]]).

## As 4 notas (fev–mar/2026)

| Nº | Prestador | Tomador (doc) | Valor Serviço | Obra |
|---|---|---|---|---|
| 649 | TERRA MIX (01363021) | 46522967000134 | **R$ 397.063,68** | Reparos prediais CAPS — contrato 106/2025, **1ª medição** |
| 657 | TERRA MIX | 26702577000139 | R$ 70.000,00 | Manutenção predial unidade de saúde c/ mão de obra — contrato de gestão 383/2022, **23ª medição** |
| 2944 | DAVI (00700428) | 46523270000188 | R$ 235.649,14 | **Locação de máquinas e caminhões** — RP 149/2025, Sec. Infraestrutura, **9ª medição** |
| 647 | TERRA MIX | 46522967000134 | R$ 155.520,92 | Tapa-buraco malha viária Ribeirão Pires — proc. 1449/2021, **medição 03** |

Nota: o prestador vem como **EIRELI** no XML (razão social do Qive pode divergir do SAP LTDA — conferir).

## O que a estrutura REAL revela (além do que a memória supunha)

1. **Faturamento é por MEDIÇÃO.** Toda nota tem "Nª MEDIÇÃO" e "PERÍODO DE EXECUÇÃO: dd/mm à dd/mm".
   Isso É a baixa parcial do contrato/empenho que o Mafra descreveu — cada medição = uma parcela do
   contrato faturada num período. A 23ª medição do contrato 383/2022 prova que um contrato gera MUITAS
   notas ao longo do tempo. Modelar: Contrato → N medições → N NFS-e.
2. **A discriminação carrega dados estruturados em texto livre:** CONTRATO Nº, PROCESSO Nº, OBJETO,
   nº da MEDIÇÃO, PERÍODO, e **dados bancários** (banco/agência/conta p/ o pagamento da prefeitura).
   Hoje é texto; num portal viraria campos.
3. **Retenções são o coração fiscal** (nota 2944, locação):
   - ValorServicos 235.649,14 | ISS 8.247,72 (aliq 3,5%, **IssRetido=1**)
   - **INSS retido 25.921,41** (IN 971/2009) | **IR retido 2.827,79** (IN 2.145/2023)
   - PIS/COFINS/CSLL = 0 nesta; ValorLiquidoNfse **198.652,22** (= serviços − retenções)
   → O faturamento público retém INSS/IR/ISS na fonte; o líquido a receber é bem menor que o bruto.
   Contas a Receber tem que trabalhar com o LÍQUIDO e registrar as retenções.
4. **Item lista serviço:** 7.02 (execução de obra) e 7.05 (reparação/conservação) — ISS no município.
   Terra Mix aliq 5%, Davi 3,5% (municípios diferentes).

## Implicação para o modelo

- **Empenho/Contrato** não é 1:1 com a nota — é **1 contrato : N medições : N notas**. O "Pedido de Venda
  = empenho" precisa suportar faturamento parcial recorrente (a cada medição).
- A nota (NFS-e de serviço) já traz **retenções** que o SAP precisa lançar (INSS/IR/ISS na fonte).
- Antes de modelar tela: decidir se o portal vai CAPTAR as emitidas do Qive (como já faz com as
  recebidas) para alimentar Contas a Receber, ou se o fluxo nasce no portal (empenho→medição→nota) e
  a NFS-e é emitida por fora (prefeitura/sistema próprio) e só conciliada.
- CT-e (frete) NÃO habilitado no Qive — ver [[integracao-qive]].

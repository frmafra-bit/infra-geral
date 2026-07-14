---
name: faturamento-davi-2026-analise
description: "Análise do faturamento REAL da Davi (NFS-e emitidas, Qive) — 34 notas de mar/2026, R$10,5M; estrutura contrato/medição/retenção"
metadata:
  type: reference
---

**Levantado 12/07/2026** via `/v1/nfse/emitted` (Qive), CNPJ Davi 00700428000147. Só a Davi.
São **34 NFS-e** capturadas, TODAS de **março/2026** (a captura paginou até esgotar; jan/fev estão em
páginas mais antigas do cursor — a API do Qive parece devolver as mais recentes primeiro). Complementar
depois puxando cursor mais fundo para pegar jan/fev.

## Números (34 notas de mar/2026)

- **Faturamento bruto:** R$ 10.530.980,06 | **líquido (após retenções):** R$ 9.082.343,52
- Retenções somaram **~R$ 1,45 milhão** (INSS + IR + ISS na fonte) — **13,7% do bruto retido**.
- Valores por nota de R$ 7 mil a **R$ 1,66 milhão** (contrato 92/2022, transporte).

## Por tipo de serviço (ItemListaServico)

| Cód | Serviço | Total mar/2026 |
|---|---|---|
| 16.02 | **Transporte** | R$ 5.180.424 (maior receita) |
| 7.10 | Limpeza/conservação | R$ 2.687.888 |
| 7.02 | Obra/execução | R$ 2.470.693 |
| 14.01b | Manutenção | R$ 184.808 |
| 11.04 | Estacionamento | R$ 7.167 |

Transporte (16.02) é o carro-chefe — coerente com "Davi Caminhões". Locação/obra vêm depois.

## Contratos (o guarda-chuva)

**15 contratos distintos** ativos, faturando em paralelo. Maiores:
- **92/2022** → R$ 3,31M (transporte, notas de R$1,6M cada — 2 medições no mês)
- **69/2023** → R$ 1,17M | **117/2023** → R$ 912k | **10/2024** → R$ 788k | **002/2022** → R$ 692k

Um mesmo contrato gera VÁRIAS notas no mês (ex.: 278/2022 tem 6 notas em 03/03; 189/2021 tem 4).
Confirma o modelo **1 contrato : N medições : N notas** ([[faturamento-nfse-emitida-real]]).

## Estrutura de referência (o que a nota carrega, em texto livre na Discriminacao)

- **CONTRATO Nº** (sempre) — o guarda-chuva. Ex.: "CONTRATO Nº 189/2021".
- **EMPENHO Nº** (às vezes) — ex.: nota 2936 "EMPENHO N° 434"; nota 2944 NÃO tem. O empenho é a
  autorização orçamentária da prefeitura; nem toda nota o cita.
- **Nª MEDIÇÃO** (frequente) — a parcela do contrato faturada naquele período.
- **PERÍODO DE EXECUÇÃO: dd/mm à dd/mm**.
- **"VALOR DOS SERVIÇOS CONFORME PLANILHA/MEDIÇÃO EM ANEXO"** — a quantidade real (horas-máquina,
  m² de via, itens) NÃO está na nota; está num ANEXO/planilha externa.
- Dados bancários (banco/agência/conta) para o pagamento.
- Campo estruturado `<CodigoObra>` (construção civil) quando é obra registrada na prefeitura.

## Fatos fiscais (confirmados nos XML)

- NFS-e ABRASF 2.02, **serviço** — NÃO tem item/quantidade/unidade como NF-e de mercadoria.
- ISS retido na fonte (`IssRetido=1`), incidência no **município do TOMADOR** (a prefeitura).
- Retenções por tipo: locação/obra (7.02) retém ISS+INSS+IR; transporte (16.02) retém ISS+IR (sem INSS).
- Simples Nacional = 2 (NÃO optante). Alíquota ISS varia por município (3,0 / 3,5 / 5,0%).

## O ponto que decide o portal (pergunta ao Mafra em aberto)

A quantidade/medição real vive num ANEXO (planilha). **Onde essa planilha mora hoje?** Se é Excel
solto, o portal agrega valor transformando a medição em documento estruturado que GERA a nota. A NFS-e
em si é emitida no sistema da prefeitura/emissor — o portal provavelmente CAPTA (do Qive) para
Contas a Receber + painel, e/ou ORIGINA a medição. Ver [[ciclo-vendas-setor-publico]].

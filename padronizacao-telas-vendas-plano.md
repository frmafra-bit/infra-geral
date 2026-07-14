---
name: padronizacao-telas-vendas-plano
description: "Plano/auditoria da padronização de TODAS as telas do ciclo de vendas (cadastro/consulta/alteração)"
metadata:
  type: project
---

**Pedido do Mafra (12/07/2026):** revisar TODAS as telas do ciclo de vendas e deixá-las IGUAIS —
cadastro, consulta e alteração, da Cotação à NF. "tem tela que tem campos, processos e não em outras".
Pode demorar; fazer em blocos com deploy incremental.

**Padrão-alvo (por tela de cadastro/consulta):** Cliente unificado (nunca Fornecedor), Condição de
Pagamento (lupa, editável, sempre disponível), toggle Produto/Serviço (`_ModoVenda`), dimensões
completas (Projeto/CentroCusto/Frota/OS/SubContratos + CFOP + Imposto de SAÍDA `ValidForAR` + Desconto
+ Referência), Conta de RECEITA no serviço (`at_Revenues`), Mapa de Relações, e NADA de cancelamento
(só devolução). Ver [[padrao-linhas-venda-dimensoes]], [[busca-cliente-8-copias]].

**Auditoria inicial (S=tem, -=falta):**
| Tela | Cliente | CondPag | Prod/Serv | Dimensões | CFOP | Imposto | Conta | Mapa |
|---|---|---|---|---|---|---|---|---|
| Cotação/Cadastro | S | ~~-~~→S | S | S | S | S | S | - |
| Cotação/Consulta | (abre pelo Adicionar edit) | | | | | | | |
| Pedido/Cadastro | S | S | - | - | - | - | - | - |
| Pedido/Consulta | S | S | ~~-~~→S(tipo) | S | S | S | S | S |
| Entrega/Cadastro | S | - | - | - | - | - | - | - |
| Entrega/Consulta | - | - | ~~-~~→S(tipo) | S | S | S | S | S |
| NF/Cadastro | S | S | S | S | - | S | - | - |
| NF/Consulta | **FORN** | S | ~~-~~→S(tipo) | S | S | S | S | S |

**FEITO:**
- Bloco 1 (v2026-07-13T12:48): Cotação ganhou Condição de Pagamento (lupa) no cadastro + no payload.
  `ajudaPesqCondPag` do Base corrigido (writeback genérico p/ o modelo do campo; binding composto; encodeURI).
- Bloco 2 (v2026-07-13T13:02): Kanban card NF mostra "A receber" + botão "Dar Baixa" → abre tela
  Contas a Receber (fluxo de baixa/IncomingPayments já testado; NÃO reimplementar baixa no Kanban).
- Consultas Pedido/Entrega/NF: colunas de dimensão + indicador Tipo Produto/Serviço (v11:07/11:39).

**FALTA (próximos blocos):**
1. Pedido/Cadastro (`PedidoVendaAdicionar`): toggle Produto/Serviço + dimensões + CFOP + Imposto saída +
   Conta receita + Mapa. Hoje só tem Cliente+CondPag e ItemCode read-only.
2. Entrega/Cadastro (`EntregaAdicionar`): idem (só tem Cliente).
3. NF/Cadastro (`NFSaidaAdicionar`): faltam CFOP, Conta receita, Mapa (já tem toggle/dim/imposto).
4. NF/Consulta (`NFSaidaDetalhe`): trocar `ajudaPesqFornecedor`/"Fornecedor" por Cliente (bug herdado).
5. Cotação/Consulta própria (`CotacaoVendaDetalhe`) está pobre — hoje o card abre o Adicionar edit; decidir
   se aposenta a Detalhe ou padroniza.
6. Mapa de Relações em TODAS as telas de cadastro.
7. Bug pendente: link do card de Pedido/NF (Detalhe) não abria — Cotação/Entrega passaram a abrir via
   Adicionar-edit (v11:30); confirmar com Mafra qual cenário (ver se detalhe está quebrada).

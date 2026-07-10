---
name: auditoria-estoque-vendas
description: Auditoria movimentos de estoque (29/06) — funcionais e quase no padrão; 1 ajuste fino + 1 decisão; só diagnosticado
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Auditoria dos **movimentos de estoque** feita em 29/06/2026 a pedido do Mafra. Decisão: **só auditar, NÃO mexer**.
Padrão de referência (aprovado): **Projeto + 4 dimensões = ProjectCode + CostingCode1..4** (a Dimensão 5 NÃO é padrão).

Situação (endpoints SAP todos corretos):
- **Entrada de Mercadoria** (`InventoryGenEntries`): funcional, Projeto + CC1-4 na inclusão. ⚠️ a CONSULTA
  (eMercadoriaview) mostra um campo extra `{CostingCode5}` (linha ~115) que a inclusão não tem → quebra leve
  do formulário único ([[padrao-formulario-unico-processo]]). Fix trivial: remover esse campo da consulta.
- **Saída de Mercadoria** (`InventoryGenExits`): funcional, Projeto + CC1-4, inclusão=consulta consistente. OK.
- **Transferência de Estoque** (`StockTransfers`): funcional, mas SEM Projeto/dimensões. Decisão de negócio
  pendente: se quiserem rastrear transferência por projeto/frota, adicionar (mesma mecânica das compras).
- **Solicitação de Transferência** (`InventoryTransferRequest`): funcional, sem dimensões (aceitável p/ requisição).

Conclusão: estoque está bem mais saudável que o [[backlog-faturamento-vendas|faturamento]]. Só padronização fina
(Entrada consulta) + 1 decisão (dimensões na Transferência). Registrado em `Status_Geral_Para_Testes_29jun2026.md`.

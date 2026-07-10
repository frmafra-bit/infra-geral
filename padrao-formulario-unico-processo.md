---
name: padrao-formulario-unico-processo
description: "Regra de arquitetura do portal — um único formulário exato por processo, do início ao fim"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

No portal Davi, cada processo deve ter **um único formulário, exato do início ao fim** — inclusão, edição e consulta usam o MESMO layout e os MESMOS campos. Não pode haver vários tipos de formulário para o mesmo processo (ex.: tela de inclusão de Recebimento ≠ tela de detalhe/consulta era um erro a corrigir).

**Why:** É a aplicação direta da regra SAP de "documentos de marketing" (ver [[projeto-migracao-sap-davi]]): todos os formulários compartilham os mesmos campos. Divergência de layout/campos entre telas do mesmo processo é tratada como inconsistência, não como feature.

**How to apply:** Ao mexer numa tela de um processo, replicar exatamente nas demais telas do mesmo processo (campos, ordem, colunas de item: Qtd, Preço Unit., Desc.%, Desc.R$, Total, Projeto + 4 dimensões). Edição liberada só para documento aberto; consulta mostra tudo read-only. Qualquer mudança real de formulário deve ser tratada como **customização** (item de escopo/governança, passa pelo Junior — ver [[equipe-e-governanca-portal]]), não como ajuste pontual numa tela só.

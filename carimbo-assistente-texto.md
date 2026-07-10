---
name: carimbo-assistente-texto
description: Frase padrão para documentar nos cadastros que a alteração/criação foi feita pelo assistente de IA
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

O usuário quer que toda alteração/criação em massa fique **documentada no próprio cadastro** (campo de observações), como vitrine do serviço para o cliente. Sempre incluir data/hora da execução (`DD/MM/AAAA HH:MM`).

Frases padrão (manter exatamente, COM acento):
- Alteração de itens (`User_Text`) e fornecedores (`FreeText`): **"Atualização realizada por Agente de Inteligência Artificial via SAP Service Layer em <data/hora>"**
- Criação de fornecedor (`FreeText`): **"Cadastro criado por Agente de Inteligência Artificial via SAP Service Layer em <data/hora>"**

**Why:** o usuário enfatizou que isso é "propaganda do serviço" e prova de valor da parceria; não terceirizar tarefas (faz a equipe parecer fraca no projeto). **How to apply:** usar nos lotes do [[projeto-migracao-sap-davi]]; em itens, concatenar com o valor antigo: `<valor antigo> | <frase>`.

---
name: projeto-migracao-sap-davi
description: Projeto de migração/padronização de cadastros SAP B1 via Service Layer para o grupo Davi Caminhões / Terra Mix
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Projeto de automação de cadastros no SAP Business One (HANA) via Service Layer, para o grupo do usuário. Service Layer único em `https://b1.ativy.com:17162/b1s/v1` (hospedagem Ativy); o mesmo login/usuário funciona em todas as empresas, trocando só o `CompanyDB`. NÃO guardar a senha aqui.

**As "4 bases" do projeto** (confirmado 22/06/2026): `SBO_DAVI_PRD` (Davi produção), `SBO_HOMOLOG` (Davi homolog), `SBO_TERRAMIX` (Terra Mix produção), `SBO_TERRAMIXHOMOLOG` (Terra Mix homolog). Outras empresas no servidor (SBO_ANGELICA, SBO_DALVES, SBO_OLIVEIRA) ficam fora — são empresas separadas e pequenas. **Replicação de impostos para Angelica e Oliveira: DESCARTADA/fora do radar (24/06/2026)** — são pessoa física, não serão tratadas (as 2 contas contábeis INSS/ISS chegaram a ser criadas, mas a estrutura de impostos não vai adiante).

**Tarefas executadas/recorrentes:**
1. Itens: mover `ForeignName` (Nome Estrangeiro, código de fabricante) → `ValidRemarks` (Observações); guardar valor antigo + carimbo em `User_Text`; limpar ForeignName.
2. Fornecedores: completar `BPPaymentMethods` com TODAS as formas próprias da empresa (cada empresa usa as dela — NUNCA copiar cartão entre empresas) + carimbo no `FreeText`.
3. Grupos de fornecedor: atribuir `GroupCode` conforme planilha `D:\Luma3D\Projeto Davi\Fornecedores_Grupo.xlsx` (casar por CNPJ/`FederalTaxID`, não por CardCode; mapear grupo por NOME normalizado sem acento, pois códigos diferem por empresa).
4. Criar fornecedores faltantes (a equipe faz, não terceiriza p/ "Ricardo") nas 4 bases, a partir da planilha, com carimbo de criação.

**Auditoria fornecedores "F" (24/06/2026, pedido da Bruna):** regra de código = **F + CNPJ**. Excluídos **15 fornecedores Categoria A** (códigos sequenciais `F00000x`, sem movimento) em SBO_DAVI_PRD (8) e SBO_TERRAMIX (7) — script `limpeza_fornecedores_F.py` (tenta DELETE; se há movimento, inativa via Frozen). **NÃO mexer** nos de Categoria B (divergência código×CNPJ por typo — fornecedor real) nem C (código já é F+CNPJ mas campo CNPJ vazio — ex.: Oliveira RAFISA/DAVI ALVES/CONSIGAZ): viraram lista de correção p/ a Bruna (`Correcao_Fornecedores_F.pdf`). GABRIELA `F0001` segue inativo. Para auditar: comparar `dig(CardCode[1:])` com `dig(FederalTaxID)`.

Regra de ouro do usuário: **sempre validar em homologação antes de produção**, backup antes de cada base, e documentar tudo com o [[carimbo-assistente-texto]] (vitrine do serviço p/ o cliente). Scripts Python ficam em `D:\Davi caminhoes\` (python nativo Windows em `C:\Users\frezu\AppData\Local\Programs\Python\Python312\python.exe`; não há jq). Detalhes técnicos em [[sap-sl-gotchas]].

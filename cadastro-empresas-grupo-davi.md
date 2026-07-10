---
name: cadastro-empresas-grupo-davi
description: "Dados legais (CNPJ, razão social, endereço) e quadro societário das empresas do grupo Davi/Terra Mix"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Dados cadastrais oficiais (Receita, via BrasilAPI 24/06/2026) das empresas do grupo. Cadastro da empresa no SAP fica em **BusinessPlaces** (Locais de Negócio/Filiais), campo MainBPL=tYES = matriz. Atualizado endereço+CNPJ via Service Layer (PATCH BusinessPlaces), **sem alterar os nomes** (preserva Produção × Homologação).

**Empresas com CNPJ (PJ):**
- **Davi Caminhões** (base SBO_DAVI_PRD / SBO_HOMOLOG) = razão social **DAVI ALVES DE OLIVEIRA LTDA**, CNPJ **00.700.428/0001-47**, CNAE 4930201 (transporte rodoviário de carga), Rod. Índio Tibiriçá 2505, Ouro Fino, Ribeirão Pires/SP, CEP 09411500.
- **Terra Mix** (SBO_TERRAMIX / SBO_TERRAMIXHOMOLOG) = **TERRA MIX PAVIMENTACAO E CONSTRUCAO LTDA**, CNPJ **01.363.021/0001-34**, CNAE 4313400 (obras de terraplenagem), Rod. Índio Tibiriçá **2511** (estava errado como 2505), Centro de Ouro Fino Paulista, CEP 09442000.
- **Oliveira & Filhos** (SBO_OLIVEIRA) = **OLIVEIRA & FILHOS EMPREENDIMENTOS LTDA**, CNPJ **23.760.674/0001-90**, CNAE 6810201 (compra e venda de imóveis), R. Irineu Sortino 37, Centro, Ribeirão Pires/SP, CEP 09401120.

**Quadro societário (sócios-administradores):**
- **Davi Alves de Oliveira** (PF) → sócio da Davi Caminhões e da Oliveira & Filhos. Base própria: SBO_DALVES.
- **Angélica Megumi Sekitani Alves de Oliveira** (PF) → sócia da Terra Mix e da Oliveira & Filhos. Base própria: SBO_ANGELICA.

**Pendências do cadastro das empresas (não preenchíveis pela Receita federal):**
- **Inscrição Estadual (VATRegNum)** vazia em TODAS as bases → precisa da contabilidade.
- **Natureza Jurídica (NatureOfCompanyCode) e Atividade Econômica/CNAE (EconomicActivityTypeCode)** = -1 em todas → exigem o código de SETUP do SAP (não o código da Receita); definir com fiscal.
- **SBO_ANGELICA e SBO_DALVES** (pessoas físicas) seguem sem CPF/endereço — QSA não traz CPF/endereço pessoal; precisa fornecer.
- **Filiais:** decisão do Mafra (24/06) = nenhuma empresa deve ter filial. Estado atual: só a MATRIZ ativa em cada base; as filiais extras (Davi/Terra Mix: "Principal" vazia + filiais de outras empresas) estão TODAS desativadas. **SAP NÃO permite apagar Local de Negócio já usado** (erro -10 "Business place must not be deleted; already in use") — só dá para deixar desativado (já feito). Angélica/Davi Alves/Oliveira já tinham só a matriz.

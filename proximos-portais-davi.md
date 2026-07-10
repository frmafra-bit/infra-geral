---
name: proximos-portais-davi
description: Próxima fase do projeto Davi/Terra Mix — dois portais específicos com responsáveis definidos
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Próxima fase do [[projeto-migracao-sap-davi]] (definido em 22/06/2026): construção de **portais específicos** que automatizam processos ponta a ponta, apoiados na base de cadastros já padronizada e no SAP Service Layer. Duas frentes:

1. **Portal de Contratos & Projetos** — capitaneado pela **Vanessa** (grupo Davi). Gestão de Contratos, Sub-Contratos e Projetos. Já existe base: Sub-Contrato = Dimensão 4/Centro de Lucro no SAP; fluxo Contrato → Sub-Contrato → Ordem de Serviço no WFrota. Escopo detalhado a levantar com a Vanessa.

2. **Projeto Financeiro** — capitaneado pela **Bruna** (grupo Davi). Processos financeiros (escopo a definir: contas a pagar/receber, conciliação, fluxo de caixa, aprovações). Formas de pagamento já padronizadas no SAP.

Roadmap detalhado em `Projeto SAP Davi TerraMix/Roadmap_Proximos_Processos.md` (Google Drive). Ação imediata: reuniões de levantamento de escopo com Vanessa e Bruna. Não confundir esta "Bruna" (financeiro) com a [[carimbo-assistente-texto]] / cadastro de Clientes pendente.

**Decisão de arquitetura (22/06/2026):** TUDO dentro do SAP B1 HANA via Service Layer (sem banco paralelo). Camadas: objeto padrão > UDF > UDT/UDO. Mapeamento: Contrato Guarda-Chuva = **BlanketAgreements** (padrão, controle de teto nativo); CRM/funil = **SalesOpportunities** (padrão); Sub-Contrato = ProfitCenters/Dim4; Licitação/Medição/Aditivo = UDO próprios. Docs em `AS-IS_TO-BE_Contratos_CRM_Vendas.docx/.pdf` e `Diagrama_Arquitetura.svg`.

**POC validada em SBO_HOMOLOG:** criei via Service Layer a tabela UDT `@DAVI_LICIT` + 6 campos + 4 licitações de exemplo (endpoint `/U_DAVI_LICIT`). Confirma que dá pra criar/operar estruturas próprias 100% por SL (UserTablesMD/UserFieldsMD/UserObjectsMD funcionam na instância). Script replicável: `scripts_estrutura/criar_estrutura_licitacao.py`. NADA criado em produção — aguarda OK. Resumo da noite em `BOM-DIA_resumo_do_trabalho.md`.

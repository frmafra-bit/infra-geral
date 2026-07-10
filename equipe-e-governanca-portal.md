---
name: equipe-e-governanca-portal
description: Equipe de desenvolvimento e regras de governança do repositório do portal Davi (GitHub)
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Portal SAP da Davi (SAPUI5 + Node/Express + PostgreSQL) no repositório GitHub **MafraFiori/DAVI_CAMINHOES**. Integra o SAP B1 via Service Layer; apps ficam em `webapp/Apps/<Nome>/Controller` + `View`; chamadas ao SL pelo helper `this.fetchApi({method,endpoint,data})` do `Base.controller`; rotas/targets no `manifest.json`; menu em `view/fragmentos/Menu.fragment.xml` (item chama handler `navTo` no Base.controller).

**Equipe de desenvolvimento (mín. 3):**
- **Junior** — diretor/chefe de desenvolvimento. **É quem AUTORIZA tudo que entra no repositório.**
- **Ricardo** — programador (fez o programa que busca dados na Receita Federal).
- **Mafra (Francisco Ezualdo Mafra, i16br)** — é o **CEO da empresa** e o usuário. Tem autoridade de decisão; valida os cases diretamente com o assistente.
- **Junior** — **diretor de tecnologia**; toca MUITOS outros projetos. Neste projeto (um _case_/piloto), ele **entra DEPOIS** que o CEO (Mafra) e o assistente validam tudo — recebe já pronto pra levar adiante. NÃO é portão antes de cada passo durante a validação.
**Negócio / comitê:** Bruna (Financeiro), Vanessa (Contratos & Projetos), Ana Pataquini (responsável por RH/pessoas; participa como membro do COMITÊ do projeto; entrou no lugar da Gabriela, que SAIU). Demais áreas: Rafaela (Fiscal), Rute (Compras), e o time do sistema de frotas/WFrota — neste, **Thiago** e **Milena** (outros nomes a confirmar).

**PADRÃO DE CÓDIGO (adotar o do Ricardo — decisão do usuário 24/06/2026):** para value-help/busca em outra tabela, usar `Input` + `showValueHelp="true"` + `valueHelpRequest="ajudaPesqX"` (NÃO ComboBox); o handler chama um helper `buscaX()` e abre um fragment de busca via `sap.ui.xmlfragment`; `onConfirmaDialogX` grava o selecionado. REUTILIZAR os helpers já existentes do `Base.controller` (`buscaProjeto`, `buscaFrota`, `buscaSubContratos`...) e os fragments `view/fragmentos/Projetos|Frota|SubContratos`, em vez de criar fetchApi novo. Fontes: dimensões via `DistributionRules` (InWhichDimension + Active eq 'Y'); projetos via `Projects` (Active eq 'tYES'). Validar existência na mesma fonte do dropdown. Objeto de amarração placa→contrato/sub: UDO `DAVI_AMARR` (endpoint /DAVI_AMARR, SEM prefixo U_ por ser UDO).

**REGRA DE GOVERNANÇA (atualizada 01/07/2026 — reforçada pelo CEO):** **É TUDO COM O MAFRA (CEO).** Ele instruiu explicitamente "esquece o Jr e o Ricardo, é tudo comigo" — as decisões são **diretas com o CEO**, sem rotear/deferir pra Junior ou Ricardo. NÃO dizer "precisa do aval do Junior/Ricardo". A única salvaguarda que o assistente mantém é **confirmar com o PRÓPRIO Mafra antes de ações irreversíveis/de alto impacto** (mudar config global do SAP, apagar dados, mexer em produção, credenciais) — e isso é **proteção ao CEO, decisão dele**, não hierarquia. Ponto prático que continua real (não é "pedir permissão"): o repo `MafraFiori/DAVI_CAMINHOES` é compartilhado com o time, então push pra lá expõe o trabalho ao Ricardo — por isso manter na NOSSA área separada; mas a decisão é do Mafra. `git fetch` (inbound, leitura) é livre e deve ser feito antes de cada trabalho para acompanhar Ricardo/Junior (há branch `Teste-FioriElements` no remoto). Clone local em `D:\portal-davi`. Ver [[proximos-portais-davi]].

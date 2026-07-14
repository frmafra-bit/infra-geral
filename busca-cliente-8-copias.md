---
name: busca-cliente-8-copias
description: "A busca de cliente (buscaClientes) está COPIADA em 8 lugares — corrigir uma não conserta as outras"
metadata:
  type: reference
---

**Descoberto 12/07/2026** ao corrigir "Erro busca Fornecedor" na pesquisa de CLIENTE.

**Armadilha:** `buscaClientes` (e `preencheModeloClientes`/`alimentaModeloCLientes`) está **copiada em
8 lugares** — NÃO é uma função única no Base:
- `webapp/controller/Base.controller.js` (a que a maioria das telas herda)
- `NFSaida/Controller/NFSaidaAdicionar` · `Entrega/Controller/{Entrega,EntregaAdicionar}`
- `Devolucao/Controller/{Devolucao,DevolucaoAdicionar}` · `PedidoDevolucao/Controller/{PedidoDevolucao,PedidoDevolucaoAdicionar}`

Corrigir uma NÃO conserta as outras — o usuário via erro conforme a tela. **Todas as 8 foram
unificadas (deploy v2026-07-13T02:26Z).**

**O padrão correto (aplicado nas 8):**
- Busca ÚNICA: um campo, procura em nome+código+CNPJ com `or` (não força escolher critério).
  Vazio = lista todos. Fragmento `view/fragmentos/Clientes.fragment.xml` virou `SearchField`.
- **Filtro CODIFICADO:** `encodeURIComponent(filtro)` — a causa do "Erro busca Fornecedor" era o filtro
  composto (parênteses + `or`) indo **sem encode** na URL; o `fetchApi` faz `fetch(servidor+endpoint)`
  cru, e parênteses/`or` se perdiam. Filtro simples antigo não tinha parênteses, por isso funcionava.
- **CNPJ via FederalTaxID** (não `BPFiscalTaxIDCollection[0].TaxId0`) — a coleção deixava a busca lenta
  e quebrava com o `$select` enxuto. FederalTaxID já É o CNPJ.
- Mensagem de erro mostra a causa real ("Erro ao buscar clientes: <msg>"), não o rótulo trocado "Fornecedor".
- `onFiltraCliente` (Base) lê o SearchField por id e chama `buscaClientes(null, valor)`.

**Segunda armadilha — `onPressCliente` copiado em 13 lugares** (todas as telas de venda +
ContasAreceber). Cada cópia lia `oValue.getSource().mProperties.title` como CÓDIGO. Ao trocar o
fragmento para mostrar o NOME no título (`title="{CardName}" description="{CardCode}"`), código e nome
vieram trocados; e a cópia da Cotação/Licitação lia `mProperties.subtitle`, que o `StandardListItem`
NEM TEM → nome `undefined` e, ao setar `Cotacao.CardCode` num objeto ainda não inicializado, exceção
antes do `.close()` → **lupa travada, não fecha**. Corrigido (deploy v2026-07-13T02:47Z): todas as 13
cópias agora leem do **binding context** (`oValue.getSource().getBindingContext().getProperty("CardCode"/"CardName")`),
independente de como a linha é exibida. Lista `grep -rln "onPressCliente:" webapp/`.

**Regra:** se mexer na busca OU seleção de cliente/fornecedor, procurar TODAS as cópias
(`grep -rl "buscaClientes:"` e `grep -rl "onPressCliente:"`) antes de dar por resolvido. Nunca ler
`mProperties.title/subtitle` para pegar dado — usar `getBindingContext().getProperty(...)`.
Idem para `buscaFornecedor`. Ver [[ciclo-vendas-setor-publico]].

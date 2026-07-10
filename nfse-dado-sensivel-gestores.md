---
name: nfse-dado-sensivel-gestores
description: Diretriz de privacidade — NFS-e do Qive expõe notas dos PJ dos gestores; NÃO deixar visível no portal
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

**Diretriz do Mafra (09/07/2026):** a captura de NFS-e (serviços) do Qive traz TODAS as notas de quem presta serviço ao grupo — inclusive as **notas fiscais dos PJ que são os próprios GESTORES/donos** da empresa (pró-labore/pagamentos via PJ). Isso é **dado altamente confidencial** dos sócios.

**Por quê:** expor isso num portal que a operação (Rute/Rafaela/Bruna) acessa vazaria a intimidade financeira da gestão pra dentro da própria empresa — risco grande.

**Como aplicar:**
- **NÃO deixar a tela de Notas de Serviço no ar** (nem no 161 de teste, onde o time valida). NÃO oferecer/comentar proativamente que temos essa capacidade.
- Só construir/reativar isso **se e quando eles pedirem explicitamente** — e, mesmo aí, **com controle de acesso** (só perfil autorizado vê).
- Princípio: **minimização de dados** — não expor informação confidencial sem necessidade e sem trava de acesso. (Não é "esconder que dá"; é governança de dado.)
- O backend de captura NFS-e (`/api/mosaico/nfse`, extractNfseData em mosaico.js — ver [[integracao-qive]]) pode ficar DORMENTE; a **visibilidade** (card Home, item de menu, rota RouteNotasServico) deve ser removida. Considerar apagar o cache `data/mosaico/nfse/` no 161.

**Status (RESOLVIDO 09/07 v2026-07-09T19:58:39Z):** tela `Apps/NotasServico` **INIBIDA** — removida do menu (Menu.fragment.xml) e da Home (card tile_notas_servico apagado). Acesso restrito por trava no controller: `_onObjectMatched` redireciona pra Home se `retornaUsuario() !== 'manager'`. Ou seja, **só o Mafra logado como usuário `manager`** consegue abrir, e SÓ pela URL direta `#/NotasServico/` (não há mais link visível). Rota RouteNotasServico + backend `/api/mosaico/nfse` continuam existindo (dormentes p/ os demais). Cache `data/mosaico/nfse/` mantido (Mafra: "não refazer tudo"). Captura de NF-e de mercadoria do Mosaico segue normal — restrição é só NFS-e de serviço.

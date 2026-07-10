---
name: tratamento-pneu-pendente
description: Cenário do pneu (ativo/estoque/despesa) em definição pela gestão Davi — afeta custo e amarração
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

**Tratamento contábil do PNEU está EM ABERTO (decisão de gestão/contábil, não técnica) — registrado 24/06/2026.**

Ciclo de vida e a indefinição:
1. Pneu comprado **junto com o caminhão** → embutido no **ativo** (veículo); não existe como item separado ainda.
2. Pneu desgasta → vai para **recauchutagem**; é a partir daí que passa a ser tratado como **estoque** (item rastreável).
3. Classificação em transição: **gestora antiga lançava como DESPESA**; **gestão nova entende como ESTOQUE** (só após o recauchute).

Ou seja, se o pneu é **ativo, estoque ou despesa** depende do momento do ciclo e ainda **não foi decidido** (pendente da nova gestão — Bruna). Não modelar regra de pneu antes da definição.

Impacto no projeto:
- **Dimensão 2 = Frota/Pneu** tem placa E pneu como coletores de custo (intencional, não é bug — ver [[equipe-e-governanca-portal]]). Uma OS pode receber custo de placa e de pneu separadamente; visão de custo = por OS, por placa e por pneu.
- **Amarração `DAVI_AMARR` fica SÓ POR PLACA por enquanto.** Pneu NÃO entra na amarração até a gestão definir o tratamento (ativo/estoque/despesa) e o fluxo de recauchutagem. Ver [[proximos-portais-davi]].

Quando a gestão decidir: modelar como o pneu vira estoque pós-recauchute, o rateio de custo e se precisa de amarração própria (Pneu→Sub-Contrato) ou se herda da placa.

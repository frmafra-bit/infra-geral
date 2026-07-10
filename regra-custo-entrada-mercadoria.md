---
name: regra-custo-entrada-mercadoria
description: "Regra de negócio (Mafra, 30/06): Entrada de Mercadoria deve usar pelo menos o custo médio; sem custo bloqueia e manda pro Inventário"
metadata:
  type: project
---

Regra de negócio definida pelo Mafra (30/06/2026) para a **Entrada de Mercadoria** (Goods Receipt, `InventoryGenEntries`):
o portal deve usar **pelo menos o custo médio existente**; se o item não tem custo (custo médio R$ 0,00),
**bloquear a entrada** e orientar a usar a **rotina de Inventário** (contagem/ajuste).

Motivação: descoberto que o SAP **aceita** estoque sem custo (não reclama via Service Layer) — entrada com
`UnitPrice: 0` cria quantidade no estoque com valor 0 e **sem lançamento contábil** (sem TransNum). Isso distorce
o custo médio. Ex.: Entrada nº 44 (item AC-0002, UnitPrice 0) ficou Total 0 e sem JE.

Comportamento do SAP confirmado por teste:
- Entrada com `UnitPrice: 0` explícito → aceita, Total 0, sem JE.
- Entrada SEM informar preço → SAP usa o custo do item → Total e JE corretos.

IMPLEMENTADO em `eMercadoriaADD.controller.js` (onSalvaSMercadoria): no save, se a linha vier com preço <= 0,
chama `_custoMedioItem(itemCode)` que lê **`MovingAveragePrice`** (fallback `AvgStdPrice`) do item
(`GET Items('cod')?$select=MovingAveragePrice,AvgStdPrice` — é NÍVEL DO ITEM, não por depósito; o campo
`StandardAveragePrice` da ItemWarehouseInfoCollection vem 0 e NÃO serve). Se custo > 0 usa ele; se 0, bloqueia
com mensagem mandando usar o Inventário. Deployado no VPS teste. Custos exemplo: EL-1254=15,24, AC-0002=89,70.

Relacionado: [[estorno-e-ver-lancamento-contabil]] (Ver Lançamento Contábil mostra "sem lançamento" justamente
nos docs de valor 0). Bug de case corrigido junto: controllerName eMercadoria**view** (v minúsculo) tinha de bater
com o arquivo no Linux case-sensitive.

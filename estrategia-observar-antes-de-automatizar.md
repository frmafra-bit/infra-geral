---
name: estrategia-observar-antes-de-automatizar
description: "Direção acordada — acompanhar as rotinas dos times antes de automatizar; padrão primeiro, automação depois"
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

**Direção acordada com o Mafra (09/07/2026):** antes de construir novas automações, ACOMPANHAR as rotinas reais dos times para PEGAR O PADRÃO. Só depois de entender o padrão, pensar em automação. Motivo: já entregamos muito; automatizar sem ver o time rodando é risco de automatizar o processo errado.

**Times a observar:** [[backlog-compras-portal|Rute (Compras)]], [[backlog-fiscal-rafaela|Rafaela (Fiscal)]], Bruna (Financeiro), Financeiro (Contas a Pagar/Receber — Rafaela+Bruna, ver [[estado-portal-receb-junho2026]]).

**Como capturar cada rotina (estruturado, p/ virar critério de automação):**
1. **Gatilho** — o que dispara (chega XML/NFe no Qive, chega pedido, título vence, etc.)
2. **Passos** — o que a pessoa faz, tela a tela
3. **Frequência/volume** — quantas vezes por dia/semana
4. **Dor** — onde trava, retrabalha ou depende de outra pessoa

Regra de decisão: **gatilho claro + repetição alta + dor = candidato a automação**; o resto fica manual.

**Telemetria que o portal JÁ dá (observar sem incomodar o time):** Fila de Pendências (o que falta cadastrar e com que frequência); [[vigia-telegram-documentos|Vigia do Telegram]] (ritmo real de documentos novos no SAP); [[pratica-registro-inicio-fim|registros início/fim no wiki]] (tempo por atividade); Monitor de IA (o que já perguntam ao assistente).

**Status:** fase de OBSERVAÇÃO. Não iniciar automações novas até ter o padrão mapeado. (CFOP inteligente e outras ideias ficam represadas até isso — ver [[backlog-fiscal-rafaela]].)

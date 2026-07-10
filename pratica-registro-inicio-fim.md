---
name: pratica-registro-inicio-fim
description: Toda atividade deve registrar início (dia/hora) e conclusão (dia/hora) — argumento de velocidade da consultoria
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Mafra (09/07/2026): registrar em TODA atividade o **início (dia/hora)** e a **conclusão (dia/hora)** — mesma prática combinada com o outro Claude (MacBook). Vira argumento comercial de velocidade da consultoria ao vender o serviço.

**Why:** o tempo entre pedido→entrega é o diferencial de venda da consultoria; sem carimbo de hora não há prova.

**How to apply:**
1. **Wiki** (Base de Conhecimento): preencher os campos `rec` (recebido) e `res` (resolvido) SEMPRE com `dd/mm hh:mm` — nunca "tarde"/"noite". Feito retroativamente nos ids 31-37 (recebido 08/07 19:44 = timestamp do docx; resolvido = hora BRT do deploy).
2. **Telegram** (`~/bin/tg` no 161 já carimba data/hora): ao INICIAR atividade relevante mandar "▶️ INÍCIO: <atividade>"; ao concluir "✅ CONCLUÍDA: <atividade> — iniciada <dd/mm hh:mm>, concluída <dd/mm hh:mm> (duração)".
3. Fonte da hora de início: timestamp do pedido (arquivo/mensagem); fim = hora do deploy/validação.
4. O relógio do VPS 161 é BRT (-03); version.json do deploy fica em UTC (converter -3h).

Ver [[base-conhecimento-wiki]] e [[vigia-telegram-documentos]].

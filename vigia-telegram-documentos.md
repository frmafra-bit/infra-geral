---
name: vigia-telegram-documentos
description: Vigia que avisa no Telegram a cada documento novo criado no SAP (rodando no Contabo)
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

**Vigia de documentos → Telegram** (montado 01/07/2026). Avisa o CEO (Mafra) no Telegram toda vez que um documento novo é criado no SAP.

- **Onde roda:** Contabo de TESTE `161.97.185.151`, em `/home/administrador/vigia/vigia.py` (chmod 700, o token fica só aí). Estado em `state.json` (maior DocEntry visto por tipo); log em `vigia.log`; lock em `vigia.lock`.
- **Agendamento:** cron do usuário `administrador` — `* * * * *` (a cada 1 min). Ver com `crontab -l`.
- **Como funciona:** loga no Service Layer (SBO_HOMOLOG, usuário `manager`; senha no .env do servidor), para cada tipo busca `DocEntry gt <ultimo_visto>`, manda 1 mensagem por documento (tipo, nº, parceiro, quem criou via UserSign→Users.UserName, valor) e atualiza o state. 1ª execução = baseline (só grava o máximo atual, sem spam).
- **Bot:** @Daviportalalertas_bot (nome "Davi portal"). **chat_id do Mafra = 6734074491** (Francisco Mafra / @Femafra). Token no script (se vazar, `/revoke` no BotFather e trocar).
- **Tipos monitorados (15):** PurchaseRequests, PurchaseQuotations, PurchaseOrders, PurchaseDeliveryNotes, PurchaseInvoices, Quotations, Orders, DeliveryNotes, Invoices, Returns, ReturnRequest, CreditNotes, VendorPayments, IncomingPayments, **InventoryGenExits (Baixa de Estoque)**.
- **Baixas de estoque (Frotas):** a integração com o sistema de Frotas cria `InventoryGenExits` sob o usuário **frota01 = UserSign 17**. O vigia marca essas com "(via integração Frotas)". Constante `FROTA_USERSIGN=17` no script. Ao adicionar um tipo novo, o vigia faz baseline automático (não spamma os antigos).

**Gerência:** pausar = remover a linha do `crontab -l`/`crontab -e`; mudar escopo/formato (individual vs resumo) = editar `TIPOS`/lógica no vigia.py e reenviar. Fonte é a base SBO_HOMOLOG (onde caem os docs reais dos usuários, ex.: Rute). Para produção/outras empresas, precisaria do acesso às outras bases. Ver [[fluxo-completo-compra-venda-teste]] e [[equipe-e-governanca-portal]].

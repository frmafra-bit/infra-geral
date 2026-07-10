---
name: arquitetura-dados-portal
description: Arquitetura de dados do portal — camadas SAP (Service Layer) x banco próprio no 161; onde cada dado mora
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

**Direção de arquitetura de dados (decidida com o Mafra, 09/07/2026)** — base pra "camada de inteligência" própria (planejamento de estoque, telemetria, sugestões, ideias que o SAP não tem). Ver [[estrategia-observar-antes-de-automatizar]].

**Modelo em 3 camadas:**
1. **SAP HANA = sistema de registro (a verdade transacional).** Hospedado na **Ativy**. Nosso acesso é **SÓ Service Layer (REST)** — NÃO há SQL/ODBC direto (a Ativy só libera via VPN controlada; **Mafra descartou** — não compensa). Nunca escrever direto nas tabelas do SAP (quebra integridade/suporte). Dado nosso dentro do SAP = **User Table (UDT)** via SL (já fazemos: `@AMARR_FORNPROD`). Não encher o HANA (licença/espaço da Ativy) com nossa telemetria.
2. **Nosso banco no 161 = o CÉREBRO.** Analítico/inteligência: telemetria (erros/chamados/uso), parâmetros calculados de planejamento, sugestões, observação de rotina, espelho de movimentos. **Estado atual: SQLite `data/tickets.db` JÁ RODA** (Chamados em backend/tickets.js + RBAC em backend/roles.js). A lib `postgres` está no package.json mas **o servidor Postgres NÃO está de pé** (sem config, fora do docker-compose) — hoje "temos Postgres" = só o cliente. Decisão pendente: subir Postgres no 161 (container+volume) como casa oficial, OU começar no SQLite e migrar depois.
3. **SQLite = operacional leve** (o que já roda: tickets/RBAC).

**Regra de ouro:** saldo/movimento = LÊ do SAP na hora; nosso banco guarda o CÁLCULO e a DECISÃO (referência por ItemCode/Warehouse/DocEntry — nunca duas verdades). Parâmetro oficial que o SAP precisa usar (mín/máx/ROP) → VOLTA pro SAP (campo do item ou UDT) via SL.

**"Lentidão do Service Layer" resolvida por ESPELHO INCREMENTAL:** job puxa só o delta de movimentos por cursor/data (mesmo padrão do sync do Qive no Mosaico, já pronto) → acumula histórico no nosso banco → cálculo roda local/rápido. SL só entrega o delta, nunca vira gargalo.

**Planejamento de estoque (visão, ainda não construído):** por item+depósito calcular estoque de segurança (Z×desvio×√leadtime), ponto de pedido (demanda média no leadtime + seg), mín/máx, cobertura, giro, curva ABC → sugestão de compra quando saldo ≤ ROP (gera SC que o Mosaico/Compras já sabe criar). Tabelas esboçadas: errors, tickets(existe), item_planning, purchase_suggestions, routine_log.

**CONFIRMADO no 161 (09/07, via SSH administrador):** PostgreSQL **14.23 nativo RODANDO** (systemd postgresql ativo, dados /var/lib/postgresql/14/main), escutando **127.0.0.1:5432** (só localhost). **Usado por vários projetos irmãos** no mesmo VPS (ai-news-monitor, folha-contabil, import-xml, nfe-portal-bw, portal-cafe, portal-folha…). O `administrador` **NÃO tem sudo sem senha** → não dá pra `sudo -u postgres` nem inspecionar bancos/roles sem credencial. Nosso `davi_caminhoes/.env` tem `DATABASE_URL` **comentada** apontando p/ **Neon (Postgres cloud AWS)** — DESLIGADA; o portal hoje não usa Postgres (só SQLite). ⚠️ Essa URL Neon tem credencial real exposta no arquivo (mesmo comentada) → ROTACIONAR/remover. **Para usar o PG local como cérebro falta:** (1) criar banco+role nossos (ex. portal_davi/davi_app) — precisa superusuário postgres (sudo/senha, o Mafra libera); (2) rede: PG em 127.0.0.1 + portal em Docker → ligar container ao PG do host (gateway + pg_hba). Sync incremental (padrão Qive) roda por cima.

**PROTÓTIPO B ENTREGUE (09/07, só leitura, scratchpad/proto_estoque.js):** motor de curva ABC rodando contra o SAP real via SL. Resultado homolog: 20 itens com saldo, R$ 90.069 valor total, classe A=5 itens=75,4% do valor (padrão 80/20 confirmado). Método validado. GOTCHAS SL aprendidos: (a) `ItemWarehouseInfoCollection` NÃO aceita $expand nem vem confiável em lista — usar `QuantityOnStock` (saldo total, vem certo na lista); (b) SEM header `Prefer: odata.maxpagesize` o SL devolve só 20/página; (c) dá pra filtrar `QuantityOnStock gt 0` no servidor; (d) campos `IsCommited`/`OnOrder` NÃO existem no Item. Falta p/ ROP: demanda (espelho de movimentos) + lead time (param nosso).

**PENDENTE (Mafra pediu 09/07): AUDITAR os fontes dos projetos no 161** (feitos por mim via chat claude.ai) — vários no VPS: nfe-portal-bw, portal-cafe, portal-cnc, portal-folha, folha-contabil, import-xml, tela-usuarios, ai-news-monitor, nfse-backend, meu-projeto. Fazer depois do banco.

**PROVISIONADO E SCHEMA CRIADO (09/07) — Mafra liberou a senha padrão dele p/ sudo, eu fiz tudo por SSH:** banco **`portal_davi`** + role **`davi_app`** (LOGIN, senha = padrão do Mafra) criados via `sudo -u postgres` (senha padrão do Mafra). Conexão app: `PGHOST=127.0.0.1 PGPORT=5432 PGUSER=davi_app PGDATABASE=portal_davi` (senha = padrão do Mafra; NÃO gravar literal aqui). **Schema criado** (5 tabelas): `errors`, `routine_log`, `estoque_snapshot` (PK data_ref+item_code), `item_planning` (PK item_code+warehouse), `purchase_suggestions`. **Carga inicial OK:** curva ABC do SAP gravada em estoque_snapshot (data_ref 2026-07-09): A=5/R$67.892, B=6/R$16.323, C=9/R$5.853. Scripts em scratchpad (gera_seed.js gera o SQL; aplicado por psql -f no VPS). NOTA: o brain job gerou o SQL LOCAL e aplicou via psql; p/ recorrência mover o job pro VPS. Container ainda NÃO ligado ao PG (rede Docker→host pendente — só quando a UI precisar). Credencial Neon exposta no .env AINDA a limpar (Mafra não pediu ainda).

**PRÓXIMOS (quando retomar):** (1) espelho incremental de MOVIMENTOS (consumo) no Postgres → calcular demanda média/desvio → preencher item_planning (ROP, estoque segurança) → gerar purchase_suggestions; (2) coletor `/api/errors` (gatilho Telegram); (3) ligar container ao PG (Docker networking) qd a UI usar; (4) auditar fontes dos projetos do 161.

**Primeiro passo (a decidir):** (A) fundação + coletor `/api/errors` (erro crítico sobe pro servidor → gatilho Telegram, reusa vigia); (B) protótipo só-leitura de planejamento (movimentos de 1 depósito → ROP/seg/ABC) pra ver o padrão. Tudo no 161, NADA na produção do Ricardo.

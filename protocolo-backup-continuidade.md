---
name: protocolo-backup-continuidade
description: REGRA PERMANENTE — manter código e memória redundantes no GitHub p/ o notebook não ser ponto único de falha
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

**Compromisso com o Mafra (10/07/2026):** o notebook Windows está instável. Se ele morrer, o Mafra tem que abrir em OUTRA máquina e **continuar como se nada tivesse acontecido**. Portanto, código e memória NÃO podem existir só localmente.

**Locais de redundância (os "3-4 lugares"):**
1. **Código** → GitHub `MafraFiori/DAVI_CAMINHOES`, branch `feature/despesas-cotacao` (backup; NÃO é a main, não afeta Ricardo/Jr). ✅ EM DIA (push 10/07, commit d6e7ddf).
2. **Memória** → repo git LOCAL já commitado (em `~/.claude/projects/.../memory/`), + [🔴 PENDENTE: repo PRIVADO no GitHub — falta criar; sugestão sob a conta pessoal do Mafra, NÃO no repo do código que o Jr acessa, pois a memória tem notas internas — ver [[nfse-dado-sensivel-gestores]]].
3. **Cópia no Mac** → `~/davi-memoria/` e `~/.claude/projects/-Users-mafra-portal-davi/memory/` (defasada — de 09/07; Mac estava dormindo em 10/07; sincronizar quando acessível).
4. **Backup no VPS 161** → `~/davi-memoria-backup/` (administrador@161.97.185.151) — cópia FRESCA da memória, atualizada 10/07 15:45 via scp do Windows. É o backup off-machine mais confiável hoje (servidor estável, com meu acesso SSH), enquanto o GitHub da memória não sobe.

**STATUS 10/07:** push da memória pro GitHub BLOQUEADO — a conta `frmafra` (a que tem credencial salva no Windows) pede **2FA**, e os repos que o Mafra criou estão em OUTRAS contas (`mafra-byte`, `frmafra-bit`) onde `frmafra` não tem escrita. Mafra vai resolver o 2FA depois. Enquanto isso, a redundância está garantida pelo **backup no 161** (item 4). Quando o 2FA for resolvido: criar `frmafra/davi-memoria` (privado) e `git push` (o repo local da memória já está commitado e limpo).

**REGRA (toda sessão de trabalho relevante — SEM EXCEÇÃO):**
- Mudou **memória** → `git add/commit` no repo da memória e **`git push`** (assim que o remoto privado existir).
- Mudou **código** → deploy no 161 **E** `git commit`/`push` da branch de backup. Nunca encerrar sem o push.
- Antes de qualquer push: **varredura de segredos** (a chave do Qive/Telegram/SAP fica só no `.env`; senhas mascaradas na memória). Ver [[integracao-qive]].

**HONESTIDADE:** o Claude NÃO roda em background — só sincroniza durante as sessões. Por isso o ideal é **AUTOMATIZAR** o push (hook git ou tarefa agendada) que sobe a memória sozinho, removendo a dependência de "lembrar". Configurar isso assim que o repo privado da memória existir.

**PARA FECHAR A CONTINUIDADE (pendências):** (1) criar repo privado da memória + push; (2) ligar auto-push da memória; (3) rotacionar as 4 credenciais que vazaram (Google service account = a mais séria, Qive, senha root/sudo, SAP) — ver [[estado-portal-receb-junho2026]] e [[integracao-qive]].

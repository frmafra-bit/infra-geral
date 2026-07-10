---
name: foto-produto-armazenamento
description: Onde ficam as fotos de produto do portal (volume no VPS agora; decisão futura pendente)
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c164253-460f-45fc-b134-e695e45e6667
---

Foto de produto no Cadastro de Produto (tela ProdutoView, aba "Informações Adicionais", logo abaixo de Referência). Backend: [[catalogo-objetos-customizados-sap]] não se aplica — é armazenamento portal-side, não SAP.

Rota `backend/routes/produtoFoto.js` (POST/GET/DELETE `/api/produto/foto/:itemCode`), registrada no `server.js` (antes do express.static). Vínculo é só por **código do produto** (ItemCode).

**Problema (04/07/2026):** gravava em `/app/webapp/uploads/produtos` → **efêmero**: some quando o container é recriado (compose up). A foto gravava e depois desaparecia. (Sobrevive a `docker restart`, só não sobrevive à recriação.)

**Solução aplicada (08/07/2026) — SEM recriar container:** anexos gravam em `/app/data/uploads/produtos`, dentro do bind-mount **persistente** que já existe (`~/davi_caminhoes/data` → `/app/data` no VPS 161). `produtoFoto.js` DIR = `path.join(__dirname,'..','..','data','uploads','produtos')`. Deploy por `docker cp server.js` + `docker cp produtoFoto.js` + `docker restart` (backend não passa pelo deploy-161.sh).

**Alinhado ao padrão do Ricardo (08/07/2026):** ele grava anexos de Chamados em `data/attachments` (mesmo mount `data`) e **serve por ROTA dedicada** (`res.download`/`sendFile`), não por pasta estática pública. Espelhei: removi o `express.static('/uploads')` do server.js; agora serve por `GET /api/produto/foto/:code/arquivo` (res.sendFile, inline). Metadata `GET /api/produto/foto/:code` retorna `{url:'/api/produto/foto/<code>/arquivo', ext, isImagem}`. O estático `/uploads/...` agora dá 404 (não exposto). Ref.: `backend/routes/tickets.js` (upload.array em `data/attachments`, download por rota).

**VÁRIOS anexos por produto, cada um com NOME (08/07/2026):** aceita jpg/jpeg/png/webp/gif + pdf/doc/docx/xls/xlsx. Modelo espelha o SL (Attachments2_Lines: FileName + FreeText/nome) — pronto p/ migrar ao SAP quando a ativy liberar a Attachments folder (testado 08/07: ainda dá -5002 "Attachments folder not defined"). Items tem AttachmentEntry; Attachments2_Lines é coleção com FileName, FileExtension, SourcePath, FreeText.
Armazenamento novo: `data/uploads/produtos/<code>/` com arquivos `<id><ext>` + `_meta.json` (array {id,nome,fileName,ext,isImagem,size,uploadedAt}). Migração automática do formato antigo (1 arquivo `<code>.<ext>` solto) na 1ª leitura — nada se perde.
Endpoints: GET `/api/produto/anexos/:code` (lista), POST `/anexos/:code` (campo 'arquivo' + 'nome'), PATCH `/anexos/:code/:id` (renomear), GET `/anexos/:code/:id/arquivo` (serve inline), DELETE `/anexos/:code/:id`. Filtro da lista usa GET `/fotos/codigos` (detecta formato novo por pasta+meta). Compat: GET `/foto/:code` retorna o 1º anexo.
UI (ProdutoView, aba Informações Adicionais): Input nome + FileUploader "Anexar" + Table `{/Anexos}` com Nome/Arquivo/Visualizar/Excluir. `carregaAnexos` chamado em `preencheModeloItem` (ponto único) por ItemCode. Foto por produto corrigida (não gruda entre produtos).

IMPORTANTE: o host `~/davi_caminhoes/webapp` está DESATUALIZADO — deploys vão via `docker cp` só pro container. Recriar o container faz perder os deploys feitos por docker cp. Por isso a solução usou a pasta `data` já montada, em vez de bind-mount novo.

**Decisão futura (Mafra):** por ora persistente no VPS 161 (teste). Depois pedir local definitivo ao cliente OU usar **anexos do SAP B1** quando a ativy liberar a pasta ("Attachments folder not defined", ver [[sap-sl-gotchas]]). Não subir para a produção do Ricardo (ver [[equipe-e-governanca-portal]]).

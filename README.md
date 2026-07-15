# nima-n8n

Serviço de IA do **Nima** (projeto separado do backend). Recebe as respostas do
questionário, chama a **OpenAI** com o prompt de avaliação de adoção e devolve o
parecer + nota para o backend.

- Workflow importável: [`nima-analise-adocao.json`](./nima-analise-adocao.json)
- Deploy no Render: [`render.yaml`](./render.yaml)

```
Webhook (POST /nima-analise-adocao)
   → Montar Prompt (formata as respostas)
   → OpenAI Chat (gpt-4o-mini)
   → Extrair Parecer + Nota ("Nota Final: X/100")
   → Responder ao Backend  →  { "relatorio": "...", "score": 87 }
```

O backend (`nima-backend`, `src/servicos/n8nService.js`) manda `{ tutor, respostas }`
para a `N8N_WEBHOOK_URL` e lê a resposta `{ relatorio, score }`.

---

## Rodar localmente

```bash
npx n8n
# ou
docker run -it --rm -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

1. Abra `http://localhost:5678` e crie a conta local.
2. **⋯ → Import from File** → `nima-analise-adocao.json`.
3. **Credentials → New → OpenAI** → cole a API key; selecione essa credencial no nó **OpenAI Chat**.
4. Ligue **Active**. Webhook de produção: `http://localhost:5678/webhook/nima-analise-adocao`.
5. No backend, defina `N8N_WEBHOOK_URL` com essa URL.

### Teste rápido
```bash
curl -X POST http://localhost:5678/webhook/nima-analise-adocao \
  -H "Content-Type: application/json" \
  -d '{"tutor":{"nome":"Teste"},"respostas":{"1. Tempo disponível":"6h/dia","11. Espécie desejada":"Cão"}}'
```

---

## Deploy no Render (servidor próprio)

1. Suba este projeto num repositório GitHub próprio (ex.: `nima-n8n`).
2. Render: **New → Blueprint** → conecte o repo (lê o `render.yaml`).
   - Ou **New → Web Service → Existing image** `docker.n8n.io/n8nio/n8n:latest` e configure manualmente.
3. Após o primeiro deploy, defina no painel `N8N_HOST` e `WEBHOOK_URL` com a URL final
   (ex.: `nima-n8n.onrender.com` / `https://nima-n8n.onrender.com/`) e redeploy.
4. Abra a URL, importe o workflow e recadastre a credencial da OpenAI.
5. No **backend** (Render), aponte `N8N_WEBHOOK_URL` para
   `https://nima-n8n.onrender.com/webhook/nima-analise-adocao`.

> ⚠️ n8n precisa de **disco** (persistência) — o `render.yaml` já usa `plan: starter`
> (o free tier do Render não permite disco). Sem disco, workflows/credenciais somem a
> cada deploy. O backend não quebra se o n8n estiver fora: a análise fica `pendente`.

> Segurança: proteja o editor do n8n em produção — ative
> `N8N_BASIC_AUTH_ACTIVE=true` + `N8N_BASIC_AUTH_USER`/`N8N_BASIC_AUTH_PASSWORD`
> (ou o owner login do n8n) para o painel não ficar público.

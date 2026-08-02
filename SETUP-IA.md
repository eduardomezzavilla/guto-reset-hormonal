# Aba "IA Insights" — backend (Cloudflare Worker)

A aba **IA Insights** manda os dados do funil para um **Cloudflare Worker**, que
chama a IA (Claude `claude-sonnet-5`) e devolve os insights. A **chave da Anthropic**
e a **senha** ficam como *secrets* do Worker — nunca na página pública. A página só
guarda, no seu navegador, a **URL do Worker** e a **senha** que você digita.

> Nada roda sozinho: a IA só é chamada quando você clica em **Gerar insights**.

## Opção A — Painel da Cloudflare (sem instalar nada, ~2 min)

1. Acesse **dash.cloudflare.com** → **Workers & Pages** → **Create** → **Create Worker**.
2. Dê um nome (ex.: `reset-ia-insights`) → **Deploy**.
3. Clique **Edit code**, apague tudo e cole o conteúdo de **`ia-worker/worker.js`** deste repositório → **Deploy**.
4. Vá em **Settings → Variables and Secrets → Add**, tipo **Secret**, e crie os dois:
   - `ANTHROPIC_API_KEY` = sua chave `sk-ant-...`
   - `INSIGHTS_PASSWORD` = a senha que você vai digitar na aba (ex.: `abretesesamo`)
   → **Deploy** de novo para aplicar os secrets.
5. Copie a **URL** do Worker (algo como `https://reset-ia-insights.SEU-SUBDOMINIO.workers.dev`).
6. No dashboard, aba **IA Insights** → **Configurar** → cole a **URL** e a **senha** → **Salvar**.
   Pronto: clique em **Gerar insights**.

## Opção B — Wrangler (linha de comando)

```bash
cd ia-worker
npx wrangler deploy
echo "SUA_CHAVE_ANTHROPIC" | npx wrangler secret put ANTHROPIC_API_KEY
echo "SUA_SENHA"          | npx wrangler secret put INSIGHTS_PASSWORD
```
Depois copie a URL que o deploy imprime e configure na aba IA Insights (passo 6 acima).

## Segurança

- A chave da Anthropic vive **só no Worker** (secret). A senha protege o endpoint
  para ninguém com o link gastar seus tokens — ela não fica no código público, só
  no seu navegador (localStorage) e como secret no Worker.
- Recomendo **rotacionar a chave da Anthropic** depois de configurar tudo.
- Custo: cada clique em **Gerar insights** = 1 chamada à API (poucos centavos).

# CLAUDE.md — Template de Dashboard de Captura de Leads

> Este arquivo é lido automaticamente pelo Claude Code ao abrir o repositório.
> É um **TEMPLATE**: NÃO contém dados de nenhum cliente. Preencha os `<<PREENCHER>>`
> seguindo o checklist abaixo para gerar a dashboard de um cliente novo.

---

## ✅ CHECKLIST DE NOVO CLIENTE (faça nesta ordem)

1. **Repositório** — crie um repo (privado) para o cliente e habilite Pages
   (Settings → Pages → Source: **GitHub Actions**; o workflow habilita sozinho na 1ª execução).
2. **`build/build.py` → identificadores da planilha**
   - `SPREADSHEET_ID` — ID na URL da planilha (entre `/d/` e `/edit`).
   - `GID_LEADS` — número após `gid=` na URL da aba de Leads.
   - `GID_META` — idem para a aba de Meta Ads.
3. **`build/build.py` → imposto**
   - `TAX_FACTOR` — ex.: `1.13806` (+13,806%); use `1.0` se não houver imposto.
4. **`build/build.py` → critério de MQL** (o que é um lead qualificado)
   - Ajuste `MQL_MIN_MIL` **ou** reescreva `is_qualified()` para o critério real
     (faixa de faturamento, leadscore "A", coluna "QLF", resposta de formulário…).
5. **`build/build.py` → mapa de colunas** (`header_index` da aba de Leads)
   - Confira/ajuste os aliases e o fallback posicional, principalmente
     `profession` e `faturamento` (são perguntas do formulário; o nome varia por cliente).
6. **`build/build.py` → rótulos da interface**
   - `CLIENT_NAME`, `CLIENT_SUB`, `MQL_LABEL`, `TAX_LABEL`, `QUAL_DESC`, `BUCKET_ORDER`.
7. **`CLAUDE.md` e `README.md`** — preencha nome do projeto, URL pública, tabela de
   abas/colunas e convenções de campanha do cliente (`<<PREENCHER>>`).
8. **Teste local** — `python build/build.py --leads-file leads.csv --meta-file meta.csv --out dist/index.html`
   e confira as 2 páginas, tema claro/escuro, filtros e multi-seleção (Ctrl).
9. **Publicar** — commit → `main`; o GitHub Actions builda e publica no Pages.
10. **Automatizar** — configure o `cron-job.org` (ver `SETUP-CRON.md`) com um token
    do cliente (fine-grained, só **Actions: read/write** no repo dele).

> Terminou o checklist quando **nenhum `<<PREENCHER>>`** restar:
> `grep -rn "PREENCHER" .` deve não retornar nada.

---

## O que é

Dashboard de **Captura de Leads** — app de BI estático (HTML/CSS/JS + Chart.js via
CDN) publicado no **GitHub Pages**, que cruza a lista de **Leads** com o gerenciador
**Meta Ads** e se atualiza a cada ~30 min (build na nuvem via GitHub Actions,
disparado pelo cron-job.org). **Somente leitura** das planilhas.

- **URL pública:** `<<PREENCHER: https://<owner>.github.io/<repo>/>>`
- **Cliente/projeto:** `<<PREENCHER: nome do cliente>>`

## Fontes de dados (Google Sheets)

Spreadsheet ID: `<<PREENCHER: ID da planilha>>` (público — leitura via export CSV).

| Aba | gid | Colunas usadas |
|-----|-----|----------------|
| **Leads** | `<<PREENCHER: gid Leads>>` | `<<PREENCHER: mapa de colunas — id · created_time · ad_name · adset_name · campaign_name · is_organic · platform · profissão · faturamento · nome · email · phone>>` |
| **Meta Ads** | `<<PREENCHER: gid Meta Ads>>` | Day · Campaign Name · Ad Set Name · Ad Name · Amount Spent · Impressions · Link Clicks · Leads |

URL de export CSV: `https://docs.google.com/spreadsheets/d/<ID>/export?format=csv&gid=<GID>`

### Regra de Lead Qualificado (MQL)
`<<PREENCHER: descreva o critério, ex.: faturamento médio mensal ≥ 30 mil (coluna N)>>`
(lógica em `build.py` → `is_qualified` / `MQL_MIN_MIL`).

### Imposto Meta Ads
Toggle ON aplica `<<PREENCHER: ex.: ×1,13806 (+13,806%)>>` sobre custos do Meta.
Constante `TAX_FACTOR` em `build.py`.

### Convenções de campanha do cliente
`<<PREENCHER: ex.: E2-CAP = Captura, E6-VEN = Vendas; Campaign=utm_campaign, Ad Set=utm_medium, Ad Name=utm_content>>`

## Arquitetura / arquivos

```
build/build.py        # lê os 2 CSVs (read-only), emite REGISTROS BRUTOS (leads[]/meta[]) no HTML
build/template.html   # o app inteiro: CSS + JS (ENGINE — não muda entre clientes)
.github/workflows/deploy.yml  # roda build.py e publica no Pages (workflow_dispatch + schedule + push)
dist/index.html       # saída gerada (gitignored; o Actions reconstrói)
GUIA-REPLICACAO.md    # engine explicada + como adaptar + solução dos problemas de publicação
SETUP-CRON.md         # valores do cron-job.org (marcadores)
```

O `build.py` **não agrega**: exporta as linhas cruas e toda a lógica (filtros, KPIs,
tabelas, gráficos, heatmap, imposto, tema) roda no navegador. A **engine**
(`template.html`) é igual para todos os clientes; só o `build.py` (config) e os
textos de contexto mudam.

## Como estender (outros cruzamentos/funis)

A engine (tabela, filtro cruzado, gráficos, publicação) serve para qualquer funil.
Para um tipo diferente (vendas, tráfego, high-ticket) ou mais fontes (agendamentos,
check-ins, compradores): adicione novos arrays no `build.py` e declare os novos
KPIs/etapas/dimensões — **sem** reescrever o motor. Ver `GUIA-REPLICACAO.md` §9.

## Publicação — problemas conhecidos e soluções

1. **Push com integração somente‑leitura:** se `git push`/MCP derem `403 Resource not
   accessible by integration`, faça push com o **PAT do usuário** direto ao github.com
   (`git push https://x-access-token:<TOKEN>@github.com/<owner>/<repo>.git main:main`).
   **Nunca** grave o token no `.git/config` (use a URL efêmera).
2. **cron-job.org só funciona na `main`:** `workflow_dispatch` só existe na branch padrão.
3. **Pages liga sozinho:** `actions/configure-pages@v5` com `enablement: true`
   (+ `permissions: {pages: write, id-token: write}`).
4. **Proxy do sandbox:** o agente NÃO alcança `docs.google.com`, `*.github.io` nem a API
   REST de Actions/Pages — mas o runner do Actions alcança tudo. Teste dados com CSV local.
5. **Token exposto no chat:** revogar e gerar um novo (fine‑grained, só Actions: r/w no repo).

# CLAUDE.md — Dashboard de Controle de Tráfego Pago (Reset Hormonal · VSL)

> Este arquivo é lido automaticamente pelo Claude Code ao abrir o repositório.
> Dashboard de um funil **VSL / tráfego direto** (Meta Ads × Compradores).
> Nasceu do `dash-template` (originalmente Captura de Leads) e foi adaptado para VSL.

---

## O que é

Dashboard de **Controle de Tráfego Pago** — app de BI estático (HTML/CSS/JS + Chart.js
via CDN) publicado no **GitHub Pages**, que cruza o gerenciador **Meta Ads** com a lista
de **Compradores** e se atualiza a cada ~30 min (build na nuvem via GitHub Actions,
disparado pelo cron-job.org). **Somente leitura** das planilhas.

- **URL pública:** `https://eduardomezzavilla.github.io/guto-reset-hormonal/`
- **Cliente/projeto:** Reset Hormonal (Guto Galamba) — funil VSL
- **Tipo de funil:** VSL (não há etapa de Leads/MQL)

## Fontes de dados (Google Sheets)

Spreadsheet ID: `1SmqXOiITq97O1gtYvUfIbAAjXP3dOM50UNZ_4VepypU` (leitura via export CSV).

| Aba | gid | Colunas usadas |
|-----|-----|----------------|
| **Meta Ads** | `1195145852` | Day · Campaign Name · Ad Set Name · Ad Name · Amount Spent · Impressions · Link Clicks · Landing Page Views · Checkouts Initiated |
| **Compradores** | `1836439885` | Data de Criação · Cliente / Nome · Cliente / E-mail · Produto · Valor da Venda · UTM Content · UTM Campaign · UTM Medium · Status |

URL de export CSV: `https://docs.google.com/spreadsheets/d/<ID>/export?format=csv&gid=<GID>`

### Métricas do funil VSL (`build.py` + `template.html`)
`Gasto → Impressões → Cliques → Page Views → Checkouts → Vendas → Faturamento`

Gasto · Impressões · CPM · Cliques · CPC · CTR · Page Views · CPV · CR (Cliques/PageViews) ·
Checkouts · CPIC · VisCHK (Checkouts/PageViews) · Vendas · CAC (Gasto/Vendas) ·
ConvCHK (Vendas/Checkouts) · Faturamento · ROAS (Faturamento/Gasto) · Ticket (Faturamento/Vendas).

### Produto principal / atribuição
- **Produto principal** = `MAIN_PRODUCT_PREFIX = "protocolo reset hormonal"` (inclui a
  variante "- 35%"). Base de **Vendas / CAC / ConvCHK / Ticket**.
- **Faturamento / ROAS** = soma de **todos os produtos** do funil (orderbumps/upsells).
- Uma venda entra no funil se: é o produto principal **OU** seu `UTM Content` casa com
  um `Ad Name` do Meta (captura orderbumps/upsells que carregam a UTM do anúncio). Vendas
  de outros funis (UTM/produto não relacionados) ficam de fora. Só conta status pago.
- **Sem coluna de Receita** → não há Receita/ROAS R/Ticket R (por decisão do cliente).

### Imposto Meta Ads
Toggle ON aplica **×1,13806 (+13,806%)** sobre os custos do Meta. Constante `TAX_FACTOR`.

### Convenções de campanha
`Campaign Name = utm_campaign`, `Ad Set Name = utm_medium`, `Ad Name = utm_content`.
As vendas são atribuídas à campanha/conjunto pelo `Ad Name` correspondente no Meta.

## Arquitetura / arquivos

```
build/build.py        # lê os 2 CSVs (read-only), emite REGISTROS BRUTOS (meta[]/sales[]) no HTML
build/template.html   # o app inteiro: CSS + JS (ENGINE)
.github/workflows/deploy.yml  # roda build.py e publica no Pages (workflow_dispatch + schedule + push)
dist/index.html       # saída gerada (gitignored; o Actions reconstrói)
GUIA-REPLICACAO.md    # engine explicada + solução dos problemas de publicação
SETUP-CRON.md         # valores do cron-job.org
```

O `build.py` **não agrega**: exporta as linhas cruas e toda a lógica (filtros, KPIs,
tabelas, gráficos, heatmap, imposto, tema) roda no navegador.

Teste local:
`python build/build.py --meta-file meta.csv --sales-file sales.csv --out dist/index.html`

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

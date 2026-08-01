# Template — Dashboard de Captura de Leads (Meta Ads × Leads)

Modelo reutilizável de dashboard de BI estática (HTML/CSS/JS + Chart.js) publicada
no **GitHub Pages**, atualizada a cada ~30 min (GitHub Actions + cron-job.org),
**somente leitura** das planilhas.

> **Este é um TEMPLATE.** Não contém dados de cliente. Para usar, siga o
> **CHECKLIST DE NOVO CLIENTE** no topo de `CLAUDE.md` e preencha os `<<PREENCHER>>`
> (o build recusa rodar enquanto o `SPREADSHEET_ID` não for preenchido).

## Como usar (resumo)

1. Preencha `build/build.py` (IDs da planilha, gids, imposto, critério de MQL, mapa de
   colunas e rótulos). Detalhes no checklist do `CLAUDE.md`.
2. Teste local:
   ```bash
   python build/build.py --leads-file leads.csv --meta-file meta.csv --out dist/index.html
   ```
3. Commit → `main`. O GitHub Actions builda e publica no Pages.
4. Configure o `cron-job.org` (ver `SETUP-CRON.md`).

**URL pública:** `<<PREENCHER: https://<owner>.github.io/<repo>/>>`

## O que a dashboard mostra

- **Aba 1 — Visão Geral de Leads:** KPIs (Gasto, Leads, CPL, MQLs, CPMQL, Tx‑MQL…),
  gráfico combinado diário, barras por origem/faixa/plataforma/profissão e tabela
  diária com heatmap (todos os leads).
- **Aba 2 — Captura Meta Ads:** funil em etapas, combinado diário, barras por
  utm_content, tabela diária com heatmap (só Meta) e 3 tabelas hierárquicas
  (Campanha → Conjunto → Anúncio) com gráfico de linha e **filtro cruzado**.

Recursos: filtro global de data + presets, toggle de imposto, tema claro/escuro,
tabelas com ordenação/redimensionamento/multi-seleção, cache-bust.

## Arquivos

- `build/template.html` — a **engine** (CSS + JS); igual para todos os clientes.
- `build/build.py` — leitura das planilhas + config do cliente.
- `.github/workflows/deploy.yml` — build + deploy no Pages.
- `GUIA-REPLICACAO.md` — arquitetura, CSS/JS, lógica de tabelas/gráficos e
  solução dos problemas de publicação.
- `CLAUDE.md` — contexto + **checklist de novo cliente**.
- `SETUP-CRON.md` — configuração do cron-job.org.

## Privacidade

E‑mail e telefone são **mascarados** no build (a página é pública). Para exibir
contatos completos, use repositório/Pages **privado**.

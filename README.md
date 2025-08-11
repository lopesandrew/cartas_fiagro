# Cartas Mensais FIAGRO (Brasil)

Projeto em Python para **baixar automaticamente** cartas mensais (PDFs) de FIAGROs no Brasil, extrair metadados (mês/ano, gestora, ticker) e manter um índice em CSV/Parquet. Inclui agendamento via GitHub Actions.

## Visão geral
- **Entrada**: lista de páginas/alvos (`sites/catalog.yaml`) onde as gestoras publicam cartas.
- **Coleta**: crawler genérico procura links de PDF com heurísticas para "carta", "mensal", "relatório", etc.
- **Download**: salva PDFs em `data/raw/<ticker>/<yyyy-mm>-<slug>.pdf`.
- **Metadados**: extrai texto, detecta mês/ano, gestora e ticker. Salva em `data/index.csv`.
- **Agendamento**: workflow em `.github/workflows/crawl.yml` roda mensalmente e sob demanda.

> ⚠️ O ecossistema de sites é heterogêneo. Começamos com um **crawler genérico** + catálogo declarativo. Para sites difíceis (JS, iframes, portais), crie um "plugin" específico.

## Como usar
1. **Clone** este repositório e crie um ambiente:
   ```bash
   python -m venv .venv && source .venv/bin/activate  # (Windows: .venv\Scripts\activate)
   pip install -r requirements.txt
   playwright install  # necessário para headless browser
   ```

2. **Edite** `sites/catalog.yaml`: adicione/ajuste páginas de cartas por gestora/ticker.
3. **Rodar o crawler**:
   ```bash
   python -m fiagro_letters crawl --site all
   ```
4. **Atualizar índice** (metadados a partir dos PDFs baixados):
   ```bash
   python -m fiagro_letters index
   ```

### Dicas
- Comece com poucos alvos e valide o resultado em `data/index.csv`.
- Se um site não listar PDFs diretamente, crie um plugin em `fiagro_letters/plugins/`.
- Use `--since 2024-01` para limitar período, ou `--tickers RZTR11,XPCI11` para filtrar.

## Estrutura
```
fiagro_letters/          # pacote principal
  __init__.py
  cli.py                 # interface de linha de comando (Typer)
  crawler.py             # coletor genérico com httpx + Playwright (fallback)
  extract.py             # extração de texto e metadados
  utils.py
  plugins/               # spiders específicas (opcional)
sites/
  catalog.yaml           # catálogo declarativo de alvos (gestora/ticker/url)
data/
  raw/                   # PDFs baixados
  index.csv              # índice consolidado
.github/workflows/
  crawl.yml              # agendamento (cron + manual)
```

## Roadmap rápido
- [ ] Adicionar mais FIAGROs ao `catalog.yaml`
- [ ] Implementar deduplicação por hash e URL canônica
- [ ] Suporte a RSS/sitemaps quando disponíveis
- [ ] Exportar `data/index.parquet` e HTML leve

## Licença
MIT

# Hackathon Big Data - Forecast de Demanda

Projeto em Python usado no hackathon para prever a demanda das semanas 1..5 a partir do historico semanal de 2022. Ha dois fluxos principais:
- `forecast/`: pipeline completo com Transformer (treino, validacao e geracao de submissao).
- `eda_wmape_drivers.py`: EDA numerica com drivers de WMAPE e graficos salvos em `eda_out/`.
- `src/`: funcoes de apoio (carregamento, features, metricas e baselines sazonais).

## Preparacao rapida
1) Python 3.10+ (testado com 3.12) e pip instalados.
2) Ambiente virtual (opcional, recomendado):
   ```bash
   python -m venv .venv
   .venv\\Scripts\\activate  # Windows
   ```
3) Dependencias:
   ```bash
   pip install -r requirements.txt
   ```
   Para GPU, instale o `torch` correspondente a sua placa seguindo o guia oficial da PyTorch.

## Estrutura de dados esperada
- Transacoes 2022: colunas obrigatorias `data`, `pdv`, `produto`, `quantidade` (ou equivalentes; o script normaliza nomes como `transaction_date`, `sku`, etc).
- Catalogos opcionais: `cadastroprodutos` (`produto`) e `pdvs` (`pdv`) para enriquecer logs e EDA.
- Coloque os arquivos brutos em `data/` (nao sao versionados).

## Como rodar o modelo Transformer
Gera a submissao e metadados (WMAPE de validacao + hiperparametros):
```bash
python -m forecast.transformer_forecast ^
  --transactions data/transacacoes.csv ^
  --products data/cadastroprodutos.csv ^
  --stores data/pdvs.csv ^
  --out outputs/submission_transformer.csv ^
  --use_returns_feature ^
  --only_jan_pairs ^
  --max_pairs 200000 ^
  --cap_mult_pair_q90 2.0 ^
  --save_ckpt_dir checkpoints
```
- Ajuste `--context_len`, `--d_model`, `--epochs`, etc. conforme o hardware.
- Para retomar treino: `--resume checkpoints/epoch2.pt`.
- Auto-tune rapido: adicione `--auto_tune --tune_trials 5 --tune_epochs 2 --tune_max_train_batches 500`.
- Logs e validacao sao exibidos no console; a submissao final fica em `outputs/` e o resumo em `outputs/*.meta.json`.

## EDA focada em WMAPE
1) Edite no topo de `eda_wmape_drivers.py` os caminhos para `TRANSACTIONS_PATH`, `PRODUCTS_PATH`, `STORES_PATH`.
2) Rode:
   ```bash
   python eda_wmape_drivers.py
   ```
Gera CSVs, PNGs e `console_report.txt` em `eda_out/` com achados por par pdv-produto.

## Baselines sazonais e utilitarios
- `src/baselines.py`: combinacoes de medias semanais/anuais com caps (usado como blend ou baseline).
- `src/features.py` e `src/data_loader.py`: normalizacao de colunas, calendario ISO e agregacao semanal.
- `src/metrics.py`: WMAPE em pandas/numpy.
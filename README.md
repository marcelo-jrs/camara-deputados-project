# Mapa Ideológico — Câmara dos Deputados (2025)

Mapa 2D de deputados federais com base nas votações nominais de **Plenário** em **2025**, usando dados abertos da Câmara dos Deputados.

## O que faz

- Posiciona cada deputado num espaço 2D (PCA) a partir de como votaram
- Calcula coesão partidária e identifica "rebeldes"
- Agrupa deputados em blocos ideológicos (k-means)

## Fonte de dados

[Dados Abertos da Câmara dos Deputados](https://dadosabertos.camara.leg.br/swagger/api.html) — arquivos em massa (CSV).

## Estrutura

```
data/raw/           -> CSVs brutos da Câmara
data/processed/     -> Parquets tratados (saída do ETL)
notebooks/
  01_etl.ipynb      -> Limpeza e transformação dos dados
  02_eda.ipynb      -> Análise exploratória, coesão e rebeldes
  03_ideology.ipynb -> Mapa ideológico e clusters
```

## Como rodar

```bash
pip install -r requirements.txt
```

Executar os notebooks em ordem (01, 02, 03).

## Dados processados

| Arquivo | Conteúdo |
|---|---|
| `deputados.parquet` | Cadastro dos deputados (id, nome, partido, UF) |
| `votacoes.parquet` | Votações de Plenário (id, data, resultado) |
| `votos.parquet` | Voto individual de cada deputado em cada votação |
| `orientacoes.parquet` | Orientação das bancadas/partidos por votação |

## Status

- [x] ETL
- [ ] EDA (coesão + rebeldes)
- [ ] Mapa ideológico + clusters
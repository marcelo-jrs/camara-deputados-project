# Mapa Ideológico — Câmara dos Deputados (2025)

Mapa 2D de deputados federais com base nas votações nominais de **Plenário** em **2025**, usando dados abertos da Câmara dos Deputados.

## O que faz

- **ETL**: coleta e limpa dados brutos de votações, deputados e orientações partidárias, filtrando apenas votações de Plenário
- **EDA**: analisa distribuições de votos, calcula coesão partidária e identifica deputados "rebeldes" (que mais divergem da maioria do seu partido)
- **Mapa ideológico**: posiciona cada deputado num espaço 2D via PCA e agrupa em blocos ideológicos com K-Means

## Fonte de dados

[Dados Abertos da Câmara dos Deputados](https://dadosabertos.camara.leg.br/swagger/api.html) — arquivos em massa (CSV com separador `;`).

Arquivos utilizados:
- `deputados.csv` — cadastro de todos os deputados (sem coluna de ID — extraído da URI)
- `votacoes-2025.csv` — metadados de votações (data, órgão, resultado, proposição)
- `votacoesVotos-2025.csv` — voto individual de cada deputado (inclui partido e UF no momento do voto)
- `votacoesOrientacoes-2025.csv` — orientação de bancada/partido por votação

## Estrutura

```
data/raw/           -> CSVs brutos da Câmara (separador ;)
data/processed/     -> Parquets tratados (saída do ETL)
notebooks/
  01_etl.ipynb      -> Limpeza e transformação dos dados
  02_eda.ipynb      -> Análise exploratória, coesão e rebeldes
  03_pca.ipynb      -> Mapa ideológico e clusters
```

## Como rodar

```bash
pip install -r requirements.txt
```

Executar os notebooks em ordem (01 → 02 → 03).

## Dados processados

| Arquivo | Conteúdo | Filtros aplicados |
|---|---|---|
| `deputados.parquet` | Cadastro dos deputados (id, nome, partido, UF) | Legislatura 57 + presentes em votações de Plenário |
| `votacoes.parquet` | Votações de Plenário (id, data, resultado) | `siglaOrgao == 'PLEN'` e `votosSim + votosNao > 0` |
| `votos.parquet` | Voto individual de cada deputado em cada votação | Apenas votações de Plenário, deduplicado por (votação, deputado) |
| `orientacoes.parquet` | Orientação das bancadas/partidos por votação | Apenas votações de Plenário |

## Números-chave (2025)

- **429** votações de Plenário (fev–dez 2025)
- **551** deputados votantes (513 titulares + 38 suplentes)
- **171.217** registros de voto individual
- Tipos de voto: Sim (99.870), Não (69.794), Artigo 17 (420), Obstrução (375), Abstenção (337)

## Metodologia

### ETL (`01_etl.ipynb`)
1. Carrega 4 CSVs brutos com separador `;`
2. Extrai `deputado_id` da URI (último segmento)
3. Filtra votações para Plenário (`siglaOrgao == 'PLEN'`) e remove votações sem votos Sim/Não
4. Filtra votos e orientações pelos IDs de votações de Plenário
5. Enriquece tabela de deputados com partido e UF do voto mais recente
6. Valida integridade (sem duplicatas, chaves consistentes) e salva em Parquet

### EDA (`02_eda.ipynb`)
1. **Distribuições básicas**: tipos de voto, participação por deputado, quórum por votação
2. **Coesão partidária**: para cada votação, determina a maioria do partido (Sim vs Não); calcula taxa de concordância de cada deputado com seu partido; agrega por partido (partidos com ≥ 5 deputados)
3. **Rebeldes**: ranking dos deputados com menor concordância com a maioria do partido (mínimo 50 votos Sim/Não)

### Mapa ideológico (`03_pca.ipynb`)
1. **Codificação**: Sim → +1, Não → −1, Abstenção/Obstrução → 0, Artigo 17/ausência → NaN (preenchido com 0)
2. **Matriz deputado × votação**: pivô dos votos codificados; filtra deputados com < 50 votos
3. **PCA**: centraliza a matriz (subtrai média de cada coluna) e projeta em 2 componentes principais
4. **Mapa**: scatter plot PC1 × PC2, colorido por partido — PC1 separa esquerda/direita
5. **K-Means**: agrupa deputados em k=3 clusters (governo, oposição, centrão) e compara com partidos
6. **Variações**: testa k=4 e k=5 para verificar sub-estruturas

## Status

- [x] ETL — dados limpos em Parquet
- [x] EDA — coesão partidária + ranking de rebeldes
- [x] Mapa ideológico + clusters (PCA + K-Means)
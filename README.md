# MVP Engenharia de Dados — PUC-Rio

Pipeline de dados em Databricks para analisar candidaturas a Deputado Estadual no estado de São Paulo entre as eleições de 2022 e 2026.

## 1. Contexto e Objetivo

O MVP organiza dados públicos do Tribunal Superior Eleitoral (TSE) em uma arquitetura Bronze, Silver e Gold. O objetivo é disponibilizar uma base analítica confiável para comparar candidaturas de 2022 e 2026, responder perguntas de perfil e recorrência e documentar a linhagem dos dados no Unity Catalog.

## 2. Perguntas de Negócio

- Como variou a distribuição de candidaturas por cor/raça entre 2022 e 2026?
- Como variou a distribuição de candidaturas por gênero entre os dois anos?
- Quantos candidatos estiveram presentes nas duas eleições?
- Entre os candidatos presentes nos dois anos, quantos mudaram de sigla partidária?

## 3. Fonte e Coleta dos Dados

A fonte é o TSE, a partir dos dados de candidaturas e dos arquivos complementares utilizados no pipeline. O recorte considera apenas candidaturas ao cargo de Deputado Estadual no estado de São Paulo, para 2022 e 2026.

Os arquivos brutos não são versionados neste repositório. Também não são publicados CPFs individuais: eles são usados apenas internamente como identificador técnico para o cruzamento entre eleições.

## 4. Arquitetura da Solução

```text
Dados TSE
   ↓
Bronze — ingestão e preservação das fontes
   ↓
Silver — padronização, filtros e enriquecimento
   ↓
Gold — tabelas analíticas e comparativas
   ↓
Dashboard — análises visuais

Unity Catalog — Data Catalog e linhagem
```

## 5. Modelagem e Data Catalog

As tabelas principais foram documentadas no Unity Catalog com descrição, origem, granularidade, regras e comentários de colunas.

| Camada | Tabela | Finalidade |
|---|---|---|
| Silver | `silver_candidatos_deputado_estadual_sp` | Base consolidada de candidaturas, com 3.489 registros e 32 colunas. |
| Gold | `gold_comparacao_candidatos_2022_2026` | Comparação entre anos por CPF válido distinto, com 3.196 registros e 20 colunas. |
| Gold | `gold_distribuicao_raca` | Distribuição de candidaturas por cor/raça. |
| Gold | `gold_distribuicao_genero` | Distribuição de candidaturas por gênero. |

![Data Catalog da tabela Silver](docs/prints/03_data_catalog_silver.png)

![Data Catalog da tabela Gold de comparação](docs/prints/04_data_catalog_gold.png)

## 6. Pipeline de Dados

Os notebooks do pipeline estão em [`notebooks/`](notebooks/), numerados conforme a sequência de execução.

### 6.1 Bronze

A Bronze recebe os dados de candidaturas e os dados complementares do TSE, preservando a origem por ano eleitoral.

### 6.2 Silver

A Silver consolida os dados de 2022 e 2026, filtra o recorte de Deputado Estadual em São Paulo, padroniza tipos e datas, enriquece os registros com a fonte complementar e cria o indicador `cpf_valido_para_cruzamento`.

### 6.3 Gold

A Gold materializa as tabelas para análise: comparação de candidatos entre anos, distribuição por cor/raça e distribuição por gênero.

## 7. Qualidade dos Dados

Os controles aplicados incluem:

- validação do recorte de cargo, UF e ano eleitoral;
- padronização de datas e tipos entre as fontes;
- controle de registros e colunas nas tabelas Gold;
- deduplicação lógica para comparação de candidatos por CPF válido;
- preservação de categorias com quantidade zero, sem removê-las da análise;
- proteção de dados pessoais: CPFs não são publicados no repositório.

A validação final confirmou 3.196 registros na tabela de comparação, 7 categorias em `gold_distribuicao_raca` e 3 categorias em `gold_distribuicao_genero`.

![Validação final da camada Gold](docs/prints/01_validacao_final_gold.png)

## 8. Análise e Resultados

A comparação utiliza CPF válido distinto. Foram identificados 2.053 candidatos em 2022, 1.430 em 2026 e 3.196 pessoas distintas no universo combinado.

### 8.1 Cor/Raça

| Categoria | 2022 | 2026 | Variação |
|---|---:|---:|---:|
| Branca | 64,30% | 66,85% | +2,55 p.p. |
| Preta | 13,31% | 11,89% | -1,42 p.p. |
| Parda | 21,03% | 19,93% | -1,10 p.p. |
| Indígena | 0,29% | 0,56% | +0,27 p.p. |

### 8.2 Gênero

A composição por gênero foi estável no período: candidaturas femininas passaram de 32,83% para 32,80%, enquanto candidaturas masculinas passaram de 67,07% para 67,20%.

### 8.3 Recorrência de candidatos

| Situação | Quantidade |
|---|---:|
| Somente 2022 | 1.766 |
| Ambos os anos | 287 |
| Somente 2026 | 1.143 |

### 8.4 Mudança de partido

Entre os 287 candidatos presentes nos dois anos, 153 (53,31%) mudaram de sigla partidária e 134 (46,69%) permaneceram na mesma sigla.

## 9. Dashboard

O dashboard consolida recorrência de candidatos, mudança de partido e distribuições de cor/raça e gênero.

![Dashboard final](docs/prints/02_dashboard_final.png)

## 10. Linhagem dos Dados

A linhagem documentada no Unity Catalog evidencia o fluxo da Silver para a Gold de comparação e o consumo analítico posterior.

![Linhagem da Silver para a Gold](docs/prints/05_lineage_gold.png)

## 11. Limitações

- Os dados representam o recorte e o momento de coleta definidos no MVP; dados de candidatura podem ser atualizados pelo TSE.
- A análise trata candidaturas, não resultados eleitorais.
- O cruzamento entre eleições é limitado aos CPFs considerados válidos pelo pipeline.
- Dados brutos e identificadores pessoais não são disponibilizados publicamente.

## 12. Autoavaliação

O MVP entrega ingestão, tratamento, camada analítica, controles de qualidade, Data Catalog, linhagem e dashboard. Como evolução, o projeto pode incorporar execução agendada, testes automatizados e atualização periódica das fontes.

## 13. Estrutura do Repositório

```text
mvp-engenharia-de-dados-puc-rio-eleicoes-sp/
├── README.md
├── notebooks/
│   ├── 01_Exploracao_Dados_TSE.ipynb
│   ├── 02_Camada_Bronze.ipynb
│   ├── 03_Camada_Silver.ipynb
│   ├── 04_Camada_Gold.ipynb
│   ├── 05_Dashboard_Analise.ipynb
│   └── 06_Data_Catalog_Documentacao.ipynb
├── docs/
│   └── prints/
│       ├── 01_validacao_final_gold.png
│       ├── 02_dashboard_final.png
│       ├── 03_data_catalog_silver.png
│       ├── 04_data_catalog_gold.png
│       └── 05_lineage_gold.png
└── data/
    └── README.md
```

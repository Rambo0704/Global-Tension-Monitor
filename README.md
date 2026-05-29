# 🌍 Global Tension Monitor (GTM)

[![Python 3.12](https://img.shields.io/badge/Python-3.12-blue.svg)](https://www.python.org/)
[![XGBoost](https://img.shields.io/badge/Model-XGBoost-orange.svg)](https://xgboost.readthedocs.io/)
[![Optuna](https://img.shields.io/badge/Optimization-Optuna-brightgreen.svg)](https://optuna.org/)
[![Google BigQuery](https://img.shields.io/badge/Data-BigQuery-blueviolet.svg)](https://cloud.google.com/bigquery)

Bem-vindo ao **Global Tension Monitor**, um projeto de Ciência de Dados e Machine Learning focado no monitoramento e previsão de tensões e crises geopolíticas globais.

Este sistema utiliza as ricas bases de dados de eventos do **Projeto GDELT** para rastrear, processar e prever níveis de crise por país ao longo do tempo.

---

## 📋 Resumo do Projeto
O objetivo do GTM é classificar a intensidade de tensões geopolíticas (`target_crise` classificado em 3 níveis: 0, 1 e 2). O projeto usa o volume de menções, o tom da cobertura midiática e a Escala de Goldstein (uma medida de quão cooperativo ou conflituoso é um evento) para alimentar um modelo preditivo baseado em árvores de decisão.

---

## ⚙️ Pipeline do Projeto

A arquitetura do projeto foi estruturada em três grandes etapas, representadas nos notebooks da pasta `notebooks/`:

### 1. Coleta de Dados Cloud (`01_cloud_coleta.ipynb`)
*   **Integração:** Conexão direta com os servidores do Google através do BigQuery.
*   **Fonte de Dados:** Tabela pública de eventos do GDELT v2 (`gdelt-bq.gdeltv2.events`).
*   **Recorte Temporal:** 10 anos de histórico financeiro e de eventos (01/01/2016 a 01/01/2026).
*   **Extração de Variáveis:**
    *   `goldstein`: Impacto potencial do evento no país de origem (Escala de Goldstein).
    *   `tone`: Sentimento médio e tom das notícias e publicações.
    *   `mentions`, `sources`, `articles`: Volume bruto de atenção da mídia (número de menções, fontes e artigos únicos).
*   **Armazenamento:** Exportação eficiente dos dados históricos para o formato `.parquet` (`historico_agrupado_10anos.parquet`).

### 2. Engenharia de Features (`02_features.ipynb`)
Nesta etapa, a base histórica processada é refinad:
*   Construção de indicadores temporais[cite: 2].
*   Criação da variável alvo multicasse (`target_crise` com classes 0, 1 e 2)].
*   Geração da base analítica "padrão ouro" (`gold.parquet`) pronta para alimentar o treinamento do modelo.

### 3. Modelagem e Otimização (`03_model.ipynb`)
A etapa de machine learning foca em construir um modelo preditivo robusto que entenda a cronologia dos eventos:
*   **Split Temporal Estrito:** Para evitar o vazamento de informações futuras para o passado (*data leakage*), os dados foram divididos em ordem cronológica:
    *   **70%** Treinamento
    *   **15%** Validação
    *   **15%** Teste (futuro cego)
*   **Otimização de Hiperparâmetros:** Utilização da biblioteca **Optuna** com o `XGBoostPruningCallback`[cite: 2]. O estudo realizou 100 *trials* buscando maximizar a métrica **F1-Score Macro** do algoritmo `XGBClassifier`.
*   **Performance Final no Teste Cego:**
    *   **Acurácia (Accuracy):** 77%
    *   **F1-Score Macro:** ~71%
    *   O modelo demonstra alta capacidade preditiva, especialmente para a classe majoritária de crise (precisão de 89% na classe 2).
*   **Exportação:** O modelo final treinado é consolidado com os dados de treino + validação e salvo como arquivo serializado (`gtm_xgb_model.json`).

---

## 🛠️ Stack Tecnológico
*   **Dados e Extração:** `pandas`, `pandas-gbq`, `Google BigQuery`
*   **Machine Learning:** `xgboost`, `scikit-learn`
*   **Busca de Hiperparâmetros:** `optuna`
*   **Armazenamento de Dados:** `Apache Parquet`
*   **Visualização:** `matplotlib`, `seaborn`

---

## 📁 Estrutura de Diretórios
Uma visão simplificada da organização adotada no repositório:

```text
Global-Tension-Monitor/
│
├── data/
│   ├── processed/    # Base agrupada extraída da nuvem (.parquet)
│   └── features/     # Base final com a variável target (gold.parquet)
│
├── notebooks/
│   ├── 01_cloud_coleta.ipynb   # Queries e extração do BigQuery
│   ├── 02_features.ipynb       # Feature engineering e criação do target
│   └── 03_model.ipynb          # Treinamento, Optuna e validação do XGBoost
│
├── models/
│   └── gtm_xgb_model.json      # Modelo final pronto para inferência
│
└── diagrams/                   # Arquitetura de software, fluxo e diagramas UML

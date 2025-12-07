# Inteligência de Mercado para Locadoras: Expansão de Frotas e Lead Scoring (SP 2025)

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=for-the-badge&logo=python&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/scikit--learn-%23F7931E.svg?style=for-the-badge&logo=scikit-learn&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-visuals-777BB4?style=for-the-badge&logo=seaborn&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-success?style=for-the-badge)

## 📋 Resumo Executivo

Este repositório contém a implementação técnica de uma solução de **Data Science & Strategic Analytics** desenvolvida para otimizar a alocação de ativos no mercado de **Locação de Veículos (Frotas)** em São Paulo.

Utilizando a base censitária de condutores habilitados (Outubro/2025) do Detran-SP, o projeto aplica um pipeline de **Engenharia de Dados** e **Machine Learning** para solucionar o problema de "Cold Start" na expansão de agências. O modelo transforma dados brutos em indicadores de **Volume (Market Density)** e **Propensão (Lead Scoring)**, permitindo que locadoras identifiquem onde instalar novos pátios e qual perfil demográfico priorizar em campanhas de marketing para maximizar o LTV (Lifetime Value) e reduzir a sinistralidade.

---

## 📂 Estrutura do Repositório

O projeto foi arquitetado em módulos independentes para garantir a integridade do processamento e a reprodutibilidade da análise visual.

| Arquivo | Função Técnica | Descrição |
| :--- | :--- | :--- |
| `Fase_2_BI.ipynb` | **Backend (Engenharia)** | Pipeline de ingestão, limpeza (ETL), engenharia de atributos e treinamento do modelo *Random Forest*. Gera o CSV final. |
| `data_visualization.ipynb` | **Frontend (Analytics)** | Notebook de apresentação. Consome o CSV processado e gera os gráficos executivos (Matriz de Calor, Rankings e Dashboard Interativo). |
| `baseBICNH_dashboard_2025_fase2.csv` | **Dataset Processado** | Base de dados tratada e enriquecida com os Scores Preditivos, pronta para consumo analítico. |

---

## ⚠️ Nota sobre o Dashboard Interativo

No final do notebook de visualização (`data_visualization.ipynb`), existe uma ferramenta de **Data Mining Interativa** (Menu de Cidades) desenvolvida com a biblioteca `ipywidgets`.

> **Atenção:** Como o GitHub renderiza notebooks como páginas estáticas, **os menus suspensos e botões não funcionam na pré-visualização do navegador**, podendo aparecer como código vazio ou mensagem de erro de renderização.

Para utilizar o filtro dinâmico:
1.  Baixe o notebook ou abra-o no **Google Colab**.
2.  Execute todas as células (`Runtime > Run All`).
3.  Vá até a última seção ("Análise Tática por Cidade") para interagir com o menu e filtrar os dados por município.

---

## ⚙️ Arquitetura e Metodologia

A solução foi desenvolvida sob uma arquitetura **Serverless em Nuvem**, utilizando Python para todo o ciclo de vida do dado, eliminando dependências de ferramentas proprietárias de BI.

### Pipeline de Dados (Fluxo de Execução)

O pipeline executa as seguintes etapas sequenciais:

#### 1. Ingestão e Tratamento (ETL)
* **Filtros de Negócio:** Aplicação de regex para isolar condutores da **Categoria B** (aptos a dirigir automóveis), descartando categorias exclusivas de moto ou carga pesada, alinhando a análise ao portfólio das locadoras.
* **Limpeza:** Exclusão de registros com flag `condutor_bloqueado = 'S'`, garantindo que a análise reflita apenas a frota ativa.

#### 2. Engenharia de Atributos (Feature Engineering)
Transformação dos dados transacionais em uma **Tabela Mestra de Perfis**.
* **Agrupamento:** Clusterização por `Município`, `Faixa Etária` e `Gênero`.
* **Cálculo de KPI:** Geração da **Taxa de Adesão Regional (KPI 1)** através da pivotagem de dados.

#### 3. Modelagem Preditiva (Lead Scoring)
Implementação de algoritmo supervisionado para cálculo do **Score de Propensão (KPI 3)**.
* **Algoritmo:** **Random Forest Regressor** (*Scikit-Learn*).
* **Justificativa:** A escolha por Florestas Aleatórias deve-se à sua capacidade de capturar relações não-lineares (ex: o risco e a propensão variam drasticamente entre jovens e adultos) e sua robustez para inferir scores em municípios com baixa amostragem estatística, onde a taxa bruta (KPI 1) poderia ser enganosa.

#### 4. Visualização de Dados (Data Viz)
Geração de gráficos estáticos de alta fidelidade (**Seaborn**) com identidade visual executiva ("Dark/Tech"), focados em responder perguntas de negócio sobre Ociosidade e Custo de Aquisição (CAC).

---

## 📊 Dicionário de Dados (Output Final)

Descrição das variáveis estratégicas contidas no arquivo `baseBICNH_dashboard_2025_fase2.csv`:

| Variável | Tipo | Aplicação no Negócio (Locadoras) |
| :--- | :--- | :--- |
| `Cidade` | String | Filtragem geográfica para expansão de rede física e logística. |
| `Faixa_Etaria` | String | Segmentação para análise de Risco (Jovens vs Adultos) e LTV. |
| `Genero` | String | Segmentação demográfica para personalização de campanhas. |
| `KPI_Volume_Demanda` | Int64 | **(KPI 2 - Volume)** Define o tamanho do pátio/estoque necessário na região (CAPEX). |
| `KPI_Taxa_Conversao` | Float64 | **(KPI 1 - Eficiência)** Diagnóstico histórico. Mede a porcentagem real de adesão em cada micro-região. |
| `KPI_Score_Propensao` | Float64 | **(KPI 3 - Qualidade)** Índice calculado via **Machine Learning**. Corrige distorções estatísticas de municípios pequenos (generalização) e projeta a probabilidade de adesão baseada no padrão do perfil (0.0 a 1.0). |

---

## 🛠️ Requisitos para Reprodução

Para executar o código fonte deste projeto, são necessárias as seguintes bibliotecas Python:

```python
pandas>=1.3.5
numpy>=1.21.6
scikit-learn>=1.0.2
seaborn>=0.11.2
ipywidgets>=7.6.5

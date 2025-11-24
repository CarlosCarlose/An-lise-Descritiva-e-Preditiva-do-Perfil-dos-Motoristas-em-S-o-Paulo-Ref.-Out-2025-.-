# Inteligência de Mercado na Mobilidade Urbana: Análise Descritiva e Preditiva (Censo SP 2025)

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=for-the-badge&logo=python&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/scikit--learn-%23F7931E.svg?style=for-the-badge&logo=scikit-learn&logoColor=white)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![Power BI](https://img.shields.io/badge/power_bi-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Status](https://img.shields.io/badge/Status-Concluído-success?style=for-the-badge)

## Resumo Executivo

Este repositório contém a implementação técnica de uma solução de **Business Intelligence & Analytics** desenvolvida para mitigar a assimetria de informações no mercado de frotas e serviços automotivos no Estado de São Paulo.

Utilizando a base de dados censitária de condutores habilitados (Outubro/2025) do Detran-SP, o projeto aplica um pipeline de **Engenharia de Dados** e **Machine Learning** para transformar dados brutos governamentais em indicadores estratégicos de **Volume (Market Density)** e **Propensão (Predictive Scoring)**, orientando decisões de investimento em ativos e infraestrutura.

---

## Acesso aos Artefatos

| Componente | Link de Acesso |
| :--- | :--- |
| **📂 Fonte de Dados Primária** | [Dados Abertos Detran-SP]([https://dadosabertos.sp.gov.br/dataset/emissao-de-carteiras-nacionais-de-habilitacao-cnh]) |
| **📊 Dashboard Interativo** | [Acessar Drive com o arquivo do Power BI ]([https://drive.google.com/drive/folders/1eWn0TQrnvIIyfdFF6NJhRvtwLBy1sdU0?usp=drive_link]) |

---

## Arquitetura e Metodologia

A solução foi desenvolvida sob uma arquitetura híbrida: processamento em nuvem (*Serverless*) utilizando Python para o tratamento pesado e modelagem estatística, integrada a uma camada de visualização local (Power BI).

### Pipeline de Dados (Fluxo de Execução)

O script `Fase_2_BI.ipynb` executa as seguintes etapas sequenciais:

#### 1. Ingestão e Tratamento Inicial (Data Ingestion & Cleaning)
* **Leitura:** Carregamento de dados em formato CSV utilizando codificação `latin1` e separadores customizados para garantir a integridade dos caracteres.
* **Filtros de Negócio:**
    * *Categoria:* Aplicação de regex (`str.contains('B')`) para isolar condutores aptos a dirigir automóveis (incluindo categorias mistas como AB, AC), descartando categorias exclusivas de moto ou carga pesada.
    * *Status:* Exclusão de registros com flag `condutor_bloqueado = 'S'`, garantindo que a análise reflita apenas a força de trabalho ativa.
* **Tipagem:** Conversão forçada (`pd.to_numeric`) da coluna de quantidade para garantir consistência matemática.

#### 2. Engenharia de Atributos e Agregação (Feature Engineering)
Para viabilizar a análise estatística e de negócio, os dados transacionais foram transformados em uma **Tabela Mestra de Perfis**.
* **Granularidade:** Os dados foram agrupados (*Groupby*) pelas dimensões `Município`, `Faixa Etária` e `Gênero`.
* **Pivotagem (Crosstab):** A variável categórica `exerce_atividade_remunerada` (S/N) foi pivotada para colunas distintas. Isso permitiu o cálculo vetorial da **Taxa de Conversão Real (KPI 1)**.

#### 3. Modelagem Preditiva (Predictive Analytics)
Implementação de algoritmo supervisionado para cálculo do **Score de Propensão (KPI 3)**.
* **Pré-processamento:** Aplicação de `LabelEncoder` para transformação de variáveis categóricas (Cidade, Gênero, Idade) em vetores numéricos interpretáveis.
* **Algoritmo:** Utilização do **Random Forest Regressor** (*Scikit-Learn*).
    * *Justificativa:* A escolha de Florestas Aleatórias deve-se à sua capacidade de capturar relações não-lineares entre idade e propensão (ex: a propensão não cresce linearmente com a idade, ela tem picos e vales) e sua robustez contra *overfitting* em dados categóricos de alta cardinalidade.
* **Target:** O modelo foi treinado para prever a `Taxa de Conversão Real`, gerando um *score* suavizado que corrige distorções de municípios com baixa amostragem estatística.

#### 4. Carga de Dados (Loading)
* **Output:** Geração do artefato final `baseBICNH_dashboard_2025_fase2.csv`, contendo as dimensões originais enriquecidas com os três KPIs calculados, formatado para ingestão direta no Microsoft Power BI.

---

## Dicionário de Dados (Output Final)

Abaixo, a descrição técnica das variáveis contidas no dataset processado entregue para a camada de visualização.

| Variável | Tipo | Descrição Técnica | Aplicação no Negócio |
| :--- | :--- | :--- | :--- |
| `Cidade` | String | Município de registro da CNH. | Filtragem geográfica e Mapas de Calor. |
| `Faixa_Etaria` | String | Intervalo etário do condutor. | Segmentação de Perfil. |
| `Genero` | String | Classificação de gênero. | Segmentação de Perfil. |
| `KPI_Volume_Demanda` | Int64 | Soma absoluta de condutores com EAR='S'. | **(KPI 2)** Dimensionamento de mercado (TAM) para abertura de oficinas. |
| `KPI_Taxa_Conversao` | Float64 | Razão entre EAR e Total no perfil. | **(KPI 1)** Diagnóstico de eficiência histórica. |
| `KPI_Score_Propensao` | Float64 | Probabilidade predita pelo modelo (0.0 a 1.0). | **(KPI 3)** Identificação de leads qualificados ("O Cliente Ideal") para frotas. |

---

## 🛠️ Requisitos para Reprodução

Para executar o código fonte deste projeto, são necessárias as seguintes bibliotecas Python (já incluídas no ambiente Google Colab):

```python
pandas>=1.3.5
numpy>=1.21.6
scikit-learn>=1.0.2

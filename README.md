# 🛒 Pipeline de Engenharia de Dados: E-Commerce Medallion Architecture com Apache Spark

## 📝 Descrição do Projeto
Este projeto simula o ambiente de engenharia de dados de uma empresa real de e-commerce utilizando o dataset público da **Olist** (plataforma de e-commerce brasileira). O objetivo principal é construir um pipeline de dados robusto de ponta a ponta, coletando dados brutos e transformando-os em tabelas de alto valor analítico para tomadas de decisão estratégicas.

O projeto foi desenvolvido inteiramente dentro do ecossistema **Databricks (Serverless)**, utilizando o **Apache Spark** (via APIs PySpark e Spark SQL) como motor de processamento distribuído e o **Delta Lake** para garantir transações ACID.

---

## 🏗️ Arquitetura do Projeto (Medallion Architecture)

O pipeline segue o padrão de medalhão para garantir a qualidade, governança e organização dos dados dentro do Data Lakehouse:



1. **Landing Zone (Data Lake / Volumes):** Armazenamento dos arquivos originais (`.csv`) extraídos do Kaggle em uma área de staging isolada no Unity Catalog.
2. **Camada Bronze (Dados Brutos):** Leitura dos arquivos CSV via Spark e persistência imediata no formato Delta Lake, mantendo o histórico original sem nenhuma alteração estrutural.
3. **Camada Silver (Dados Higienizados):** Limpeza de dados, padronização de strings, eliminação de duplicidades e correção da tipagem de dados (ex: conversão de strings para Timestamps).
4. **Camada Gold (Tabelas de Negócio):** Cruzamento das tabelas estruturadas via Spark SQL utilizando agregações avançadas e Window Functions para responder a dores reais de negócio.

---

## 🛠️ Tecnologias e Ferramentas Utilizadas
* **Plataforma:** Databricks Free Edition (Serverless Compute)
* **Motor de Processamento:** Apache Spark 
* **Linguagens:** Python (PySpark DataFrame API) e SQL (Spark SQL)
* **Armazenamento:** Delta Lake & Unity Catalog Volumes
* **Fonte de Dados:** [Brazilian E-Commerce Public Dataset by Olist (Kaggle)](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

---

## 🧭 Estrutura dos Notebooks do Pipeline

### 📍 1. Ingestão (`01_ingestion_bronze`)
* **Linguagem:** PySpark
* **Objetivo:** Lê as tabelas de Clientes, Pedidos e Itens dos Pedidos diretamente do Volume corporativo e realiza o parse inicial com `inferSchema`. Os DataFrames resultantes são salvos como tabelas Delta Lake com a estratégia de overwrite.

### 📍 2. Transformação e Qualidade (`02_transform_silver`)
* **Linguagem:** PySpark
* **Objetivo:** Aplicação de regras de higienização de dados:
  * Remoção de registros nulos em chaves primárias usando `.dropna()`.
  * Padronização de strings (Cidades e Estados) com `upper()` e `trim()`.
  * Conversão de tipos de dados textuais para `Timestamp` real (`to_timestamp()`).
  * Deduplicação de registros de vendas através de `.dropDuplicates()`.

### 📍 3. Modelagem e Indicadores (`03_analytics_gold`)
* **Linguagem:** Spark SQL
* **Objetivo:** Cruzamento dos dados limpos para disponibilização de indicadores estratégicos:
  * `gold_faturamento_mensal`: Consolidação de receita de produtos, fretes e volume de pedidos agrupados por mês/ano.
  * `gold_performance_logistica`: Cálculo de tempo médio real de entrega por estado e análise de desvios operacionais.
  * `gold_ranking_clientes`: Identificação e classificação dos 10 clientes com maior volume financeiro de compras utilizando funções de janela (`DENSE_RANK`).

---

## 📈 Próximos Passos (Roadmap de Produção)
Para aproximar este projeto de um cenário real de produção corporativa, as próximas etapas de evolução envolvem:
* [ ] **Orquestração:** Agendamento e monitoramento do pipeline ponta a ponta utilizando o **Databricks Workflows (Jobs)**.
* [ ] **Ingestão Incremental:** Substituição da carga total pelo **Apache Spark Auto Loader** (`cloudFiles`) para processar apenas arquivos novos.
* [ ] **CI/CD:** Vinculação do ambiente de desenvolvimento diretamente ao GitHub utilizando o **Databricks Git Folders**.

---
*Projeto desenvolvido como parte dos estudos práticos em Engenharia de Dados e Big Data.*


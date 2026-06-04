# 🛒 Pipeline de Engenharia de Dados: E-Commerce Medallion Architecture com Apache Spark

## 📝 Descrição do Projeto
Desenvolvi este projeto com o objetivo de simular o ecossistema de dados e o pipeline analítico de uma empresa real de e-commerce. Para isso, utilizei o dataset público da **Olist** (uma grande plataforma de e-commerce brasileira), que contém informações reais de clientes, pedidos e itens vendidos.

O objetivo principal foi construir uma esteira de dados ponta a ponta dentro do ecossistema **Databricks**, utilizando o **Apache Spark** (via APIs PySpark e Spark SQL) como motor de processamento distribuído e o formato **Delta Lake** para garantir a confiabilidade e transações ACID no nosso repositório.

---

## 🏗️ Arquitetura do Projeto (Medallion Architecture)

O pipeline foi estruturado seguindo o padrão de mercado da Arquitetura Medallion para garantir governança, organização e qualidade dos dados:

1. **Landing Zone (Data Lake / Volumes):** Como utilizei a versão Free (Serverless) do Databricks, simulei o nosso Data Lake corporativo criando um **Volume no Unity Catalog**. É aqui que os arquivos originais em `.csv` extraídos do Kaggle foram armazenados inicialmente.
2. **Camada Bronze (Dados Brutos):** Criei um notebook em PySpark para ler os CSVs do Volume e persistí-los imediatamente como tabelas Delta Lake, garantindo o armazenamento do histórico original sem nenhuma alteração estrutural.
3. **Camada Silver (Dados Higienizados):** Utilizando PySpark, apliquei regras severas de higienização, tratamento de nulos em chaves primárias, deduplicação de registros e padronização de campos de texto.
4. **Camada Gold (Tabelas de Negócio):** Mudei a abordagem para o Spark SQL para cruzar as tabelas limpas da Silver e gerar agregados de alto valor para tomadas de decisão analíticas.

---

## 🧠 Desafios Encontrados e Soluções Práticas
* **Manipulação de Infraestrutura Limitada:** Por estar em um ambiente de estudos gratuito (Serverless), não dispunha de storages externos como AWS S3 ou Azure ADLS. Contornei o problema mapeando caminhos físicos via **Volumes do Unity Catalog** (`/Volumes/workspace/default/...`), replicando com sucesso a estrutura de pastas de uma empresa real.
* **Tipagem de Datas em Larga Escala:** Os arquivos originais traziam todas as colunas de data (como momentos de compra, aprovação e entrega) formatadas como texto (String). Na camada Silver, utilizei funções nativas do Spark (`to_timestamp`) para convertê-las nos tipos corretos, permitindo cálculos de intervalo de tempo precisos na camada seguinte.
* **Garantia de Unicidade:** Identifiquei registros duplicados de pedidos na carga bruta. Utilizei o método `.dropDuplicates(["order_id"])` do PySpark para limpar a massa de dados antes que ela chegasse aos relatórios finais.

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
* Realiza a leitura estruturada dos arquivos CSV (`olist_customers_dataset.csv`, `olist_orders_dataset.csv` e `olist_order_items_dataset.csv`) e faz a carga inicial na camada `bronze_` em formato Delta.

### 📍 2. Transformação e Qualidade (`02_transform_silver`)
* **Linguagem:** PySpark
* Executa o tratamento de dados: remoção de nulos críticos via `.dropna()`, padronização de strings de localização com `upper()` e `trim()`, e a conversão de strings de data para timestamps reais. Salva os dados limpos nas tabelas `silver_`.

### 📍 3. Modelagem e Indicadores (`03_analytics_gold`)
* **Linguagem:** Spark SQL
* Consome a camada Silver para estruturar três visões de negócio fundamentais na camada Gold:
  * `gold_faturamento_mensal`: Consolidação histórica de receita de produtos, custos de frete e volume de pedidos por mês/ano.
  * `gold_performance_logistica`: Cálculo do tempo médio real de entrega por estado e desvio em relação à estimativa passada ao cliente.
  * `gold_ranking_clientes`: Identificação dos 10 clientes com maior volume financeiro acumulado em compras utilizando funções de janela (`DENSE_RANK`).

---
*Projeto finalizado com sucesso, servindo como base sólida para estudos de processamento distribuído de dados e arquitetura Lakehouse.*

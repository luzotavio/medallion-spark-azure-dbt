# Medallion Architecture with DBT, Spark and Azure Data Factory

Este projeto implementa uma arquitetura de dados **Medallion (Bronze -> Silver -> Gold)** completa, integrando ferramentas de orquestração, extração e transformação no ecossistema **Azure**.

O objetivo é processar os dados do banco de dados `Azure SQL Database` (AdventureWorks) através de um pipeline automatizado até a camada analítica final.

---

## 🏗️ Arquitetura e Fluxo de Dados

A solução segue o fluxo **ELT (Extract, Load, Transform)**:

1.  **Extração e Carga (E & L):** Realizada via **Azure Data Factory (ADF)**.
    -   **Lookup & ForEach:** O ADF consulta os metadados da fonte (schemas e tabelas).
    -   **Copy Data:** Extrai os dados dinamicamente do `Azure SQL Database` e os carrega na camada **Bronze** do Data Lake em formato **Parquet**, organizados por pastas de data.
2.  **Transformação (T):** Realizada via **Azure Databricks** e **dbt**.
    -   **Camada Silver:** O dbt executa `snapshots` para criar tabelas com histórico (SCD Tipo 2) no formato **Delta**.
    -   **Camada Gold:** O dbt modela as dimensões e fatos finais para consumo.

---

## ☁️ Infraestrutura Azure

A infraestrutura foi montada com os seguintes componentes:

-   **Resource Group:** Organização lógica de todos os recursos.
-   **Azure Data Lake Storage (ADLS Gen2):** Armazenamento hierárquico com containers dedicados:
    -   `bronze`: Dados brutos em Parquet.
    -   `silver`: Dados históricos em Delta.
    -   `gold`: Dados analíticos em Delta.
-   **Azure Data Factory:** Orquestrador dos pipelines de ingestão.
-   **Azure Key Vault:** Gestão segura de segredos (chaves de acesso e strings de conexão).
-   **Azure SQL Database:** Fonte primária dos dados (AdventureWorks).
-   **Azure Databricks:** Engine de processamento Spark.
    -   Configuração de clusters e conexão segura com ADLS via Secrets do Key Vault.
    -   Integração com o ADF para execução de notebooks de suporte.

---

## 📸 Camada Silver (Snapshots dbt)

A camada Silver utiliza o **dbt** para gerenciar o **CDC (Change Data Capture)**. A estratégia `check` monitora alterações em todas as colunas para garantir a integridade do histórico.

### Snapshots:
-   `address_snapshot`, `customer_snapshot`, `product_snapshot`, etc.
-   Localização: `abfss://silver@medallionlo.dfs.core.windows.net/`

---

## 🏆 Camada Gold (Marts dbt)

A camada Gold transforma os dados da Silver em modelos dimensionais prontos para BI.

### Modelos:
-   **dim_customer:** Dados consolidados de clientes e endereços.
-   **dim_product:** Catálogo de produtos refinado.
-   **sales (Fato):** Visão completa das transações de vendas.
-   Localização: `abfss://gold@medallionlo.dfs.core.windows.net/`

---

## 🛠️ Tecnologias Utilizadas

-   **Azure Data Factory** (Orquestração)
-   **Azure Databricks & Spark** (Processamento)
-   **dbt Core** (Transformação e Modelagem)
-   **Delta Lake** (Armazenamento de Tabelas)
-   **Azure Data Lake Gen2** (Storage)
-   **Azure Key Vault** (Segurança)

---

## 🚀 Como o Pipeline Funciona

1.  O **ADF** inicia o pipeline, lê as tabelas do **SQL Database** e salva no **Bronze**.
2.  O **Databricks** (via dbt) lê os dados do **Bronze**, aplica a lógica de snapshots e salva no **Silver** como Delta.
3.  O **dbt** processa os dados do **Silver**, cria as tabelas dimensionais e salva no **Gold**.

---
*Documentação consolidada a partir das análises de infraestrutura Azure, snapshots e modelos de marts.*

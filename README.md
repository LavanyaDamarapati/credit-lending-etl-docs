   #     Credit Lending ETL

## a) Project overview:

This project implements an ETL pipeline for credit lending data using PySpark and Delta Lake.  
It ingests data from ADLS and Azure SQL, applies Bronze-Silver-Gold transformations, and performs audit logging.

### Code Repo: https://github.com/LavanyaDamarapati/credit-lending-etl

### Simple architecture diagram

                +-------------------+
                |   ADLS / Azure SQL|
                +---------+---------+
                          |
                          v
                  [ Readers Layer ]
                          |
                          v
                 +-----------------+
                 |     Bronze      |
                 |  Raw Delta Data |
                 +-----------------+
                          |
                          v
                 +-----------------+
                 |     Silver      |
                 | Clean + SCD2 +  |
                 | PII Masking     |
                 +-----------------+
                          |
                          v
                 +-----------------+
                 |      Gold       |
                 | Aggregations   |
                 +-----------------+
                          |
                          v
                 +-----------------+
                 |   Audit Logs    |
                 +-----------------+

             (Orchestrated by Airflow DAG)



## b) Folder Structure:

- `framework/` : Core ETL modules (readers, writers, DQ, SCD2, transformations, audit, engine)
- `pipelines/` : YAML configuration for datasets
- `notebooks/` : Notebooks for running the pipeline and generating sample data
- `airflow/` : Jinja2 template and rendered DAGs
- `tests/` : Pytest test modules

```text
credit-lending/
│
├── framework/ # CORE ETL FRAMEWORK
│ ├── readers/
│ ├── writers/
│ ├── dq/
│ ├── transformations/
│ ├── scd/
│ ├── audit/
│ └── engine.py
│
├── pipelines/ # CONFIG ONLY
│ └── credit_lending.yaml
│
├── notebooks/ # OPTIONAL execution layer
│ ├── run_engine.ipynb
│ └── sample_data_generate.ipynb
│
├── airflow/
│ ├── dag_template.jinja2
│ └── dags/ # rendered DAGs
│
├── tests/
│ ├── test_config.py
│ ├── test_dq.py
│ ├── test_transformations.py
│ └── test_writers.py

```
## c) Running the Pipeline


### 1. From Notebooks (Databricks)
1. Open `notebooks/run_engine.ipynb`.
2. Ensure `project_root` points to repository root.
3. Set `config_path` to `../pipelines/credit_lending.yaml`.
4. Run the notebook cells.

### 2. From Airflow
1. Render the DAG using `dag_template.jinja2`:
2. Place generated DAG inside Airflow dags/ folder.
3. Trigger DAG in Airflow UI.

## d) Output Samples
   ### please refer the document link - https://github.com/LavanyaDamarapati/credit-lending-etl-docs/blob/main/Architecture_and_SampleOutput.docx
  #### Execution framework result
  #### Bronze Layer (Raw Ingestion)
  #### Silver Layer (Transformed + SCD2 + PII Masking)
  #### Gold Layer (Aggregations)
  #### Audit result

## e) Design Decisions
  #### Configuration Management
  
  - YAML is used to define:
  - Source details
  - Target layers
  - DQ rules
  - Transformations
  - Aggregations
  - This allows pipelines to be extended without modifying core framework code.

 #### Modular Framework

  -  Readers – Handle source ingestion from ADLS and Azure SQL
  -  Writers – Manage Delta table writes with schema evolution support
  -  Data Quality (DQ) – Apply validation rules and enforce data standards
  -  Transformations – Perform business logic such as PII masking and column selection
  -  SCD – Implement Slowly Changing Dimension (Type 2) processing
  -  Audit – Track pipeline execution status, row counts, and timestamps

## f) Orchestration

- Airflow DAG is generated using Jinja2 templating
- Enables reusable DAG structure across environments and pipelines

## e) Trade-offs & Assumptions
- YAML was chosen for pipeline configuration for simplicity and readability.

     - Alternatively, configurations could be stored in database tables or managed via metadata services.
- Sample data was generated via a Python script and loaded into the ADLS landing layer to simulate real-world ingestion for testing purposes.
-  Unity Catalog is configured to manage Delta tables, with schemas organized into Bronze, Silver, Gold, and Control layers.
- Ingestion is handled directly via Spark readers.

    - In production scenarios, Azure Data Factory (ADF) or similar tools could be leveraged for ingestion orchestration depending on scalability, governance, and operational requirements.

- Given timeline constraints, this implementation focuses on framework flexibility and core ETL patterns.

    - There is room for further optimization, enhanced error handling, and performance tuning.

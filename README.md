# End-to-End Medallion Data Platform in Microsoft Fabric (UIDAI Aadhaar Data)

An enterprise-grade, metadata-driven data engineering pipeline built using **Microsoft Fabric**. This project ingests, transforms, and models public Aadhaar enrolment, demographic, and biometric datasets into a reporting-ready Star Schema following the Medallion Architecture (Bronze ➔ Silver ➔ Gold).

---

## Architecture & Tech Stack
* **Cloud Platform:** Microsoft Fabric (Lakehouse, Warehouse, Data Pipelines, Notebooks)
* **Languages & Engines:** PySpark, T-SQL, Delta Lake
* **Orchestration:** Fabric Data Pipelines (`Get Metadata`, `Filter`, `ForEach`, `Stored Procedure` activities)

---

## Data Pipeline Flow




```mermaid
graph TD
    %% Define Styles
    classDef bronze fill:#8B5A2B,stroke:#333,stroke-width:2px,color:#fff;
    classDef silver fill:#C0C0C0,stroke:#333,stroke-width:2px,color:#000;
    classDef gold fill:#FFD700,stroke:#333,stroke-width:2px,color:#000;
    classDef BI fill:#F2C811,stroke:#333,stroke-width:2px,color:#000;
    classDef process fill:#ebebeb,stroke:#333,stroke-width:1px,color:#000;

    %% Nodes
    A[(Raw CSVs<br/>Bronze Lakehouse)]:::bronze
    
    subgraph Pipeline [Metadata-Driven Pipeline]
        B1[Get Metadata]:::process --> B2[Filter]:::process
        B2 --> B3[ForEach Loop]:::process
    end
    
    C[Parameterized PySpark Notebook<br/>Silver Layer]:::silver
    D[(SQL Stored Procedure<br/>Gold Layer - Star Schema)]:::gold
    
    subgraph Serving [Reporting Layer]
        E1[Fabric SQL Warehouse]:::BI --> E2[Semantic Model]:::BI
        
    end

    %% Flows
    A --> Pipeline
    Pipeline -->  C
    C --> |Auto-Schema Inference & Delta Overwrites| D
    D --> |Full Load & Clean Architecture| Serving

```

---

## Project Structure
1. **Bronze Layer:** Ingests raw multi-part CSV files (`enrolment`, `demographic`, `biometric`) and reference files (`states`) into the Lakehouse file storage.
2. **Silver Layer:** 
   * A single parameterized PySpark notebook handles dynamic file reading using wildcards (`*.csv`) and schema inference.
   * Automated via a metadata-driven Fabric pipeline using `Get Metadata` and `ForEach` iteration to loop through dataset folders.
3. **Gold Layer:** 
   * A modular T-SQL Stored Procedure (`usp_Load_Gold_Model_Full`) that performs a full-reload pattern.
   * Cleans, aggregates, and transforms data into standardized dimension tables (`Dim_Date`, `Dim_Location`) and a consolidated fact table (`Fact_Aadhaar_Activity`).

---

## Key Engineering Highlights
* **Metadata-Driven Automation:** Replaced redundant, manual pipelines with a dynamic loop that processes multiple distinct data domains using pipeline parameters .
* **Optimized SQL Modeling:** Utilized Common Table Expressions (CTEs) and pre-aggregation before `FULL OUTER JOIN` operations to completely eliminate cartesian explosions and row duplication.
* **Fabric-Native Optimization:** Leveraged Delta Lake transaction formats and Fabric Warehouse V-Order storage for high-performance querying without manual indexing overhead.


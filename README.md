# Travel-AI-Assistant
```mermaid
graph TD
    %% --- Diagram Title ---
    %% This diagram outlines the complete system architecture for VoyageAI.
    %% It is split into the live Request/Response flow and the offline Data Platform flow.

    %% --- 1. LIVE REQUEST/RESPONSE FLOW ---

    subgraph UI ["User Interface"]
        style UI fill:#e6f3ff,stroke:#333,stroke-width:2px
        User([fa:fa-user User]) -.-> FE[Frontend <br/>(React/Vue SPA)]
    end

    subgraph K8sPlatform ["Cloud Platform (Managed by Kubernetes)"]
        style K8sPlatform fill:#f0f0f0,stroke:#555,stroke-width:2px,stroke-dasharray: 5 5

        subgraph ServingLayer ["Serving Layer"]
            style ServingLayer fill:#d4edda,stroke:#155724
            APIGW[API Gateway<br/>(FastAPI)]
        end

        subgraph IntelligenceLayer ["Intelligence Layer"]
            style IntelligenceLayer fill:#fff3cd,stroke:#856404
            RAGS[RAG Service<br/>(FastAPI)]
        end

        subgraph PersistenceLayer ["Persistence Layer"]
            style PersistenceLayer fill:#f8d7da,stroke:#721c24
            DWH[fa:fa-database Data Warehouse<br/>(Snowflake/BigQuery)<br/><b>Gold Layer Tables</b>]
            VDB[fa:fa-project-diagram Vector DB<br/>(Pinecone/Weaviate)<br/><b>Travel Embeddings</b>]
        end

        %% Connections for the Request Flow
        FE -- "1. POST /itinerary (JSON)" --> APIGW
        APIGW -- "2. Forward Validated Request" --> RAGS
        RAGS -- "3. Structured SQL Query" --> DWH
        RAGS -- "4. Semantic Search Query" --> VDB
    end

    subgraph ExternalAIService ["External AI Service"]
        style ExternalAIService fill:#e2d9f3,stroke:#4b2a8a
        LLM[fa:fa-robot Generative LLM<br/>(e.g., Llama 3.2)]
    end

    %% Connections for the AI Generation and Response
    RAGS -- "5. Construct Augmented Prompt" --> LLM
    LLM -- "6. Generate Itinerary" --> RAGS
    RAGS -- "7. Return Final Response" --> APIGW
    APIGW -- "8. Stream Markdown Response" --> FE

    %% --- 2. OFFLINE DATA PLATFORM FLOW (Knowledge Base Factory) ---

    subgraph DataSources ["Data Sources (External)"]
        style DataSources fill:#f4f4f4,stroke:#666
        APIs_Batch[Batch APIs<br/>(Foursquare, Weather)]
        APIs_Stream[Streaming APIs<br/>(Skyscanner Flights)]
    end

    subgraph DataPlatform ["Data Platform (Offline Pipelines)"]
        style DataPlatform fill:#f0f0f0,stroke:#555,stroke-width:2px,stroke-dasharray: 5 5

        subgraph Ingestion ["Ingestion"]
            style Ingestion fill:#cce5ff,stroke:#004085
            Airflow[fa:fa-calendar-alt Airflow<br/>(Batch DAGs)]
            Kafka[fa:fa-stream Kafka<br/>(Real-time Topics)]
        end

        subgraph ProcessingStorage ["Processing & Storage"]
            style ProcessingStorage fill:#d1ecf1,stroke:#0c5460
            DataLake[fa:fa-archive Data Lake<br/>(S3/GCS)<br/>Bronze & Silver Layers]
            Spark[fa:fa-cogs Spark Jobs<br/>(Transformation)]
            dbt[dbt Models<br/>(Gold Layer Logic)]
        end

        %% Connections for the Data Flow
        APIs_Batch --> Airflow
        APIs_Stream --> Kafka
        Airflow -- "Lands Raw Batch Data" --> DataLake
        Kafka -- "Lands Raw Stream Data" --> DataLake
        DataLake -- "Reads Bronze/Silver" --> Spark
        Spark -- "Writes Silver" --> DataLake
        Spark -- "Generates & Writes Embeddings" --> VDB
        DataLake -- "Source for dbt" --> dbt
        dbt -- "Builds & Tests Gold Tables" --> DWH
    end
```

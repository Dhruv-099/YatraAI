# Travel-AI-Assistant
```mermaid
graph TD
    %% Diagram Title
    %% --- Main Request/Response Flow ---
    subgraph "User Interface"
        style "User Interface" fill:#e6f3ff,stroke:#333,stroke-width:2px
        User([<i class='fa fa-user'></i> User]) -.-> FE[Frontend <br/>(React/Vue SPA)]
    end

    subgraph "Cloud Platform (Managed by Kubernetes)"
        style "Cloud Platform (Managed by Kubernetes)" fill:#f0f0f0,stroke:#555,stroke-width:2px,stroke-dasharray: 5 5

        subgraph "Serving Layer"
            style "Serving Layer" fill:#d4edda,stroke:#155724
            APIGW[API Gateway<br/>(FastAPI)]
        end

        subgraph "Intelligence Layer"
            style "Intelligence Layer" fill:#fff3cd,stroke:#856404
            RAGS[RAG Service<br/>(FastAPI)]
        end

        subgraph "Persistence Layer"
            style "Persistence Layer" fill:#f8d7da,stroke:#721c24
            DWH[<i class='fa fa-database'></i> Data Warehouse<br/>(Snowflake/BigQuery)<br/><b>Gold Layer Tables</b>]
            VDB[<i class='fa fa-project-diagram'></i> Vector DB<br/>(Pinecone/Weaviate)<br/><b>Travel Embeddings</b>]
        end

        %% Connections for the Request Flow
        FE -- "1. POST /itinerary (JSON)" --> APIGW
        APIGW -- "2. Forward Validated Request" --> RAGS
        RAGS -- "3. Structured SQL Query" --> DWH
        RAGS -- "4. Semantic Search Query" --> VDB
    end

    subgraph "External AI Service"
        style "External AI Service" fill:#e2d9f3,stroke:#4b2a8a
        LLM[<i class='fa fa-robot'></i> Generative LLM<br/>(e.g., Llama 3.2)]
    end

    %% Connections for the AI Generation
    RAGS -- "5. Construct Augmented Prompt" --> LLM
    LLM -- "6. Generate Itinerary" --> RAGS
    RAGS -- "7. Return Final Response" --> APIGW
    APIGW -- "8. Stream Markdown Response" --> FE

    %% --- Data Ingestion & Processing Flow (The Knowledge Base Factory) ---
    subgraph "Data Sources (External)"
        style "Data Sources (External)" fill:#f4f4f4,stroke:#666
        APIs_Batch[Batch APIs<br/>(Foursquare, Weather)]
        APIs_Stream[Streaming APIs<br/>(Skyscanner Flights)]
    end

    subgraph "Data Platform (Offline Pipelines)"
        style "Data Platform (Offline Pipelines)" fill:#f0f0f0,stroke:#555,stroke-width:2px,stroke-dasharray: 5 5

        subgraph Ingestion
            style Ingestion fill:#cce5ff,stroke:#004085
            Airflow[<i class='fa fa-calendar-alt'></i> Airflow<br/>(Batch DAGs)]
            Kafka[<i class='fa fa-stream'></i> Kafka<br/>(Real-time Topics)]
        end

        subgraph "Processing & Storage"
            style "Processing & Storage" fill:#d1ecf1,stroke:#0c5460
            DataLake[<i class='fa fa-archive'></i> Data Lake<br/>(S3/GCS)<br/>Bronze & Silver Layers]
            Spark[<i class='fa fa-cogs'></i> Spark Jobs<br/>(Transformation)]
            dbt[dbt Models<br/>(Gold Layer Logic)]
        end

        %% Connections for the Data Flow
        APIs_Batch --> Airflow
        APIs_Stream --> Kafka
        Airflow -- "Lands Raw Batch Data" --> DataLake
        Kafka -- "Lands Raw Stream Data" --> DataLake
        DataLake -- "Reads Bronze/Silver" --> Spark
        Spark -- "Writes Silver/Gold" --> DataLake
        Spark -- "Generates & Writes Embeddings" --> VDB
        DataLake -- "Loads Gold Data" --> dbt
        dbt -- "Builds & Tests Gold Tables" --> DWH
    end

    %% Add FontAwesome support (required for icons)
    linkStyle default text-decoration:none
    classDef default font-family: 'Helvetica', sans-serif;
```

# Travel-AI-Assistant
```mermaid
graph TD
    %% --- Diagram Title & Description ---
    %% This architecture is for a Form-Driven Generative AI Travel Platform.
    %% Key Design: A "Filter-then-Generate" approach. A fast, structured query on PostgreSQL 
    %% finds candidates, and RAG is then used to create a rich, narrative response.

    %% --- 1. LIVE REQUEST/RESPONSE FLOW (User-Facing) ---

    subgraph UserInteraction ["User Interaction Layer"]
        style UserInteraction fill:#e6f3ff,stroke:#004085
        User([fa:fa-user User]) -- "Fills out Q&A Form" --> FE["Frontend (React/Vue)<br/>Displays recommendations & map (OSM/Leaflet.js)"]
    end

    subgraph KubernetesCluster ["Cloud Platform (Deployed on Kubernetes)"]
        style KubernetesCluster fill:#f0f0f0,stroke:#555,stroke-width:2px,stroke-dasharray: 5 5

        subgraph ServingLayer ["API & Intelligence Layer"]
            style ServingLayer fill:#d4edda,stroke:#155724
            Backend["Backend Service (FastAPI)<br/>1. Validates Form Input<br/>2. Orchestrates Filtering & Generation"]
        end

        subgraph PersistenceLayer ["Persistence Layer"]
            style PersistenceLayer fill:#f8d7da,stroke:#721c24
            PostgresDB["fa:fa-database PostgreSQL DB<br/><b>'Travel Genome'</b><br/>Stores structured location data with filterable attributes (tags, budget, category)"]
            VectorDB["fa:fa-project-diagram Vector DB (Qdrant/Weaviate)<br/>Stores embeddings for descriptions & POIs for semantic enrichment"]
        end

        %% Connections for the Live Request Flow
        FE -- "1. POST /recommend (Form JSON)" --> Backend
        Backend -- "<b>2. PRIMARY FILTER: SQL Query</b><br/>(WHERE category='MOUNTAIN' AND budget='MID_RANGE'...)" --> PostgresDB
        PostgresDB -- "3. Return Top 5 Matching Locations" --> Backend
        Backend -- "4. Retrieve Rich Context for Matches<br/>(Descriptions, POIs, etc.)" --> PostgresDB
        Backend -- "5. (Optional) Enrich with Semantic Search<br/>'Find unique cultural spots in these cities'" --> VectorDB
    end

    subgraph ExternalServices ["External AI Services"]
        style ExternalServices fill:#e2d9f3,stroke:#4b2a8a
        LLM["fa:fa-robot Generative LLM<br/>(OpenAI/Llama 3)"]
    end

    %% Connections for AI Generation and Final Response
    Backend -- "6. Construct Augmented Prompt" --> LLM
    LLM -- "7. Generate Itinerary Text" --> Backend
    Backend -- "8. Return Final JSON<br/>(Generated Text + Structured Map Data)" --> FE


    %% --- 2. OFFLINE DATA PLATFORM FLOW ('Travel Genome' Factory) ---

    subgraph DataSources ["External Data Sources"]
        style DataSources fill:#f4f4f4,stroke:#666
        Wikidata["fa:fa-wikipedia-w Wikidata API<br/>(Cities, States, Coordinates)"]
        OSM["fa:fa-map-marked-alt OpenStreetMap API<br/>(POIs, Airports, Stations)"]
        Wikipedia["fa:fa-book Wikipedia API<br/>(Textual Descriptions)"]
    end

    subgraph DataPlatform ["Data Engineering Platform (Offline Pipelines)"]
        style DataPlatform fill:#f0f0f0,stroke:#555,stroke-width:2px,stroke-dasharray: 5 5

        subgraph Orchestration ["Orchestration"]
            style Orchestration fill:#cce5ff,stroke:#004085
            Airflow["fa:fa-calendar-alt Airflow<br/>Schedules and manages the data pipeline DAGs"]
        end

        subgraph Processing ["Processing & Staging"]
            style Processing fill:#d1ecf1,stroke:#0c5460
            DataLake["fa:fa-archive Data Lake (S3/GCS)<br/>Stores raw and intermediate data (Parquet)"]
            Spark["fa:fa-cogs Spark Jobs<br/>Fetches, cleans, joins, categorizes, and generates embeddings"]
        end

        %% Connections for the Offline Data Flow
        Airflow -- "Triggers scheduled jobs" --> Spark
        Spark -- "1. Fetch Raw Data" --> Wikidata
        Spark -- "1. Fetch Raw Data" --> OSM
        Spark -- "1. Fetch Raw Data" --> Wikipedia
        Spark -- "2. Process & Stage Data" --> DataLake
        Spark -- "3. Load Final Structured 'Genome' Data" --> PostgresDB
        Spark -- "4. Load Text Embeddings" --> VectorDB
    end

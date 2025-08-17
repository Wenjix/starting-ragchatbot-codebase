# RAG System Query Flow Diagram

```mermaid
sequenceDiagram
    participant User
    participant Frontend as Frontend<br/>(script.js)
    participant API as FastAPI<br/>(app.py)
    participant RAG as RAG System<br/>(rag_system.py)
    participant AI as AI Generator<br/>(ai_generator.py)
    participant Tools as Search Tools<br/>(search_tools.py)
    participant Vector as Vector Store<br/>(vector_store.py)
    participant Chroma as ChromaDB
    participant Claude as Claude API
    participant Session as Session Manager

    User->>Frontend: Types query & clicks send
    Frontend->>Frontend: Show loading spinner
    Frontend->>API: POST /api/query<br/>{"query": "...", "session_id": "..."}
    
    API->>API: Create session if needed
    API->>RAG: rag_system.query(query, session_id)
    
    RAG->>Session: Get conversation history
    Session-->>RAG: Previous messages (if any)
    
    RAG->>AI: generate_response(query, history, tools)
    AI->>Claude: Send query + system prompt + tools
    
    Note over Claude: Claude decides if search needed<br/>based on query type
    
    Claude-->>AI: Tool use request:<br/>search_course_content(query, filters)
    
    AI->>Tools: execute_tool("search_course_content", params)
    Tools->>Vector: search(query, course_name, lesson_number)
    
    Vector->>Vector: Convert query to embeddings<br/>(sentence transformers)
    Vector->>Chroma: Semantic similarity search
    Chroma-->>Vector: Ranked results + metadata
    
    Vector-->>Tools: SearchResults(docs, metadata, distances)
    Tools->>Tools: Format results with context<br/>Track sources for UI
    Tools-->>AI: Formatted search results
    
    AI->>Claude: Send tool results back
    Claude-->>AI: Final natural language response
    
    AI-->>RAG: Generated response text
    RAG->>Tools: Get sources from last search
    Tools-->>RAG: Source list (course + lesson info)
    
    RAG->>Session: Store query + response
    RAG-->>API: (response, sources)
    
    API-->>Frontend: JSON response:<br/>{"answer": "...", "sources": [...], "session_id": "..."}
    
    Frontend->>Frontend: Remove loading spinner
    Frontend->>Frontend: Render AI response + sources
    Frontend->>Frontend: Update session state
    Frontend->>User: Display formatted response
```

## Key Components Flow

```mermaid
graph TD
    A[User Query] --> B[Frontend UI]
    B --> C[FastAPI Endpoint]
    C --> D[RAG System]
    D --> E[Session Manager]
    D --> F[AI Generator]
    F --> G[Claude API]
    G --> H{Tool Needed?}
    H -->|Yes| I[Search Tools]
    H -->|No| K[Direct Response]
    I --> J[Vector Store]
    J --> L[ChromaDB]
    L --> M[Embeddings Search]
    M --> N[Results + Metadata]
    N --> O[Formatted Response]
    O --> P[Claude Synthesis]
    K --> P
    P --> Q[Final Response]
    Q --> R[Sources Extraction]
    R --> S[Session Update]
    S --> T[API Response]
    T --> U[Frontend Display]
    
    style A fill:#e1f5fe
    style U fill:#e8f5e8
    style G fill:#fff3e0
    style L fill:#f3e5f5
```

## Data Structures

```mermaid
classDiagram
    class QueryRequest {
        +string query
        +string session_id?
    }
    
    class QueryResponse {
        +string answer
        +string[] sources
        +string session_id
    }
    
    class SearchResults {
        +string[] documents
        +dict[] metadata
        +float[] distances
        +string error?
    }
    
    class CourseChunk {
        +string content
        +string course_title
        +int lesson_number?
        +int chunk_index
    }
    
    QueryRequest --> QueryResponse
    SearchResults --> CourseChunk
```

## Processing Stages

1. **Input Stage**: User interaction → API request
2. **Orchestration Stage**: RAG system coordinates components
3. **Decision Stage**: Claude determines if search is needed
4. **Search Stage**: Vector similarity search in ChromaDB
5. **Synthesis Stage**: Claude generates response from search results
6. **Output Stage**: Formatted response with sources returned to UI

## Key Features Illustrated

- **Tool-based Architecture**: Claude autonomously decides when to search
- **Semantic Search**: Vector embeddings enable context-aware retrieval
- **Session Management**: Conversation history maintains context
- **Source Attribution**: UI displays where information came from
- **Chunked Content**: Documents processed into searchable segments
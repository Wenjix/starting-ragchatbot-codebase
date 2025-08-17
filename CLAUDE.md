# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
```bash
# Install dependencies
uv sync

# Start development server (recommended)
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Setup
```bash
# Create .env file with required API key
echo "ANTHROPIC_API_KEY=your_key_here" > .env

# Application access
# Web interface: http://localhost:8000
# API docs: http://localhost:8000/docs
```

## Architecture

### RAG System Design
This is a **tool-based RAG system** where Claude autonomously decides when to search course materials. The key architectural pattern is:

1. **Query Router**: AI determines if search is needed based on query type
2. **Tool-based Search**: Claude calls `search_course_content` tool when needed
3. **Semantic Retrieval**: Vector similarity search using sentence transformers + ChromaDB
4. **Response Synthesis**: AI generates natural language responses from search results

### Core Components

**RAG System** (`rag_system.py`): Main orchestrator that coordinates all components and manages the tool-based search workflow.

**AI Generator** (`ai_generator.py`): Manages Claude API interactions with tool calling capability. Uses system prompt to guide tool usage decisions.

**Search Tools** (`search_tools.py`): Implements `CourseSearchTool` with vector store integration. Tracks sources for UI attribution.

**Vector Store** (`vector_store.py`): ChromaDB wrapper with semantic search capabilities. Supports filtering by course name and lesson number.

**Document Processor** (`document_processor.py`): Processes structured course documents with format:
```
Course Title: [title]
Course Link: [url] 
Course Instructor: [instructor]

Lesson N: [title]
Lesson Link: [url]
[content]
```

**Session Manager** (`session_manager.py`): Maintains conversation history for context across queries.

### Data Flow
```
User Query → FastAPI → RAG System → AI Generator → Claude API
                                         ↓
Claude decides to search → Search Tool → Vector Store → ChromaDB
                                         ↓
Search Results → AI Synthesis → Response + Sources → Frontend
```

### Key Configuration
- **Chunking**: 800 chars with 100 char overlap for context preservation
- **Embedding Model**: `all-MiniLM-L6-v2` for vector embeddings
- **Claude Model**: `claude-sonnet-4-20250514`
- **Context Limits**: 5 search results max, 2 conversation messages history

### Document Structure
Course documents in `/docs/` follow strict format with metadata headers followed by lesson markers. The processor automatically extracts course/lesson hierarchy and creates contextual chunks.

### Frontend Integration
Static file serving via FastAPI with CORS enabled. Frontend communicates via `/api/query` and `/api/courses` endpoints. Sources from search tools are tracked and displayed in collapsible UI sections.

### Tool Calling Pattern
The system uses Anthropic's tool calling where Claude decides autonomously whether to search. This differs from traditional RAG where search always occurs. Tool results are passed back to Claude for synthesis into natural language responses.

## Working with Course Data

Course documents are automatically loaded from `/docs/` on startup. To add new courses, place structured text files in the docs directory and restart the server.

The system maintains course title uniqueness - duplicate course titles are skipped during loading to prevent data duplication in the vector store.
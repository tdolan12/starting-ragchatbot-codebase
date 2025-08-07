# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Course Materials RAG (Retrieval-Augmented Generation) System - a full-stack web application that enables intelligent querying of course materials using semantic search and AI-powered responses.

## Architecture

### Core Components

**RAGSystem (`backend/rag_system.py:10`)**: Main orchestrator that coordinates all system components. Handles document ingestion, query processing, and response generation with tool integration.

**Document Processing Pipeline**:
- `DocumentProcessor`: Parses structured course documents with specific format requirements (Course Title, Lesson N: Title, content sections)
- `VectorStore`: ChromaDB-based dual storage system with separate collections for course catalog and content chunks
- Sentence-based chunking with configurable overlap for optimal retrieval

**AI Integration**:
- `AIGenerator`: Anthropic Claude API integration with tool support and conversation history management
- `SearchTools`: Extensible tool system that allows AI to dynamically search course content
- Session-based conversations with context preservation

**Web Interface**:
- FastAPI backend serving both API endpoints and static frontend
- Single-page application with real-time chat, course statistics, and responsive design
- No build system required - vanilla HTML/CSS/JavaScript with Marked.js for markdown

### Data Flow

1. Course documents → DocumentProcessor → structured Course/Lesson objects
2. Processed content → VectorStore (dual ChromaDB collections: catalog + content)
3. User query → RAGSystem → AIGenerator with SearchTools
4. AI uses tools to search relevant content → generates contextualized response
5. Response + sources returned to frontend for display

## Development Commands

### Setup
```bash
uv sync                                    # Install dependencies (auto-handles Python 3.13)
```

Create `.env` file in root with:
```
ANTHROPIC_API_KEY=your_key_here
```

### Running the Application
```bash
./run.sh                                   # Recommended: uses run script
# OR manually:
cd backend && uv run uvicorn app:app --reload --port 8000
```

**Access Points:**
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs
- API Endpoints: `/api/query`, `/api/courses`

### Development
```bash
uv run python -c "import app; print('Import test')"  # Test imports
uv run uvicorn app:app --reload                      # Development server with hot reload
```

## Key Architecture Patterns

### Configuration Management
- Environment-based config (`config.py`) with `.env` support
- All settings centralized with sensible defaults
- Separate development vs production configurations

### Document Processing
- Expected format: `Course Title: [title]` followed by `Lesson N: [lesson_title]` sections
- Text chunked at sentence boundaries with configurable overlap
- Duplicate detection prevents re-processing existing courses

### Vector Storage Strategy
- **Catalog Collection**: Course metadata for high-level queries
- **Content Collection**: Detailed lesson chunks for specific content retrieval
- Embedding model: `all-MiniLM-L6-v2` (configurable)

### Tool-Based AI Architecture
- `ToolManager`: Registry for extensible search tools
- `CourseSearchTool`: Implements semantic search with source tracking
- AI can dynamically invoke tools based on query context
- Sources automatically tracked and returned with responses

### Session Management
- Conversation history preserved per session ID
- Configurable history limits to manage context size
- Clean separation between query processing and session state

### Error Handling
- Comprehensive exception handling throughout pipeline
- Graceful degradation when components fail
- Detailed logging for debugging document processing issues

## File Structure Significance

- `backend/app.py`: FastAPI application with CORS, static file serving, and API endpoints
- `backend/rag_system.py`: Core orchestration logic
- `backend/models.py`: Pydantic models for type safety (Course, Lesson, CourseChunk)
- `docs/`: Course material storage (auto-loaded on startup)
- `frontend/`: Static web interface (no build required)

## Dependencies

Core dependencies managed by `uv`:
- ChromaDB 1.0.15: Vector database
- Anthropic 0.58.2: Claude API client
- FastAPI 0.116.1: Web framework
- Sentence Transformers 5.0.0: Embedding generation
- Python 3.13+ required

## Startup Behavior

On server startup (`app.py:88`), the system automatically:
1. Scans `../docs` directory for course materials
2. Processes and indexes any new documents
3. Skips existing courses to prevent duplication
4. Reports loading statistics

This ensures the knowledge base is always current without manual intervention.

## Development Best Practices

- Always use `uv` to manage all dependencies
- Always use `uv` to run the server, do not use `pip` directly

## Development Memories

- Use `uv` to run any python files
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Course Materials RAG (Retrieval-Augmented Generation) System - a full-stack web application that enables users to query course materials and receive intelligent, context-aware responses. The system uses ChromaDB for vector storage, Anthropic's Claude AI for response generation, and provides a clean web interface for interaction.

## Key Dependencies & Environment

- **Python**: 3.13+ required
- **Package Manager**: Uses `uv` (modern Python package manager)
- **Main Dependencies**: ChromaDB, Anthropic API, sentence-transformers, FastAPI, uvicorn
- **Environment**: Requires `ANTHROPIC_API_KEY` in `.env` file

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# Add new dependencies
uv add <package-name>

# IMPORTANT: Always use uv for dependency management, never pip
```

### No Testing/Linting Commands Found
The codebase currently has no configured test runner, linting, or formatting commands in the project files.

## Architecture Overview

### Core Components Architecture
The system follows a modular RAG architecture with clear separation of concerns:

**1. RAGSystem (`rag_system.py`)** - Main orchestrator that coordinates all components
- Initializes and manages DocumentProcessor, VectorStore, AIGenerator, SessionManager, and ToolManager
- Provides unified interface for document ingestion and query processing
- Handles course analytics and management

**2. Document Processing Pipeline**
- `DocumentProcessor` (`document_processor.py`): Parses course documents with specific format expectations
- Extracts course metadata (title, instructor, link) from first 3 lines
- Processes lessons marked as "Lesson N: Title" format
- Creates text chunks with configurable size (800 chars) and overlap (100 chars)

**3. Vector Storage System**
- `VectorStore` (`vector_store.py`): ChromaDB-based semantic search
- Dual collections: `course_catalog` (metadata) and `course_content` (chunks)
- Supports course name resolution, lesson filtering, and semantic search
- Uses sentence-transformers for embeddings (all-MiniLM-L6-v2)

**4. AI Response Generation**
- `AIGenerator` (`ai_generator.py`): Anthropic Claude API integration
- Tool-calling capabilities for search integration
- Context-aware responses with conversation history
- Uses claude-sonnet-4-20250514 model

**5. Session & Tool Management**
- `SessionManager`: Conversation history with configurable limits
- `ToolManager` + `CourseSearchTool`: Structured tool calling for search operations

### Expected Document Format
Course documents must follow this structure:
```
Course Title: [Course Name]
Course Link: [Optional URL]
Course Instructor: [Instructor Name]

Lesson 0: Introduction
Lesson Link: [Optional URL]
[Lesson content...]

Lesson 1: Topic Name
[Lesson content...]
```

### Key Configuration Settings
Located in `config.py`:
- `CHUNK_SIZE`: 800 characters
- `CHUNK_OVERLAP`: 100 characters  
- `MAX_RESULTS`: 5 search results
- `MAX_HISTORY`: 2 conversation exchanges
- `EMBEDDING_MODEL`: "all-MiniLM-L6-v2"
- `ANTHROPIC_MODEL`: "claude-sonnet-4-20250514"

## File Structure Context

```
backend/           # Core Python backend
├── app.py        # FastAPI application & API endpoints
├── rag_system.py # Main RAG orchestrator
├── config.py     # Configuration settings
├── models.py     # Pydantic data models (Course, Lesson, CourseChunk)
├── document_processor.py  # Document parsing & chunking
├── vector_store.py       # ChromaDB vector operations
├── ai_generator.py       # Anthropic API integration
├── session_manager.py    # Conversation management
└── search_tools.py       # Tool calling system

frontend/         # Static web interface
├── index.html   # Main UI with chat interface
├── script.js    # Frontend JavaScript
└── style.css    # Styling

docs/            # Course materials storage
└── course*.txt  # Expected course documents

run.sh           # Application startup script
pyproject.toml   # Python dependencies
```

## API Endpoints

- `POST /api/query` - Submit questions and get AI responses with sources
- `GET /api/courses` - Get course statistics and available courses
- `GET /` - Serves the frontend web interface
- `GET /docs` - FastAPI auto-generated API documentation

## Development Notes

- The system automatically loads documents from `docs/` folder on startup
- Course documents are processed once and stored in ChromaDB for persistence
- Frontend includes suggested questions and course statistics sidebar
- Session IDs are auto-generated for conversation tracking
- CORS is configured to allow all origins for development
- Static files are served with no-cache headers for development
- use uv to add dependencies instead of pip
# YouTube Political Commentary Search Platform

## Overview
A searchable archive platform for a Portuguese political commentary YouTube channel. The system automatically downloads, transcribes (using local AI), and indexes years of daily livestream videos (typically broadcast at 8am on weekdays). Users can search through all transcriptions, ask questions about specific topics, and analyze how political discussions have evolved over time (e.g., "What did they say about topic X last month?").

**Key capability**: Search results link directly to the exact timestamp in the original YouTube video where the topic was mentioned, allowing users to instantly jump to the relevant moment and watch the full context.

## Problem Statement
This solves the problem of content discoverability and temporal analysis for long-running political commentary channels. Currently, finding what a commentator said about a specific topic requires manually watching hours of video or relying on unreliable memory. This platform enables:
- **Researchers and journalists** to quickly find historical commentary on specific topics and watch the exact moment it was discussed
- **Regular viewers** to catch up on missed discussions or revisit previous analyses with direct video links
- **Political analysts** to track how narratives and opinions evolved over time with timestamped evidence

By indexing both transcription text AND precise timestamps, users don't just find what was said - they can immediately watch it in context by clicking through to the exact moment in the original YouTube video.

The target audience is Portuguese political enthusiasts, journalists, researchers, and the general public interested in accessing this commentary archive.

## Feasibility Analysis

### Difficulty: Medium-Hard
This project has moderate complexity across multiple domains. YouTube video parsing and metadata extraction is straightforward using established tools like yt-dlp. Local Whisper transcription is well-documented and handles Portuguese well, but managing state for hundreds of hours of video transcription with resume capability requires careful database design and process orchestration. The hybrid search architecture (keyword + future semantic search with re-ranking) demands upfront planning to avoid costly refactoring. Building a public-facing web interface adds UI/UX complexity. The main unknowns are YouTube's rate limiting behavior when parsing large video catalogs and optimizing transcription throughput on consumer hardware.

### Estimated Time to MVP
4-8 weeks for a functional public platform

**Phase breakdown:**
- **Weeks 1-2**: YouTube video parsing pipeline + local Whisper transcription with resume capability and database state tracking
- **Weeks 2-4**: Database schema design, keyword search implementation (PostgreSQL FTS or SQLite FTS5), basic topic extraction
- **Weeks 4-6**: Web interface (search UI, results display, temporal filters, video playback with timestamp linking)
- **Weeks 6-8**: Refinement, deployment, performance optimization, documentation

The timeline assumes part-time work and accounts for the long-running nature of transcribing hundreds of hours of video content incrementally.

### Key Technical Challenges
- **YouTube blocking prevention**: Avoiding rate limits and IP blocks when downloading hundreds of videos. Requires intelligent throttling, randomized delays, and potentially rotating user agents. The two-phase approach (catalog URLs first, download/process later) mitigates this risk.
- **Video filtering/pattern matching**: Identifying the specific subset of videos (8am weekday livestreams) from a large channel catalog. Needs flexible pattern matching on titles, descriptions, upload times, and video metadata.
- **Transcription state management**: Tracking progress for hundreds of videos with graceful resume, error handling, and partial completion states
- **Long-running local jobs**: Designing a robust task queue system that handles computer restarts, job failures, and prioritization without complex infrastructure. Must support pausing and resuming multi-day transcription runs.
- **Efficient full-text indexing**: Optimizing database indexes for fast search across potentially millions of words of transcription text
- **Portuguese language NLP**: Ensuring quality keyword extraction, stemming, and future semantic embeddings work well for Portuguese political terminology
- **Extensible search architecture**: Designing the search layer to seamlessly integrate future semantic search and re-ranking without major refactoring
- **Topic evolution analysis**: Building queries that can aggregate and compare topic mentions across different time windows (weekly, monthly trends)
- **Timestamp synchronization**: Accurately linking transcription segments back to video timestamps for clickable search results

### Suggested Tech Stack

**Core Processing:**
- **Python 3.10+** - Primary language per user preference
- **yt-dlp** - YouTube video downloading and metadata extraction. Key advantages: no API quotas, built-in rate limiting (`--sleep-interval`, `--max-sleep-interval`), robust against blocking, supports pattern-based filtering, extracts detailed metadata without downloading full videos
- **Whisper (OpenAI)** - Local speech-to-text transcription with excellent Portuguese support (whisper-large or whisper-medium models). Transcriptions are chunked into ~10-second segments for precise timestamp linking.
- **ffmpeg** - Audio extraction and preprocessing for Whisper
- **tenacity** or **backoff** - Retry logic with exponential backoff for download failures

**Data Storage:**
- **PostgreSQL 14+** with full-text search extensions - Robust FTS, JSON support for metadata, and easy upgrade path to pgvector for future semantic search
- **SQLAlchemy** - ORM for clean database interactions
- **Alembic** - Database migrations

**Backend API:**
- **FastAPI** - Modern async Python web framework, auto-generates API docs, excellent for building RESTful search endpoints
- **Pydantic** - Data validation and serialization

**Task Queue:**
- **Huey** or **RQ (Redis Queue)** - Lightweight task queue for managing transcription jobs with retry logic (simpler than Celery for this use case)
- **Redis** - Task queue backend and caching layer

**Search & Analysis:**
- **PostgreSQL Full-Text Search** - Phase 1 keyword search with ranking
- **spaCy (Portuguese model)** - Future topic extraction and NLP
- **sentence-transformers** - Future semantic embeddings (when adding hybrid search)
- **pgvector** - Future vector similarity search in Postgres

**Frontend:**
- **React + Vite** or **SvelteKit** - Modern reactive UI for search interface
- **TailwindCSS** - Rapid UI styling
- **React Query / SWR** - Data fetching and caching

**Deployment:**
- **Docker + Docker Compose** - Containerization for easy deployment
- **Nginx** - Reverse proxy and static file serving
- **DigitalOcean / Hetzner / Railway** - Cost-effective hosting options for public platform

**Development Tools:**
- **pytest** - Testing framework
- **Black + Ruff** - Code formatting and linting
- **pre-commit** - Git hooks for code quality

**Architecture Notes:**
The system is designed as a three-stage pipeline: (1) Video fetching/cataloging, (2) Transcription processing with state management, (3) Search indexing and API serving. Each stage can run independently, allowing incremental processing.

The database schema should include:
- **Videos**: YouTube metadata (video ID, URL, title, publish date, duration, channel info, processing status)
- **TranscriptionSegments**: ~10-second timestamped chunks (video_id, start_time, end_time, text). Whisper provides word-level timestamps which are aggregated into segments at sentence boundaries, targeting ~10 seconds per segment. This provides precise timestamp linking (~360 segments/hour of video, ~25MB per 100 hours) while keeping storage manageable.
- **ProcessingJobs**: State tracking for resume capability (video_id, job_type, status, progress, error_logs, last_updated)

Search results return matching segments with their timestamps, which the frontend renders as clickable YouTube links with `?t=XXs` timestamp parameters. The search layer abstracts the underlying search implementation, making it straightforward to add semantic search or hybrid approaches later without touching the API contract.

## Status
Registered on 2026-02-14

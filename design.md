# Design Document: Bharat Setu

## Overview

Bharat Setu is a voice-first conversational AI platform that democratizes access to government schemes, scholarships, internships, and public systems. The platform uses a three-tier architecture consisting of a responsive frontend, a scalable backend API layer, and an AI intelligence layer powered by Gemini LLM and Speech-to-Text services.

The design prioritizes accessibility, performance, and scalability while maintaining security and data privacy. The system is built to handle varying network conditions through a low-bandwidth mode and provides both voice and text input modalities to ensure universal access.

### Key Design Principles

1. **Voice-First, Not Voice-Only**: Voice input is the primary interaction mode, but text input is always available as a fallback
2. **Progressive Enhancement**: Core functionality works on basic connections; enhanced features activate on better networks
3. **Modular Architecture**: Independent services that can scale and deploy separately
4. **Security by Design**: Encryption, input validation, and secure credential management built into every layer
5. **User-Centric Language**: AI responses optimized for simplicity and clarity

## Architecture

### High-Level Architecture


```mermaid
graph TB
    subgraph "Client Layer"
        UI[Web Interface<br/>HTML5/CSS3/JS]
        MIC[Microphone Input]
        TEXT[Text Input]
    end
    
    subgraph "API Gateway Layer"
        GATEWAY[API Gateway<br/>Rate Limiting & Auth]
    end
    
    subgraph "Backend Services"
        API[FastAPI Backend<br/>Python]
        CACHE[Response Cache<br/>Redis]
    end
    
    subgraph "AI Intelligence Layer"
        LLM[Gemini LLM API<br/>Query Processing]
        STT[Speech-to-Text<br/>Voice Conversion]
        KB[Knowledge Base<br/>Scheme Data]
    end
    
    subgraph "Data Layer"
        DB[(Database<br/>User Profiles & Logs)]
        STORAGE[File Storage<br/>Audio Clips]
    end
    
    UI --> GATEWAY
    MIC --> STT
    TEXT --> GATEWAY
    GATEWAY --> API
    API --> CACHE
    API --> LLM
    API --> KB
    API --> DB
    STT --> GATEWAY
    LLM --> KB
    API --> STORAGE
```

### Architecture Layers

**Client Layer**: Responsive web interface supporting voice and text input
**API Gateway**: Request routing, rate limiting, authentication, and load balancing
**Backend Services**: FastAPI application handling business logic and orchestration
**AI Intelligence Layer**: LLM processing, STT conversion, and knowledge base management
**Data Layer**: Persistent storage for user data, logs, and audio files



## Components and Interfaces

### Frontend Component

**Technology**: HTML5, CSS3, Vanilla JavaScript

**Responsibilities**:
- Render responsive user interface for mobile and desktop
- Capture voice input through Web Audio API
- Provide text input interface with real-time validation
- Display AI responses with proper formatting
- Manage session state and conversation history
- Implement low-bandwidth mode UI optimizations
- Handle network errors and provide user feedback

**Key Interfaces**:

```javascript
// Query submission interface
interface QueryRequest {
  query: string;
  sessionId: string;
  inputMode: 'voice' | 'text';
  userProfile?: UserProfile;
  lowBandwidthMode: boolean;
}

// Response interface
interface QueryResponse {
  response: string;
  sources?: string[];
  relatedSchemes?: Scheme[];
  clarificationNeeded: boolean;
  sessionId: string;
}

// Voice recording interface
interface VoiceRecording {
  audioBlob: Blob;
  duration: number;
  language: 'en' | 'hi';
}
```



### Backend API Component

**Technology**: Python with FastAPI framework

**Responsibilities**:
- Receive and validate incoming requests
- Orchestrate calls to AI services (LLM, STT)
- Query knowledge base for scheme information
- Manage user sessions and conversation context
- Implement rate limiting and security controls
- Cache frequent responses for performance
- Log requests and errors for monitoring
- Handle low-bandwidth mode optimizations

**Key Endpoints**:

```python
# Query processing endpoint
POST /api/v1/query
Request: {
  "query": str,
  "session_id": str,
  "input_mode": "voice" | "text",
  "user_profile": Optional[UserProfile],
  "low_bandwidth": bool
}
Response: {
  "response": str,
  "sources": List[str],
  "related_schemes": List[Scheme],
  "clarification_needed": bool,
  "session_id": str
}

# Voice transcription endpoint
POST /api/v1/transcribe
Request: multipart/form-data with audio file
Response: {
  "transcription": str,
  "confidence": float,
  "language": str
}

# Opportunity finder endpoint
POST /api/v1/opportunities/find
Request: {
  "user_profile": UserProfile,
  "filters": Optional[Filters]
}
Response: {
  "opportunities": List[Scheme],
  "total_count": int,
  "relevance_scores": List[float]
}

# Health check endpoint
GET /api/v1/health
Response: {
  "status": "healthy" | "degraded" | "unhealthy",
  "services": {
    "llm": bool,
    "stt": bool,
    "database": bool
  }
}
```



### AI Intelligence Component

**LLM Service (Gemini API)**:

**Responsibilities**:
- Process natural language queries
- Generate conversational responses
- Maintain conversation context
- Simplify complex information
- Provide step-by-step guidance
- Handle clarification requests

**Integration Pattern**:
```python
class LLMService:
    def __init__(self, api_key: str, model: str = "gemini-pro"):
        self.api_key = api_key
        self.model = model
    
    def process_query(
        self,
        query: str,
        context: ConversationContext,
        knowledge_base: KnowledgeBase
    ) -> LLMResponse:
        # Build prompt with system instructions
        prompt = self._build_prompt(query, context, knowledge_base)
        
        # Call Gemini API
        response = self._call_gemini_api(prompt)
        
        # Post-process for simplicity and clarity
        simplified = self._simplify_language(response)
        
        return LLMResponse(
            text=simplified,
            sources=self._extract_sources(response),
            confidence=response.confidence
        )
```

**STT Service (Speech-to-Text)**:

**Responsibilities**:
- Convert audio to text
- Support English and Hindi languages
- Handle background noise
- Provide confidence scores
- Detect language automatically

**Integration Pattern**:
```python
class STTService:
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    def transcribe(
        self,
        audio_data: bytes,
        language: str = "auto"
    ) -> TranscriptionResult:
        # Validate audio format and duration
        validated_audio = self._validate_audio(audio_data)
        
        # Call STT API
        result = self._call_stt_api(validated_audio, language)
        
        return TranscriptionResult(
            text=result.text,
            confidence=result.confidence,
            detected_language=result.language
        )
```



### Knowledge Base Component

**Responsibilities**:
- Store structured information about government schemes
- Provide fast retrieval of scheme details
- Support filtering and ranking by relevance
- Update scheme information periodically
- Maintain scheme metadata (deadlines, eligibility, etc.)

**Data Structure**:
```python
class Scheme:
    id: str
    name: str
    description: str
    category: str  # scholarship, internship, employment, skill_development
    eligibility_criteria: List[EligibilityCriterion]
    benefits: List[str]
    application_process: List[Step]
    deadline: Optional[datetime]
    official_link: str
    source_department: str
    last_updated: datetime
    keywords: List[str]

class EligibilityCriterion:
    criterion_type: str  # age, education, income, location, category
    value: Any
    operator: str  # >=, <=, ==, in, etc.

class Step:
    step_number: int
    description: str
    required_documents: List[str]
    estimated_time: str
```

**Retrieval Interface**:
```python
class KnowledgeBase:
    def search_schemes(
        self,
        query: str,
        user_profile: Optional[UserProfile] = None,
        filters: Optional[Filters] = None
    ) -> List[Scheme]:
        # Perform semantic search using embeddings
        # Filter by eligibility if user_profile provided
        # Apply additional filters
        # Rank by relevance
        pass
    
    def get_scheme_by_id(self, scheme_id: str) -> Optional[Scheme]:
        pass
    
    def get_schemes_by_category(self, category: str) -> List[Scheme]:
        pass
```



## Data Models

### User Profile Model

```python
class UserProfile:
    user_id: str
    age: Optional[int]
    education_level: Optional[str]  # 10th, 12th, undergraduate, postgraduate
    location: Optional[Location]
    category: Optional[str]  # general, obc, sc, st, ews
    annual_income: Optional[float]
    interests: List[str]
    language_preference: str  # en, hi, hinglish
    created_at: datetime
    updated_at: datetime

class Location:
    state: str
    district: Optional[str]
    is_rural: bool
```

### Session Model

```python
class Session:
    session_id: str
    user_id: Optional[str]
    conversation_history: List[Message]
    context: Dict[str, Any]
    created_at: datetime
    last_activity: datetime
    low_bandwidth_mode: bool

class Message:
    role: str  # user, assistant
    content: str
    timestamp: datetime
    input_mode: Optional[str]  # voice, text
    metadata: Dict[str, Any]
```

### Query Log Model

```python
class QueryLog:
    log_id: str
    session_id: str
    query: str
    response: str
    input_mode: str
    response_time_ms: int
    llm_tokens_used: int
    user_feedback: Optional[Feedback]
    timestamp: datetime

class Feedback:
    rating: int  # 1-5
    was_helpful: bool
    reported_issue: Optional[str]
```



## Data Flow

### Voice Query Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant STT
    participant LLM
    participant KB
    
    User->>Frontend: Click microphone button
    Frontend->>Frontend: Start audio recording
    User->>Frontend: Speak query
    Frontend->>Frontend: Stop recording
    Frontend->>API: POST /api/v1/transcribe (audio)
    API->>STT: Send audio for transcription
    STT-->>API: Return transcribed text
    API-->>Frontend: Return transcription
    Frontend->>Frontend: Display transcription for confirmation
    User->>Frontend: Confirm/Edit transcription
    Frontend->>API: POST /api/v1/query (text)
    API->>KB: Search relevant schemes
    KB-->>API: Return matching schemes
    API->>LLM: Process query with context
    LLM-->>API: Return AI response
    API->>API: Log query and response
    API-->>Frontend: Return formatted response
    Frontend->>User: Display response
```

### Text Query Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Cache
    participant LLM
    participant KB
    
    User->>Frontend: Type query
    Frontend->>API: POST /api/v1/query
    API->>Cache: Check for cached response
    alt Cache hit
        Cache-->>API: Return cached response
        API-->>Frontend: Return response
    else Cache miss
        API->>KB: Search relevant schemes
        KB-->>API: Return matching schemes
        API->>LLM: Process query with context
        LLM-->>API: Return AI response
        API->>Cache: Store response
        API->>API: Log query
        API-->>Frontend: Return response
    end
    Frontend->>User: Display response
```

### Opportunity Finder Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant KB
    
    User->>Frontend: Provide profile information
    Frontend->>API: POST /api/v1/opportunities/find
    API->>API: Validate profile data
    API->>KB: Query schemes with filters
    KB->>KB: Filter by eligibility
    KB->>KB: Rank by relevance
    KB-->>API: Return ranked opportunities
    API->>API: Enrich with metadata
    API-->>Frontend: Return opportunity list
    Frontend->>User: Display opportunities
    User->>Frontend: Select opportunity
    Frontend->>API: GET /api/v1/schemes/{id}
    API->>KB: Get scheme details
    KB-->>API: Return full scheme info
    API-->>Frontend: Return details
    Frontend->>User: Display detailed view
```



## Voice-First Interaction Design

### Voice Input Design

**Microphone Button States**:
- **Idle**: Gray microphone icon, clickable
- **Recording**: Red pulsing icon with waveform animation
- **Processing**: Spinner animation while transcribing
- **Error**: Red icon with error message

**User Feedback Mechanisms**:
- Visual recording indicator (pulsing animation)
- Real-time audio level meter
- Transcription preview before submission
- Edit capability for incorrect transcriptions
- Clear error messages for failed recordings

**Voice Input Optimization**:
- Automatic silence detection to stop recording
- Background noise filtering (client-side pre-processing)
- Audio compression before upload (reduce bandwidth)
- Maximum recording duration: 60 seconds
- Automatic fallback to text on repeated failures

### Text Input Design

**Always-Available Text Input**:
- Text input field visible at all times
- Keyboard shortcuts (Enter to submit)
- Character counter for long queries
- Auto-resize textarea for multi-line input
- Clear button to reset input

**Hybrid Interaction**:
- Users can switch between voice and text mid-conversation
- Voice transcription can be edited as text before submission
- Copy-paste support for complex queries



## Security Design

### Authentication and Authorization

**API Authentication**:
- API key-based authentication for service-to-service calls
- JWT tokens for user sessions (if user accounts implemented)
- Rate limiting per IP address and session
- CORS configuration for allowed origins

**Rate Limiting Strategy**:
```python
# Rate limit configuration
RATE_LIMITS = {
    "query": "100/hour",  # 100 queries per hour per user
    "transcribe": "50/hour",  # 50 voice transcriptions per hour
    "opportunities": "200/hour"  # 200 opportunity searches per hour
}

# Implementation using token bucket algorithm
class RateLimiter:
    def check_limit(self, user_id: str, endpoint: str) -> bool:
        # Check if user has exceeded rate limit
        # Return True if allowed, False if exceeded
        pass
```

### Input Validation and Sanitization

**Query Validation**:
- Maximum query length: 500 characters
- Sanitize HTML and script tags
- Validate audio file format (WAV, MP3, OGG)
- Maximum audio file size: 5MB
- Validate user profile data types

**SQL Injection Prevention**:
- Use parameterized queries for all database operations
- ORM-based database access (SQLAlchemy)
- Input validation before database queries

**XSS Prevention**:
- Escape all user-generated content before rendering
- Content Security Policy (CSP) headers
- Sanitize LLM responses before display

### Data Encryption

**Encryption in Transit**:
- TLS 1.3 for all API communications
- HTTPS enforcement (redirect HTTP to HTTPS)
- Secure WebSocket connections for real-time features

**Encryption at Rest**:
- Encrypt sensitive user profile data
- Encrypt audio files in storage
- Use environment variables for API keys and secrets
- Secure secret management (AWS Secrets Manager or similar)

### Privacy and Compliance

**Data Minimization**:
- Collect only necessary user information
- Anonymous sessions by default
- Optional user accounts for personalization

**Data Retention**:
- Query logs retained for 90 days
- Audio files deleted after transcription (unless user opts to save)
- User profiles deleted on account closure

**Audit Logging**:
- Log all API requests with timestamps
- Log authentication attempts
- Log rate limit violations
- Monitor for suspicious patterns



## Scalability Design

### Horizontal Scaling Strategy

**Stateless API Design**:
- All backend services are stateless
- Session data stored in Redis (shared state)
- No server-side session affinity required
- Load balancer can distribute requests to any instance

**Service Scaling**:
```yaml
# Docker Compose scaling configuration
services:
  api:
    image: bharat-setu-api:latest
    deploy:
      replicas: 3  # Start with 3 instances
      resources:
        limits:
          cpus: '1'
          memory: 512M
    environment:
      - REDIS_URL=redis://cache:6379
      - DATABASE_URL=postgresql://db:5432/bharat_setu
```

**Auto-Scaling Triggers**:
- CPU utilization > 70% for 5 minutes
- Memory utilization > 80%
- Request queue depth > 100
- Response time > 10 seconds

### Caching Strategy

**Response Caching**:
- Cache frequent queries (TTL: 1 hour)
- Cache scheme information (TTL: 24 hours)
- Cache LLM responses for identical queries (TTL: 6 hours)
- Invalidate cache on knowledge base updates

**Cache Implementation**:
```python
class CacheService:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def get_cached_response(self, query_hash: str) -> Optional[str]:
        return self.redis.get(f"query:{query_hash}")
    
    def cache_response(self, query_hash: str, response: str, ttl: int = 3600):
        self.redis.setex(f"query:{query_hash}", ttl, response)
    
    def invalidate_scheme_cache(self):
        self.redis.delete("schemes:*")
```

### Database Optimization

**Indexing Strategy**:
- Index on scheme keywords for fast search
- Index on eligibility criteria for filtering
- Index on session_id for conversation retrieval
- Composite index on (category, deadline) for opportunity finder

**Query Optimization**:
- Use database connection pooling
- Implement read replicas for query-heavy operations
- Batch database writes for logs
- Use database-level caching (PostgreSQL shared buffers)



## Low-Bandwidth Mode Strategy

### Bandwidth Reduction Techniques

**Frontend Optimizations**:
- Disable voice input (text-only mode)
- Remove animations and transitions
- Compress images to lower quality
- Lazy load non-critical content
- Minify and compress JavaScript/CSS
- Use system fonts instead of web fonts

**Backend Optimizations**:
- Reduce response payload size
- Remove optional fields from API responses
- Compress responses with gzip
- Limit conversation history to last 3 messages
- Disable related schemes suggestions
- Return plain text instead of formatted markdown

**Implementation**:
```python
def format_response_for_bandwidth(
    response: QueryResponse,
    low_bandwidth: bool
) -> QueryResponse:
    if not low_bandwidth:
        return response
    
    # Reduce response size
    simplified = QueryResponse(
        response=response.response[:500],  # Truncate to 500 chars
        sources=None,  # Remove sources
        related_schemes=None,  # Remove suggestions
        clarification_needed=response.clarification_needed,
        session_id=response.session_id
    )
    
    return simplified
```

### Network Detection

**Automatic Detection**:
```javascript
// Frontend network detection
function detectNetworkSpeed() {
    if ('connection' in navigator) {
        const connection = navigator.connection;
        const effectiveType = connection.effectiveType;
        
        // Suggest low-bandwidth mode for slow connections
        if (effectiveType === 'slow-2g' || effectiveType === '2g') {
            return 'slow';
        }
    }
    return 'normal';
}

// Measure actual response time
function measureResponseTime(startTime) {
    const duration = Date.now() - startTime;
    if (duration > 5000) {  // > 5 seconds
        suggestLowBandwidthMode();
    }
}
```

### Progressive Enhancement

**Core Features (Always Available)**:
- Text input and output
- Basic query processing
- Scheme information retrieval

**Enhanced Features (Disabled in Low-Bandwidth)**:
- Voice input
- Rich formatting and styling
- Images and icons
- Related suggestions
- Conversation history beyond 3 messages



## API Design Overview

### RESTful API Principles

**Base URL**: `https://api.bharatsetu.in/v1`

**Authentication**: API key in header
```
Authorization: Bearer <api_key>
```

**Standard Response Format**:
```json
{
  "success": true,
  "data": { ... },
  "error": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Error Response Format**:
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "INVALID_INPUT",
    "message": "Query text is required",
    "details": { ... }
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### API Endpoints Summary

| Endpoint | Method | Purpose | Rate Limit |
|----------|--------|---------|------------|
| `/query` | POST | Process text/voice query | 100/hour |
| `/transcribe` | POST | Convert voice to text | 50/hour |
| `/opportunities/find` | POST | Find matching schemes | 200/hour |
| `/schemes/{id}` | GET | Get scheme details | 500/hour |
| `/schemes/search` | GET | Search schemes | 300/hour |
| `/health` | GET | Service health check | Unlimited |
| `/feedback` | POST | Submit user feedback | 50/hour |

### Versioning Strategy

- API version in URL path (`/v1/`, `/v2/`)
- Maintain backward compatibility for at least 6 months
- Deprecation warnings in response headers
- Clear migration documentation for version changes



## Deployment Strategy

### Containerization

**Docker Configuration**:
```dockerfile
# Dockerfile for FastAPI backend
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Docker Compose for Local Development**:
```yaml
version: '3.8'

services:
  api:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - GEMINI_API_KEY=${GEMINI_API_KEY}
      - DATABASE_URL=postgresql://postgres:password@db:5432/bharat_setu
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
  
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=bharat_setu
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - api

volumes:
  postgres_data:
```

### Cloud Deployment Options

**Option 1: AWS EC2**
- Deploy Docker containers on EC2 instances
- Use Application Load Balancer for traffic distribution
- RDS PostgreSQL for database
- ElastiCache Redis for caching
- S3 for audio file storage
- CloudWatch for monitoring

**Option 2: Kiro Platform**
- Platform-specific deployment configuration
- Managed services for database and caching
- Built-in monitoring and logging

### CI/CD Pipeline

**Deployment Workflow**:
```yaml
# GitHub Actions workflow
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: |
          pip install -r requirements.txt
          pytest tests/
  
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker image
        run: docker build -t bharat-setu-api:latest .
      - name: Push to registry
        run: docker push bharat-setu-api:latest
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to EC2
        run: |
          ssh user@server 'docker pull bharat-setu-api:latest'
          ssh user@server 'docker-compose up -d'
```

### Environment Configuration

**Environment Variables**:
```bash
# API Keys
GEMINI_API_KEY=<gemini_api_key>
STT_API_KEY=<stt_api_key>

# Database
DATABASE_URL=postgresql://user:pass@host:5432/db
REDIS_URL=redis://host:6379

# Security
JWT_SECRET=<random_secret>
API_KEY_SALT=<random_salt>

# Rate Limiting
RATE_LIMIT_ENABLED=true
MAX_REQUESTS_PER_HOUR=100

# Feature Flags
LOW_BANDWIDTH_MODE_ENABLED=true
VOICE_INPUT_ENABLED=true
```

### Monitoring and Logging

**Logging Strategy**:
- Structured JSON logging
- Log levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
- Centralized log aggregation (CloudWatch, ELK stack)
- Request/response logging with correlation IDs

**Monitoring Metrics**:
- API response times (p50, p95, p99)
- Error rates by endpoint
- LLM API usage and costs
- Database query performance
- Cache hit rates
- Active user sessions

**Alerting**:
- Alert on error rate > 5%
- Alert on response time > 10 seconds
- Alert on service downtime
- Alert on rate limit violations spike



## Correctness Properties

A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

The following properties define the correctness criteria for Bharat Setu. Each property is universally quantified and references the specific requirements it validates.

### Query Processing Properties

**Property 1: Response time performance**
*For any* valid user query, the system should return a response within 5 seconds of submission.
**Validates: Requirements 1.1**

**Property 2: Response completeness for scheme queries**
*For any* query about a specific government scheme, the AI response should contain all required information fields: scheme name, eligibility criteria, benefits, and application process steps.
**Validates: Requirements 1.2, 3.2, 3.3**

**Property 3: Clarification for ambiguous queries**
*For any* intentionally vague or ambiguous query (lacking key context like location, age, or category), the system should respond with clarifying questions rather than making assumptions.
**Validates: Requirements 1.3**

**Property 4: Step-by-step guidance structure**
*For any* query requesting guidance or "how to" information, the response should contain numbered sequential steps with clear action items.
**Validates: Requirements 1.4**

**Property 5: Session context preservation**
*For any* sequence of related queries within a session, the system should maintain context such that follow-up queries can reference previous conversation elements without repetition.
**Validates: Requirements 1.5**

**Property 6: Graceful handling of unanswerable queries**
*For any* query that cannot be answered with available knowledge base data, the system should acknowledge the limitation and suggest alternative resources or actions.
**Validates: Requirements 1.6**

### Voice Input Properties

**Property 7: STT accuracy threshold**
*For any* clear audio sample (signal-to-noise ratio > 20dB) in supported languages, the speech-to-text conversion should achieve at least 85% word-level accuracy.
**Validates: Requirements 2.2**

**Property 8: Language support for voice**
*For any* audio input in English or Hindi, the STT service should successfully transcribe the speech and correctly identify the language.
**Validates: Requirements 2.5**

**Property 9: Voice input fallback**
*For any* voice input failure (timeout, error, or unavailability), the system should automatically enable and display the text input interface without requiring user action.
**Validates: Requirements 2.6, 9.3**

**Property 10: Noise detection and feedback**
*For any* audio sample with high background noise (signal-to-noise ratio < 10dB), the system should detect the interference and notify the user to retry in a quieter environment.
**Validates: Requirements 2.7**

### Opportunity Finder Properties

**Property 11: Eligibility-based filtering**
*For any* user profile with specified attributes (age, education, location, category), all returned opportunities should match the user's eligibility criteria based on the scheme requirements.
**Validates: Requirements 3.1**

**Property 12: Category filtering accuracy**
*For any* category filter selection (scholarship, internship, skill development, employment), all returned opportunities should belong exclusively to that category.
**Validates: Requirements 3.6**

**Property 13: Relevance-based ranking**
*For any* set of opportunities returned for a user profile, the opportunities should be ordered by descending relevance score (number of matching criteria).
**Validates: Requirements 3.4**

### Language and Simplification Properties

**Property 14: Reading level compliance**
*For any* AI-generated response, the text should meet 10th-grade reading level or below as measured by Flesch-Kincaid grade level metric.
**Validates: Requirements 5.1**

**Property 15: Language simplicity and jargon avoidance**
*For any* response containing technical or bureaucratic terms, the system should either provide definitions in simple language or use simpler alternatives, and overall jargon usage should be minimal (< 5% of words).
**Validates: Requirements 5.3, 5.4, 4.4**

**Property 16: Language preference persistence**
*For any* session where a user selects a language preference (English, Hindi, Hinglish), all subsequent responses in that session should maintain the selected language.
**Validates: Requirements 5.5**

**Property 17: Process breakdown for complexity**
*For any* query about complex government processes or procedures, the response should break down the information into discrete steps (minimum 2 steps for complex processes).
**Validates: Requirements 4.2**

**Property 18: Public system knowledge coverage**
*For any* query about common public systems (Aadhaar, PAN, voter registration, ration cards, passport, driving license), the system should provide a relevant response rather than indicating lack of knowledge.
**Validates: Requirements 4.5**

### Security Properties

**Property 19: Input sanitization**
*For any* user input containing potentially malicious content (HTML tags, script tags, SQL injection patterns), the system should sanitize the input before processing, removing or escaping dangerous characters.
**Validates: Requirements 6.2**

**Property 20: PII protection in logs**
*For any* query log entry, the logged data should not contain personally identifiable information (email addresses, phone numbers, Aadhaar numbers, names) unless explicitly consented.
**Validates: Requirements 6.5**

**Property 21: Stateless API design**
*For any* API endpoint, the request should be processable without relying on server-side session state stored in memory, enabling any backend instance to handle any request.
**Validates: Requirements 7.5**

**Property 22: Error logging completeness**
*For any* error or exception that occurs during request processing, the system should log the error with timestamp, error type, stack trace, and request context.
**Validates: Requirements 7.7**

### Low-Bandwidth Mode Properties

**Property 23: Bandwidth reduction effectiveness**
*For any* query processed in low-bandwidth mode, the response payload size should be at least 60% smaller than the equivalent response in normal mode.
**Validates: Requirements 8.1**

**Property 24: Low-bandwidth content optimization**
*For any* response in low-bandwidth mode, the payload should exclude non-essential elements (images, animations, related suggestions, extended conversation history beyond 3 messages).
**Validates: Requirements 8.2, 8.3**

**Property 25: Information preservation in compression**
*For any* response compressed for low-bandwidth mode, the core information (answer to the query, key facts, action items) should be preserved even if supporting details are removed.
**Validates: Requirements 8.3**

### Text Input Fallback Properties

**Property 26: Processing consistency across input modes**
*For any* query submitted via text input versus voice input (after transcription), the AI processing and response generation should use identical logic and produce equivalent responses.
**Validates: Requirements 9.5**

### Response Quality Properties

**Property 27: Source citation inclusion**
*For any* response providing factual information about schemes or government systems, the response should include at least one source citation or link to an official government portal.
**Validates: Requirements 10.1, 10.3**

**Property 28: Disclaimer for uncertain information**
*For any* response where the system's confidence score is below a threshold (< 0.7) or information cannot be verified, the response should include an appropriate disclaimer about accuracy limitations.
**Validates: Requirements 10.4**

**Property 29: Feedback tracking**
*For any* user feedback submission (rating, helpful/not helpful, reported issue), the system should store the feedback linked to the corresponding query and response for analysis.
**Validates: Requirements 10.5, 10.6**



## Error Handling

### Error Categories

**Client Errors (4xx)**:
- `400 Bad Request`: Invalid input format, missing required fields
- `401 Unauthorized`: Missing or invalid API key
- `429 Too Many Requests`: Rate limit exceeded
- `413 Payload Too Large`: Audio file exceeds size limit

**Server Errors (5xx)**:
- `500 Internal Server Error`: Unexpected server error
- `502 Bad Gateway`: External service (LLM, STT) unavailable
- `503 Service Unavailable`: System overloaded or maintenance
- `504 Gateway Timeout`: External service timeout

### Error Handling Strategies

**LLM API Failures**:
```python
def handle_llm_error(error: Exception) -> QueryResponse:
    if isinstance(error, RateLimitError):
        # Use cached response if available
        cached = cache.get_similar_query()
        if cached:
            return cached
        # Otherwise return fallback response
        return QueryResponse(
            response="I'm experiencing high demand right now. Please try again in a moment.",
            error_code="RATE_LIMIT_EXCEEDED"
        )
    
    elif isinstance(error, TimeoutError):
        return QueryResponse(
            response="The request is taking longer than expected. Please try a simpler query.",
            error_code="TIMEOUT"
        )
    
    else:
        # Log error and return generic message
        logger.error(f"LLM API error: {error}")
        return QueryResponse(
            response="I'm having trouble processing your request. Please try again.",
            error_code="LLM_ERROR"
        )
```

**STT Service Failures**:
```python
def handle_stt_error(error: Exception) -> TranscriptionResult:
    if isinstance(error, AudioFormatError):
        return TranscriptionResult(
            error="Unsupported audio format. Please use WAV, MP3, or OGG.",
            error_code="INVALID_FORMAT"
        )
    
    elif isinstance(error, AudioTooLongError):
        return TranscriptionResult(
            error="Audio is too long. Maximum duration is 60 seconds.",
            error_code="AUDIO_TOO_LONG"
        )
    
    elif isinstance(error, LowQualityAudioError):
        return TranscriptionResult(
            error="Audio quality is too low. Please record in a quieter environment.",
            error_code="LOW_QUALITY"
        )
    
    else:
        logger.error(f"STT error: {error}")
        return TranscriptionResult(
            error="Voice transcription failed. Please try text input instead.",
            error_code="STT_ERROR"
        )
```

**Database Failures**:
```python
def handle_database_error(error: Exception, operation: str):
    logger.error(f"Database error during {operation}: {error}")
    
    # Attempt retry with exponential backoff
    for attempt in range(3):
        try:
            time.sleep(2 ** attempt)
            return retry_operation(operation)
        except Exception as e:
            if attempt == 2:
                # Final attempt failed
                raise ServiceUnavailableError(
                    "Database temporarily unavailable. Please try again later."
                )
```

### Graceful Degradation

**Feature Degradation Hierarchy**:
1. **Core Features** (Always Available):
   - Text input and output
   - Basic query processing
   - Scheme information retrieval

2. **Enhanced Features** (Degrade Gracefully):
   - Voice input → Fallback to text
   - Related suggestions → Omit if slow
   - Conversation history → Limit to recent messages
   - Rich formatting → Plain text

3. **Optional Features** (Disable on Failure):
   - Notifications
   - User profiles
   - Feedback collection

**Implementation**:
```python
class FeatureFlags:
    def __init__(self):
        self.voice_enabled = True
        self.suggestions_enabled = True
        self.history_enabled = True
    
    def check_service_health(self):
        # Disable features based on service health
        if not stt_service.is_healthy():
            self.voice_enabled = False
        
        if response_time > 5000:  # ms
            self.suggestions_enabled = False
        
        if memory_usage > 0.9:
            self.history_enabled = False
```

### User-Facing Error Messages

**Principles**:
- Clear and actionable
- Non-technical language
- Suggest next steps
- Maintain friendly tone

**Examples**:
- ❌ "LLM API returned 429"
- ✅ "I'm getting a lot of questions right now. Please try again in a minute."

- ❌ "STT confidence score below threshold"
- ✅ "I couldn't quite catch that. Could you try speaking more clearly or use text input?"

- ❌ "Database connection pool exhausted"
- ✅ "We're experiencing high traffic. Please try again in a moment."



## Testing Strategy

### Dual Testing Approach

Bharat Setu employs a comprehensive testing strategy combining unit tests and property-based tests. Both approaches are complementary and necessary for ensuring system correctness:

- **Unit tests**: Verify specific examples, edge cases, error conditions, and integration points
- **Property-based tests**: Verify universal properties across randomized inputs for comprehensive coverage

### Unit Testing

**Scope**: Unit tests focus on specific scenarios and edge cases that demonstrate correct behavior.

**Test Categories**:

1. **API Endpoint Tests**:
   - Test each endpoint with valid inputs
   - Test error responses for invalid inputs
   - Test authentication and authorization
   - Test rate limiting behavior

2. **Integration Tests**:
   - Test frontend-backend communication
   - Test backend-LLM integration
   - Test backend-STT integration
   - Test database operations

3. **Edge Case Tests**:
   - Empty or whitespace-only queries
   - Extremely long queries (> 500 characters)
   - Special characters and Unicode
   - Concurrent requests from same user
   - Network timeout scenarios

4. **Security Tests**:
   - XSS attack attempts
   - SQL injection attempts
   - CSRF protection
   - Rate limit bypass attempts

**Example Unit Tests**:
```python
def test_query_endpoint_with_valid_input():
    response = client.post("/api/v1/query", json={
        "query": "What scholarships are available for engineering students?",
        "session_id": "test-session-123",
        "input_mode": "text",
        "low_bandwidth": False
    })
    assert response.status_code == 200
    assert "response" in response.json()

def test_query_endpoint_rejects_empty_query():
    response = client.post("/api/v1/query", json={
        "query": "",
        "session_id": "test-session-123"
    })
    assert response.status_code == 400
    assert "error" in response.json()

def test_rate_limit_enforcement():
    # Make 100 requests (at limit)
    for i in range(100):
        response = client.post("/api/v1/query", json={
            "query": f"Test query {i}",
            "session_id": "test-session-123"
        })
        assert response.status_code == 200
    
    # 101st request should be rejected
    response = client.post("/api/v1/query", json={
        "query": "Test query 101",
        "session_id": "test-session-123"
    })
    assert response.status_code == 429
```

### Property-Based Testing

**Scope**: Property-based tests verify that universal properties hold across all valid inputs through randomized testing.

**Configuration**:
- Minimum 100 iterations per property test
- Each test references its design document property
- Tag format: `Feature: bharat-setu, Property {number}: {property_text}`

**Property Test Framework**: Use `hypothesis` for Python backend tests

**Example Property Tests**:

```python
from hypothesis import given, strategies as st
import pytest

# Property 1: Response time performance
@given(query=st.text(min_size=1, max_size=500))
@pytest.mark.property_test
def test_property_1_response_time(query):
    """
    Feature: bharat-setu, Property 1: Response time performance
    For any valid user query, the system should return a response within 5 seconds.
    """
    start_time = time.time()
    response = client.post("/api/v1/query", json={
        "query": query,
        "session_id": generate_session_id()
    })
    elapsed = time.time() - start_time
    
    assert response.status_code == 200
    assert elapsed < 5.0

# Property 2: Response completeness for scheme queries
@given(scheme_name=st.sampled_from(["PM Scholarship", "NSP", "INSPIRE"]))
@pytest.mark.property_test
def test_property_2_scheme_response_completeness(scheme_name):
    """
    Feature: bharat-setu, Property 2: Response completeness for scheme queries
    For any query about a specific scheme, response should contain all required fields.
    """
    query = f"Tell me about {scheme_name}"
    response = client.post("/api/v1/query", json={
        "query": query,
        "session_id": generate_session_id()
    })
    
    assert response.status_code == 200
    data = response.json()
    response_text = data["response"].lower()
    
    # Verify required information is present
    assert any(word in response_text for word in ["eligibility", "eligible", "criteria"])
    assert any(word in response_text for word in ["benefit", "amount", "scholarship"])
    assert any(word in response_text for word in ["apply", "application", "process", "steps"])

# Property 11: Eligibility-based filtering
@given(
    age=st.integers(min_value=15, max_value=30),
    education=st.sampled_from(["10th", "12th", "undergraduate", "postgraduate"]),
    category=st.sampled_from(["general", "obc", "sc", "st"])
)
@pytest.mark.property_test
def test_property_11_eligibility_filtering(age, education, category):
    """
    Feature: bharat-setu, Property 11: Eligibility-based filtering
    For any user profile, all returned opportunities should match eligibility criteria.
    """
    profile = {
        "age": age,
        "education_level": education,
        "category": category
    }
    
    response = client.post("/api/v1/opportunities/find", json={
        "user_profile": profile
    })
    
    assert response.status_code == 200
    opportunities = response.json()["opportunities"]
    
    # Verify each opportunity matches user's eligibility
    for opp in opportunities:
        assert check_eligibility(opp, profile)

# Property 14: Reading level compliance
@given(query=st.text(min_size=10, max_size=200))
@pytest.mark.property_test
def test_property_14_reading_level(query):
    """
    Feature: bharat-setu, Property 14: Reading level compliance
    For any AI response, text should meet 10th-grade reading level or below.
    """
    response = client.post("/api/v1/query", json={
        "query": query,
        "session_id": generate_session_id()
    })
    
    if response.status_code == 200:
        response_text = response.json()["response"]
        grade_level = calculate_flesch_kincaid_grade(response_text)
        assert grade_level <= 10.0

# Property 19: Input sanitization
@given(
    malicious_input=st.one_of(
        st.just("<script>alert('xss')</script>"),
        st.just("'; DROP TABLE users; --"),
        st.just("<img src=x onerror=alert(1)>"),
        st.text().map(lambda t: f"<script>{t}</script>")
    )
)
@pytest.mark.property_test
def test_property_19_input_sanitization(malicious_input):
    """
    Feature: bharat-setu, Property 19: Input sanitization
    For any input with malicious content, system should sanitize before processing.
    """
    response = client.post("/api/v1/query", json={
        "query": malicious_input,
        "session_id": generate_session_id()
    })
    
    # Should not return 500 error (should handle gracefully)
    assert response.status_code in [200, 400]
    
    # Response should not contain unsanitized script tags
    if response.status_code == 200:
        response_text = response.json()["response"]
        assert "<script>" not in response_text
        assert "DROP TABLE" not in response_text

# Property 23: Bandwidth reduction effectiveness
@given(query=st.text(min_size=20, max_size=200))
@pytest.mark.property_test
def test_property_23_bandwidth_reduction(query):
    """
    Feature: bharat-setu, Property 23: Bandwidth reduction effectiveness
    For any query in low-bandwidth mode, response should be 60% smaller.
    """
    # Normal mode
    normal_response = client.post("/api/v1/query", json={
        "query": query,
        "session_id": generate_session_id(),
        "low_bandwidth": False
    })
    
    # Low-bandwidth mode
    low_bw_response = client.post("/api/v1/query", json={
        "query": query,
        "session_id": generate_session_id(),
        "low_bandwidth": True
    })
    
    if normal_response.status_code == 200 and low_bw_response.status_code == 200:
        normal_size = len(json.dumps(normal_response.json()))
        low_bw_size = len(json.dumps(low_bw_response.json()))
        
        reduction_percentage = (normal_size - low_bw_size) / normal_size
        assert reduction_percentage >= 0.60
```

### Frontend Testing

**Unit Tests** (Jest):
- Component rendering tests
- User interaction tests
- State management tests
- API call mocking tests

**Integration Tests** (Playwright/Cypress):
- End-to-end user flows
- Voice input workflow
- Text input workflow
- Opportunity finder workflow
- Low-bandwidth mode switching

**Example Frontend Test**:
```javascript
describe('Voice Input Component', () => {
  test('should enable text input when voice fails', async () => {
    // Mock voice input failure
    mockSTTService.transcribe.mockRejectedValue(new Error('STT failed'));
    
    // Click microphone button
    await userEvent.click(screen.getByRole('button', { name: /microphone/i }));
    
    // Verify text input becomes available
    expect(screen.getByRole('textbox')).toBeEnabled();
    expect(screen.getByText(/voice input failed/i)).toBeInTheDocument();
  });
});
```

### Performance Testing

**Load Testing**:
- Simulate 1,000 concurrent users
- Measure response times under load
- Identify bottlenecks
- Verify auto-scaling triggers

**Tools**: Apache JMeter, Locust

**Stress Testing**:
- Test system behavior at 2x expected load
- Verify graceful degradation
- Test recovery after overload

### Security Testing

**Automated Security Scans**:
- OWASP ZAP for vulnerability scanning
- Dependency vulnerability checks (Snyk, Dependabot)
- Static code analysis (Bandit for Python)

**Manual Security Testing**:
- Penetration testing
- Authentication bypass attempts
- Authorization testing
- Data exposure testing

### Continuous Integration

**CI Pipeline**:
```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run unit tests
        run: pytest tests/unit/ -v
      
      - name: Run property tests
        run: pytest tests/property/ -v --hypothesis-profile=ci
      
      - name: Run security tests
        run: bandit -r src/
      
      - name: Check code coverage
        run: pytest --cov=src --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v2
```

### Test Coverage Goals

- **Unit test coverage**: > 80% of code
- **Property test coverage**: 100% of correctness properties
- **Integration test coverage**: All critical user flows
- **Security test coverage**: All OWASP Top 10 vulnerabilities

### Testing Documentation

Each test should include:
- Clear description of what is being tested
- Reference to requirements or design properties
- Expected behavior
- Test data and assumptions
- Known limitations


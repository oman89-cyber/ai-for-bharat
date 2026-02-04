# OG AI Chatbot - Design Document

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interfaces                          │
├──────────────────┬──────────────────┬──────────────────────┤
│   Web UI         │  Voice Call      │  Direct API          │
│  (Browser)       │  (Twilio)        │  (HTTP Client)       │
└────────┬─────────┴────────┬─────────┴──────────┬───────────┘
         │                  │                    │
         └──────────────────┼────────────────────┘
                            │
                    ┌───────▼────────┐
                    │  FastAPI       │
                    │  Backend       │
                    │  (Port 8000)   │
                    └───────┬────────┘
                            │
         ┌──────────────────┼──────────────────┐
         │                  │                  │
    ┌────▼────┐        ┌────▼────┐       ┌────▼────┐
    │  Local  │        │Hugging  │       │ OpenAI  │
    │ Ollama  │        │  Face   │       │  API    │
    │(11434)  │        │  API    │       │         │
    └─────────┘        └─────────┘       └─────────┘
```

## Component Design

### 1. FastAPI Backend (`main.py`)

**Responsibilities:**
- Central AI orchestration
- Provider fallback management
- Request validation
- CORS handling

**Key Functions:**
- `ask_local()` - Query local Ollama instance
- `ask_hf()` - Query Hugging Face API
- `ask_openai()` - Query OpenAI API
- `ask_ai()` - Main endpoint with fallback logic

**Data Flow:**
```
User Question
    ↓
POST /ask
    ↓
Question Model Validation
    ↓
Try Local Ollama
    ├─ Success → Return with source
    └─ Fail → Try Hugging Face
         ├─ Success → Return with source
         └─ Fail → Try OpenAI
              ├─ Success → Return with source
              └─ Fail → Return error
```

### 2. Flask Voice Interface (`SERVER.PY`)

**Responsibilities:**
- Twilio webhook handling
- TwiML response generation
- Voice conversation loop

**Request/Response Cycle:**
```
Twilio Call
    ↓
GET/POST /voice
    ↓
Check for SpeechResult
    ├─ No speech (first call)
    │  └─ Return: Welcome prompt + Gather
    │
    └─ Has speech
       ├─ Send to FastAPI /ask
       ├─ Get AI response
       └─ Return: AI answer + Gather for next input
```

**TwiML Structure:**
- `<Say>` - Text-to-speech output
- `<Gather>` - Speech input collection
- `timeout="6"` - Wait 6 seconds for speech

### 3. Web UI Frontend

**Architecture:**
```
index.html (Structure)
    ↓
style.css (Presentation)
    ↓
script.js (Behavior)
```

**Key Components:**

| Component | Purpose |
|-----------|---------|
| Chat Box | Display message history |
| Input Field | User text entry |
| Send Button | Submit text message |
| Voice Button | Trigger speech recognition |

**State Management:**
- Chat history in DOM
- LocalStorage for persistence
- No backend state (stateless)

**User Interactions:**
```
Text Input:
  User types → Press Enter/Send
    ↓
  Fetch POST /ask
    ↓
  Display typing indicator
    ↓
  Receive response
    ↓
  Display message + Speak

Voice Input:
  Click 🎤 button
    ↓
  Browser speech recognition starts
    ↓
  User speaks
    ↓
  Auto-submit recognized text
    ↓
  Same as text flow
```

## Data Models

### Question (Pydantic)
```python
{
  "text": str  # User question
}
```

### AI Response
```python
{
  "answer": str,      # AI response text
  "source": str       # "local" | "huggingface" | "openai"
}
```

### Error Response
```python
{
  "error": str  # Error message
}
```

## Provider Strategy

### Local Ollama
- **Model:** tinyllama
- **Pros:** Fast, no API costs, privacy
- **Cons:** Limited capability, requires local setup
- **Response Limit:** 120 characters
- **Timeout:** 120 seconds

### Hugging Face
- **Model:** google/flan-t5-base
- **Pros:** Free tier, good quality
- **Cons:** Slower than local, rate limits
- **Max Tokens:** 200

### OpenAI
- **Model:** gpt-4-mini
- **Pros:** Best quality, most capable
- **Cons:** Requires API key, costs money
- **Fallback:** Last resort

## UI/UX Design

### Color Scheme
- **Background:** Dark (`#020617`)
- **Container:** Slate (`#0f172a`)
- **Accent:** Cyan (`#38bdf8`)
- **User Messages:** Cyan
- **Bot Messages:** Teal (`#a7f3d0`)

### Layout
```
┌─────────────────────────────┐
│    OG Chatbot (Header)      │
├─────────────────────────────┤
│                             │
│  Chat Box (360px height)    │
│  - User messages (right)    │
│  - Bot messages (left)      │
│  - Fade-in animation        │
│                             │
├─────────────────────────────┤
│ [Input Field] [Send] [🎤]   │
└─────────────────────────────┘
```

### Responsive Behavior
- Fixed width: 420px
- Centered on screen
- Scrollable chat history
- Touch-friendly buttons

## Error Handling

### Backend
```
Try-Except Pattern:
  ├─ Network errors → Return empty string
  ├─ API errors → Return empty string
  ├─ Timeout → Return empty string
  └─ All fail → Return error object
```

### Frontend
```
Fetch Error Handling:
  ├─ Network error → Show error message
  ├─ Invalid response → Show error message
  └─ Success → Display response
```

## Security Considerations

### CORS
- Allow all origins (`*`)
- Suitable for public API
- Consider restricting in production

### Environment Variables
- API keys stored in `.env`
- Never commit `.env` to version control
- Load via `python-dotenv`

### Input Validation
- Pydantic validates request body
- Text field required
- No SQL injection risk (no database)

### Voice Interface
- Twilio handles authentication
- TwiML prevents code injection
- No direct user input execution

## Performance Optimization

### Caching Opportunities
- Cache common questions
- Store provider responses
- LocalStorage for chat history

### Timeout Strategy
- Ollama: 120 seconds
- Hugging Face: Default (varies)
- OpenAI: Default (varies)
- Fallback prevents hanging

### Response Limiting
- Local: 120 char hard limit
- Prevents excessive output
- Faster response times

## Scalability Considerations

### Current Limitations
- Single FastAPI instance
- No database
- No user authentication
- No rate limiting

### Future Improvements
- Load balancing for multiple FastAPI instances
- Database for chat history
- User authentication & authorization
- Rate limiting per user/IP
- Caching layer (Redis)
- Message queue for async processing

## Deployment Architecture

### Development
```
Local Machine:
  ├─ FastAPI (localhost:8000)
  ├─ Flask (localhost:5001)
  ├─ Ollama (localhost:11434)
  └─ Web UI (file://)
```

### Production
```
Cloud Server:
  ├─ FastAPI (gunicorn/uvicorn)
  ├─ Flask (gunicorn)
  ├─ Ollama (optional, or use cloud LLM)
  ├─ Nginx (reverse proxy)
  ├─ SSL/TLS (HTTPS)
  └─ Web UI (static hosting)
```

## Integration Points

### External APIs
1. **Ollama** - Local LLM inference
2. **Hugging Face** - Cloud inference
3. **OpenAI** - GPT-4 Mini
4. **Twilio** - Voice interface (optional)
5. **Web Speech API** - Browser voice

### Communication Protocols
- HTTP/REST for APIs
- TwiML for Twilio
- WebSocket (future enhancement)

## Testing Strategy

### Unit Tests
- Provider functions (ask_local, ask_hf, ask_openai)
- Data model validation
- Error handling

### Integration Tests
- Full /ask endpoint flow
- Fallback logic
- Voice endpoint

### E2E Tests
- Web UI chat flow
- Voice input/output
- Chat persistence

## Future Enhancements

1. **User Accounts** - Authentication & chat history
2. **Streaming Responses** - Real-time text generation
3. **Multi-language** - Support multiple languages
4. **Custom Models** - Allow user-selected models
5. **Analytics** - Track usage & performance
6. **Rate Limiting** - Prevent abuse
7. **Caching** - Improve response times
8. **WebSocket** - Real-time bidirectional communication

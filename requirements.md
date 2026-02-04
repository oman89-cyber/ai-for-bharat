# OG AI Chatbot - Requirements

## Project Overview
OG AI Chatbot is a multi-interface conversational AI application that supports text, voice, and web-based interactions. It integrates multiple AI providers with a fallback system and includes both REST API and voice-enabled interfaces.

## Core Features

### 1. AI Backend (FastAPI)
- **Multi-provider AI system** with intelligent fallback:
  - Local Ollama (tinyllama model) - primary
  - Hugging Face Inference API (google/flan-t5-base) - secondary
  - OpenAI GPT-4 Mini - tertiary
- **REST API endpoint** (`/ask`) for question answering
- **CORS middleware** for cross-origin requests
- **Response limiting** (120 character hard limit for local responses)

### 2. Voice Interface (Flask)
- **Twilio-compatible voice endpoint** (`/voice`)
- **Speech-to-text processing** via Twilio
- **Text-to-speech responses** via Twilio
- **Conversational loop** with timeout handling
- **Two implementations**: `call_server.py` and `SERVER.PY` (enhanced version)

### 3. Web UI
- **Chat interface** with message history
- **Text input** with Enter key support
- **Voice input** using Web Speech API (browser-based)
- **Text-to-speech output** using Web Speech API
- **Local storage** for chat persistence
- **Real-time typing indicator**
- **Responsive design** with dark theme

## Technical Stack

### Backend
- **FastAPI** - REST API framework
- **Flask** - Voice interface framework
- **Pydantic** - Data validation
- **python-dotenv** - Environment configuration
- **requests** - HTTP client
- **openai** - OpenAI API client
- **huggingface-hub** - Hugging Face Inference API
- **Ollama** - Local LLM inference (external service)

### Frontend
- **HTML5** - Markup
- **CSS3** - Styling
- **Vanilla JavaScript** - Interactivity
- **Web Speech API** - Voice input/output
- **LocalStorage API** - Chat persistence

## Environment Variables
```
OPENAI_KEY=<your_openai_api_key>
HF_TOKEN=<your_huggingface_token>
```

## System Requirements

### External Services
- **Ollama** running on `localhost:11434` (optional, for local inference)
- **OpenAI API** access (requires valid API key)
- **Hugging Face API** access (requires valid token)
- **Twilio** account (for voice interface)

### Python Dependencies
- Python 3.8+
- FastAPI
- Flask
- Pydantic
- python-dotenv
- requests
- openai
- huggingface-hub

### Browser Requirements
- Modern browser with Web Speech API support (Chrome, Edge, Safari)
- JavaScript enabled
- LocalStorage support

## API Endpoints

### FastAPI (Port 8000)
- `GET /` - Health check
- `POST /ask` - Submit question and get AI response
  - Request: `{"text": "question"}`
  - Response: `{"answer": "response", "source": "provider_name"}`

### Flask Voice (Port 5001)
- `GET/POST /voice` - Twilio voice interface
  - Handles speech input and returns TwiML response

## Project Structure
```
.
├── main.py              # FastAPI backend
├── call_server.py       # Flask voice interface (basic)
├── SERVER.PY            # Flask voice interface (enhanced)
├── .env                 # Environment variables
├── web-ui/
│   ├── index.html       # Chat UI
│   ├── script.js        # Frontend logic
│   └── style.css        # Styling
└── venv/                # Python virtual environment
```

## Setup Instructions

1. **Install dependencies**
   ```bash
   pip install fastapi uvicorn flask pydantic python-dotenv requests openai huggingface-hub
   ```

2. **Configure environment**
   - Copy `.env` and add your API keys

3. **Start services**
   - FastAPI: `uvicorn main:app --reload --port 8000`
   - Flask: `python SERVER.PY` (runs on port 5001)
   - Open web UI: `web-ui/index.html` in browser

4. **Optional: Start Ollama**
   - `ollama run tinyllama`

## Fallback Logic
The system attempts AI providers in this order:
1. Local Ollama (fastest, no API costs)
2. Hugging Face (free tier available)
3. OpenAI (most capable, requires API key)

If all fail, returns error response.

## Notes
- Voice interface requires Twilio integration for production use
- Web UI voice features require HTTPS in production
- Chat history stored locally in browser (not synced to server)
- Response timeout: 120 seconds for local Ollama

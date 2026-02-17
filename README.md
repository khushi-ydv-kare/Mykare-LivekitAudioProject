# Malayalam Voice Agent Backend

A production-ready FastAPI backend that integrates with LiveKit to create a real-time voice AI agent supporting English and Malayalam languages.

## Features

- **LiveKit Integration**: Connect to LiveKit rooms for real-time audio streaming
- **Real-time Speech-to-Text**: Support for English and Malayalam with auto language detection
- **AI Conversation**: Context-aware responses using OpenAI GPT
- **Natural Malayalam TTS**: Female voice synthesis using Azure Speech Services
- **Agent Playground**: WebSocket interface for testing and development
- **Production Ready**: Comprehensive error handling, logging, and configuration management

## Architecture

```
LiveKit Room → FastAPI Backend → AI Processing → FastAPI Backend → LiveKit Room
     ↓                    ↓                    ↓                    ↓
  Audio Stream → STT → AI Response → TTS → Audio Output
```

## Project Structure

```
 api/                    # FastAPI routes and endpoints
    routes.py          # Main API endpoints
 audio/                 # Audio processing utilities
    processor.py       # Audio format conversion and processing
 agent/                 # AI conversation logic
    conversation.py    # OpenAI integration and context management
 livekit/              # LiveKit integration
    client.py         # Room connection and audio streaming
 stt/                  # Speech-to-Text
    transcriber.py    # Azure Speech Services integration
 tts/                  # Text-to-Speech
    synthesizer.py    # Azure Speech Services TTS
 utils/                # Utilities
    logger.py         # Structured logging configuration
 config.py             # Application configuration
 main.py               # FastAPI application entry point
 requirements.txt      # Python dependencies
 .env.example         # Environment variables template
```

## Setup

### Prerequisites

- Python 3.8+
- LiveKit server (cloud or self-hosted)
- Azure Speech Services account
- OpenAI API key

### Installation

1. **Clone and install dependencies**:
```bash
git clone <repository-url>
cd malayalam-voice-agent
pip install -r requirements.txt
```

2. **Configure environment variables**:
```bash
cp .env.example .env
```

Edit `.env` with your configuration:

```env
# LiveKit Configuration
LIVEKIT_URL=wss://your-livekit-server.com
LIVEKIT_API_KEY=devkey
LIVEKIT_API_SECRET=secret

# OpenAI Configuration
OPENAI_API_KEY=your-openai-api-key

# Azure Speech Configuration
AZURE_SPEECH_KEY=your-azure-speech-key
AZURE_SPEECH_REGION=eastus

# Server Configuration
HOST=0.0.0.0
PORT=8000
LOG_LEVEL=INFO
```

### Running the Server

```bash
python main.py
```

The server will start on `http://localhost:8000`

## API Endpoints

### REST Endpoints

- `POST /api/v1/connect-room` - Connect to a LiveKit room
- `POST /api/v1/disconnect-room` - Disconnect from a room
- `GET /api/v1/sessions` - List active sessions
- `GET /api/v1/conversation-history` - Get conversation history
- `DELETE /api/v1/conversation-history` - Clear conversation history
- `GET /health` - Health check

### WebSocket Endpoints

- `WS /api/v1/agent-playground` - Real-time audio streaming interface

## Usage Examples

### 1. Connect to a LiveKit Room

```bash
curl -X POST "http://localhost:8000/api/v1/connect-room" \
  -H "Content-Type: application/json" \
  -d '{
    "room_name": "test-room",
    "participant_name": "AI Agent"
  }'
```

### 2. Agent Playground WebSocket

```javascript
const ws = new WebSocket('ws://localhost:8000/api/v1/agent-playground');

// Send audio chunk
ws.send(JSON.stringify({
  type: 'audio_chunk',
  audio_data: base64AudioData,
  sample_rate: 48000
}));

// Send text message (for testing)
ws.send(JSON.stringify({
  type: 'text_message',
  text: 'Hello, how are you?',
  language: 'en'
}));

// Receive transcripts and AI responses
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type === 'transcript') {
    console.log('Transcript:', data.text);
  } else if (data.type === 'ai_response') {
    console.log('AI Response:', data.text);
    // Play audio_data if needed
  }
};
```

## Audio Flow

### Input Flow
1. LiveKit participant speaks → Audio frames sent to backend
2. Backend receives audio → Format conversion (16kHz, mono, PCM)
3. Audio chunks sent to STT → Real-time transcription
4. Language detection (English/Malayalam) → Text with language info

### Processing Flow
1. Transcribed text → AI conversation engine
2. Context-aware response generation → Response text
3. Text-to-Speech synthesis → Malayalam female voice audio
4. Audio streaming back to LiveKit room

## Configuration

### Audio Settings
- Sample Rate: 16kHz
- Channels: Mono (1)
- Chunk Size: 1024 samples
- Format: PCM/WAV

### Voice Settings
- Malayalam Voice: `ml-IN-SobhNeural` (female)
- English Voice: `en-US-JennyNeural` (female)
- Speech Rate: 0.9 (slightly slower for clarity)
- Audio Format: Raw16Khz16BitMonoPcm

### STT Settings
- Languages: English (en-US), Malayalam (ml-IN)
- Auto Language Detection: Enabled
- Output Format: Simple text with confidence scores

## Testing

### 1. Health Check
```bash
curl http://localhost:8000/health
```

### 2. Test with Agent Playground
Use the WebSocket endpoint to send audio chunks and receive responses.

### 3. Test with LiveKit Room
1. Set up a LiveKit room
2. Connect via the API
3. Join the room with a LiveKit client
4. Speak to test the full pipeline

## Monitoring and Logging

The application uses structured logging with JSON format. Logs include:
- Room connections and disconnections
- Audio processing events
- Transcription results
- AI response generation
- Error conditions

Log levels: DEBUG, INFO, WARNING, ERROR

## Error Handling

- Network reconnection handling for LiveKit
- Graceful degradation for STT/TTS failures
- Fallback responses for AI errors
- Comprehensive exception logging

## Production Deployment

### Environment Variables
Ensure all required environment variables are set in production:

- `LIVEKIT_URL`, `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`
- `OPENAI_API_KEY`
- `AZURE_SPEECH_KEY`, `AZURE_SPEECH_REGION`

### Security Considerations
- Use HTTPS in production
- Configure appropriate CORS origins
- Secure API keys and secrets
- Implement rate limiting if needed

### Scaling
- Stateless design allows horizontal scaling
- Consider load balancing for multiple instances
- Monitor resource usage (CPU, memory, network)

## Troubleshooting

### Common Issues

1. **LiveKit Connection Failed**
   - Check API key and secret
   - Verify LiveKit server URL
   - Ensure network connectivity

2. **STT Not Working**
   - Verify Azure Speech key and region
   - Check language settings
   - Ensure audio format is correct

3. **TTS Not Working**
   - Verify Azure Speech configuration
   - Check voice name availability
   - Ensure text is not empty

4. **AI Responses Not Generated**
   - Check OpenAI API key
   - Verify model availability
   - Check conversation history limits

### Debug Mode
Set `LOG_LEVEL=DEBUG` in `.env` for detailed logging.

## License

[Add your license information here]

## Support

For issues and questions, please create an issue in the repository or contact the development team.

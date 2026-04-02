# LiveKit Voice Agent PoC

A proof-of-concept voice agent built with [LiveKit Agents](https://docs.livekit.io/agents/), demonstrating a real-time conversational AI pipeline.

## Architecture

```
Browser (Playground)          LiveKit Cloud            Agent (this project)
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────────┐
│                  │     │                  │     │                      │
│  Microphone ─────┼────>│   WebRTC Room    │────>│  Deepgram Nova-3 STT │
│                  │     │                  │     │         │            │
│  Speakers <──────┼<────│                  │<────│  GPT-4.1-mini LLM    │
│                  │     │                  │     │         │            │
└──────────────────┘     └──────────────────┘     │  Cartesia Sonic TTS  │
                                                  └──────────────────────┘
```

**Pipeline flow:**

1. Your microphone audio streams to the agent via LiveKit Cloud (WebRTC)
2. **Deepgram Nova-3** transcribes speech to text
3. **OpenAI GPT-4.1-mini** generates a conversational response (placeholder for AIEnabler)
4. **Cartesia Sonic** synthesizes the response as speech
5. Audio streams back to your browser speakers

Additional components:
- **Silero VAD** -- Voice Activity Detection to know when you start/stop speaking
- **Multilingual Turn Detector** -- determines when it's the agent's turn to respond
- **BVC Noise Cancellation** -- filters background noise from your microphone input

## Prerequisites

- Python 3.10+
- API keys for all services (free tiers available for each)

## Setup

### 1. Clone and create a virtual environment

```bash
cd LiveKit-PoC
python3 -m venv .venv
source .venv/bin/activate
```

### 2. Install dependencies

```bash
pip install -e .
```

### 3. Configure API keys

Copy the example environment file and fill in your credentials:

```bash
cp .env.local.example .env.local
```

Edit `.env.local` with your keys:

| Variable | Source |
|---|---|
| `LIVEKIT_URL` | [LiveKit Cloud](https://cloud.livekit.io) -- create a free project, copy the WebSocket URL |
| `LIVEKIT_API_KEY` | LiveKit Cloud project settings |
| `LIVEKIT_API_SECRET` | LiveKit Cloud project settings |
| `DEEPGRAM_API_KEY` | [Deepgram Console](https://console.deepgram.com) -- free tier: 12,000 min/year |
| `OPENAI_API_KEY` | [OpenAI Platform](https://platform.openai.com/api-keys) |
| `CARTESIA_API_KEY` | [Cartesia Playground](https://play.cartesia.ai) -- free tier available |

### 4. Download model files

The Silero VAD model needs to be downloaded before first run:

```bash
python agent.py download-files
```

### 5. Run the agent

```bash
python agent.py dev
```

The `dev` command starts the agent with auto-reload on code changes.

For production use:

```bash
python agent.py start
```

## Talking to the Agent

1. Go to [LiveKit Cloud](https://cloud.livekit.io)
2. Open your project
3. Click **Playground** in the left sidebar
4. Grant microphone access when prompted
5. Start speaking -- the agent connects automatically

## Project Structure

```
LiveKit-PoC/
├── agent.py              # Voice agent entry point and pipeline configuration
├── pyproject.toml         # Python project metadata and dependencies
├── .env.local.example     # Template for API keys
├── .env.local             # Your API keys (git-ignored)
└── .gitignore
```

## Customization

### Change the voice

Replace the Cartesia voice ID in `agent.py` with any voice from the [Cartesia voice library](https://play.cartesia.ai):

```python
tts=inference.TTS(
    model="cartesia/sonic-3",
    voice="<your-voice-id>",
)
```

### Change the agent personality

Edit the `instructions` string in the `Assistant` class:

```python
class Assistant(Agent):
    def __init__(self) -> None:
        super().__init__(
            instructions="Your custom instructions here.",
        )
```

### Swap the LLM

Change the model identifier to use a different LLM:

```python
llm=inference.LLM(model="openai/gpt-4.1-mini")  # current
llm=inference.LLM(model="openai/gpt-4o")          # example alternative
```

## CLI Commands

| Command | Description |
|---|---|
| `python agent.py dev` | Development mode with auto-reload |
| `python agent.py start` | Production mode |
| `python agent.py connect` | Connect to a specific room |
| `python agent.py console` | Interactive local testing (no browser needed) |
| `python agent.py download-files` | Download required model files |

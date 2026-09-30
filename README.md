| | |
|---|---|
| **NAME** | Prabodh Kumar Bal |
| **ROLL NUMBER** | 2330099 |
| **BATCH** | GEN AI 2026 |
| **SUBMITTED TO** | Satyam Sir (Trainer) |
| **DATE** | 30 September 2026 |

---

# Autonomous Local AI Studio & RAG Platform

**An Edge-Native, Privacy-Preserving Platform with Local LLM (Gemma 2B), OpenAI Whisper Speech Recognition, and Retrieval-Augmented Generation (RAG)**

GitHub: https://github.com/prabodh7bal/Major_project_AgenticAi

---

## Overview

Most generative AI applications depend on centralized cloud APIs (OpenAI, Anthropic, Google Cloud). These bring recurring per-token costs, vendor lock-in, dependence on internet connectivity, and the transmission of private data to third-party servers.

This project is an edge-native, multi-modal AI platform that runs **entirely on local hardware**. No user query, document, or audio recording is sent to any external AI API. The platform integrates:

- **Local LLM chatbot:** Google Gemma 2B served through Ollama, with multi-turn conversational memory.
- **Speech-to-text:** OpenAI Whisper Tiny behind a Flask microservice, using FFmpeg for audio decoding.
- **Retrieval-Augmented Generation (RAG):** sliding-window chunking with TF-IDF / BM25 cosine-similarity retrieval, so answers are grounded in a local knowledge base and come with source citations.
- **Domain-specific agents:** an AI Resume ATS Analyzer and a Smart Vending Machine Restocking Agent.
- **Frontend and orchestration:** a React 19 + TypeScript (Vite) interface with a Node.js / Express proxy layer.

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     PRESENTATION LAYER                       │
│                React 19 + Vite  (Port 5174)                  │
│  Studio Dashboard · Gemma Chatbot · RAG Assistant            │
│  Audio Transcriber · Resume ATS Analyzer · Vending Agent     │
└───────────────────────────┬──────────────────────────────────┘
                            │ HTTP / REST  (/api)
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                    ORCHESTRATION LAYER                       │
│             Node.js / Express Server  (Port 5001)            │
│  Input validation · Conversation state · RAG retrieval       │
│  Context augmentation · Error handling                       │
└───────────────┬──────────────────────────────┬───────────────┘
                │ /api/chat, /api/rag          │ /transcribe
                ▼                              ▼
┌────────────────────────────┐   ┌─────────────────────────────┐
│       LOCAL LLM ENGINE     │   │  LOCAL SPEECH-TO-TEXT ENGINE│
│   Ollama  (Port 11434)     │   │  Flask + Whisper Tiny       │
│   Model: gemma:2b          │   │  (Port 5000) + FFmpeg       │
└────────────────────────────┘   └─────────────────────────────┘
```

### Services and Ports

| Service | Technology | Port | Responsibility |
|---|---|---|---|
| Frontend Web App | React 19, TypeScript, Vite | 5174 | Dashboard, chat UI, RAG UI, transcriber, ATS and vending screens |
| API Backend Proxy | Node.js, Express (ESM) | 5001 | Validation, conversation state, RAG indexing and scoring, request forwarding |
| Whisper Microservice | Python 3.12, Flask, Torch | 5000 | Audio decoding and speech transcription |
| Ollama LLM Engine | Ollama daemon | 11434 | Local inference for Gemma 2B |

**Why an Express middleware?** The browser never talks to Ollama or Whisper directly. The proxy validates inputs, blocks uncontrolled access to the model, performs the RAG search before the LLM sees the query, and avoids CORS problems through the Vite proxy.

---

## Modules

### 1. Gemma 2B Chatbot
- Multi-turn chat with history stored in browser `localStorage`.
- Backend guardrails: removes a stray leading assistant message and prepends a consistent system prompt.
- If Ollama is offline, the backend returns a clean `503` response instead of crashing.

### 2. Speech-to-Text (Whisper Tiny)
- Drag-and-drop upload (`.mp3`, `.wav`, `.m4a`, `.ogg`, `.webm`, `.flac`) and live microphone recording.
- Real-time microphone volume meter that warns when the input is silent.
- CLI option: `python transcribe.py "audio sample.mp3"`.
- **Ask Chatbot** button sends the transcript to the Gemma chat module.

### 3. RAG Knowledge Assistant
- **Chunking:** 400-character windows with a 60-character overlap.
- **Retrieval:** stop-word filtering, TF-IDF weighting, cosine similarity with a keyword-coverage bonus, giving a confidence percentage.
- **Augmentation:** the top 3 chunks are injected into the prompt, and the answer is returned with source citations.
- If the answer is not in the knowledge base, the assistant says so instead of guessing.

### 4. AI Resume ATS Analyzer
- Parses PDF (`pdfjs-dist`) and DOCX (`mammoth`) resumes.
- Scores Structure, Impact Metrics, Technical Competencies and Soft Skills, builds a keyword-match matrix, and suggests improvements.
- Runs in the browser; no resume is stored.

### 5. Smart Vending Machine Agent
- Accepts text or voice commands (for example, "I want a Protein Bar"), validates the `dispense_product` tool call, updates inventory, and raises low-stock alerts.
- Inventory and sales telemetry are **simulated** (no physical machine).

---

## Getting Started

### Prerequisites
- Node.js (LTS) and npm
- Python 3.12
- FFmpeg (added to system PATH)
- [Ollama](https://ollama.com) with the model pulled: `ollama pull gemma:2b`

### Run (four terminals)

```bash
# 1. LLM engine
ollama serve                      # port 11434

# 2. Whisper microservice
cd <whisper-folder>
pip install -r requirements.txt
python app.py                     # port 5000

# 3. Backend
cd backend
npm install
npm start                         # port 5001

# 4. Frontend
cd frontend
npm install
npm run dev                       # port 5174
```

Open http://localhost:5174 in the browser.

### Environment Variables
Copy `.env.example` to `.env` in the frontend and backend folders and fill in the values. Never commit real `.env` files.

| Variable | Used by | Purpose |
|---|---|---|
| `VITE_API_BASE_URL` | Frontend | Backend base URL (leave empty for local development) |
| `ALLOWED_ORIGINS` | Backend | Comma-separated list of allowed CORS origins |

---

## Deployment Note

The frontend can be hosted on Vercel. Ollama (Gemma 2B), the Whisper service and the Express backend are compute-heavy and run on a local machine, exposed to the hosted frontend through a tunnel (for example, Cloudflare Tunnel). The chat, RAG and transcriber features therefore work only while that machine and tunnel are running. The Resume ATS Analyzer and Vending screens run client-side.

---

## Known Limitations

- The LLM and speech engines are not cloud-hosted; the live demo depends on the local machine being online.
- Vending machine telemetry is simulated.
- The RAG engine uses TF-IDF / BM25 rather than dense embeddings, so matching is keyword-based rather than fully semantic.
- The knowledge base is a fixed, pre-indexed set of documents.

## Future Improvements

- Dense vector embeddings with a vector database (Chroma or FAISS).
- Runtime document upload into the knowledge base.
- Real IoT data for the vending agent.
- Multilingual chat and transcription (Hindi, Telugu).
- Cloud deployment of the LLM and ASR backend.

---

## Author

**Prabodh Kumar Bal** · Roll No. 2330099 · GEN AI 2026

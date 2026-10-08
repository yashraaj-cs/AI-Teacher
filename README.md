# 👩‍🏫 AI Teacher • Personal AI Tutor & Multimodal Agent

[![Streamlit](https://img.shields.io/badge/Streamlit-1.40+-FF4B4B.svg)](https://streamlit.io)
[![Python](https://img.shields.io/badge/Python-3.11%20%7C%203.12%20%7C%203.13-3776AB.svg)](https://www.python.org/)
[![Google GenAI](https://img.shields.io/badge/Google%20GenAI-Gemini%203.8%20Flash-4285F4.svg)](https://ai.google.dev/)
[![Zep Cloud](https://img.shields.io/badge/Memory-Zep%20Cloud%203.x-6366F1.svg)](https://www.getzep.com/)
[![ElevenLabs](https://img.shields.io/badge/Voice-ElevenLabs%20%2B%20Edge%20TTS-000000.svg)](https://elevenlabs.io/)

A production-grade, multimodal AI Tutor application engineered with **Python**, **Streamlit**, **Google Gemini API** (with automatic model failover), **OpenAI API**, **Zep Cloud** graph memory, **Local NumPy RAG**, and a **Dual-Tier Conversational Voice Pipeline**.

> 📖 **Looking for comprehensive technical specs, sequence diagrams, and interview prep?**  
> Check out the complete [DOCUMENTATION.md](file:///d:/Antigravity%20Projects/AI%20agent/DOCUMENTATION.md).

---

## 🌟 Key Features

1. **Session Onboarding & Recents Switching**:
   - Sleek dark-purple glassmorphic interface that initializes learner profiles.
   - Persistent `Recents` sidebar for switching between study threads without losing context.
2. **Pedagogical Persona (7-Step Breakdown)**:
   - Formulates explanations using: Definition $\rightarrow$ Intuition $\rightarrow$ Analogy $\rightarrow$ Technical Mechanics $\rightarrow$ Runnable Code $\rightarrow$ Industry Applications $\rightarrow$ Check-Your-Understanding Quiz.
3. **Persistent Knowledge Graph Memory (Zep Cloud)**:
   - Tracks learner background independently from individual chat threads.
   - Automatically recalls learner preferences, skill levels, and prior questions across conversations.
4. **Curriculum-Grounded RAG (Vector Search)**:
   - Ingests textbooks and course materials (`.pdf`, `.md`, `.txt`) using `PyPDF`.
   - Embeds text via `gemini-embedding-001` and ranks chunks via an in-memory NumPy cosine similarity store.
   - Displays expandable source citations with relevance scores and page numbers.
5. **Dual-Tier Conversational Voice Pipeline**:
   - **Speech-to-Text (STT)**: ElevenLabs Scribe v2 realtime WebSocket audio streaming with voice activity detection (VAD).
   - **Text-to-Speech (TTS)**: ElevenLabs Flash v2.5 streaming TTS pipe.
   - **Zero-Cost Fallback**: Asynchronous Microsoft Edge Neural TTS (`en-US-JennyNeural`) when ElevenLabs quota is exhausted.
6. **Multi-Model LLM Resilience**:
   - Starts on **Gemini 3.8 Flash**. If a rate limit or HTTP 429 error occurs, it dynamically falls back to **Gemini 3.1 Flash Lite** for 60 seconds.
7. **Prompt Security**:
   - Treats retrieved memory and RAG excerpts as untrusted user-level context turns to prevent prompt injection.

---

## 🛠️ Technology Stack Overview

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Frontend / Web UI** | Streamlit 1.40+, Custom CSS | Interactive chat, dark/light themes, dynamic SVG avatar |
| **Concurrency Layer** | `concurrent.futures` | Parallelizes Zep ingestion, Zep retrieval, and RAG search |
| **Reasoning Models** | Google Gemini (3.8 Flash / 3.1 Flash Lite) & OpenAI (GPT-4o) | High-speed pedagogical reasoning and token streaming |
| **Long-Term Memory** | Zep Cloud (`zep-cloud` 3.x) | Cross-thread user profiles & temporal knowledge graph |
| **RAG Knowledge Base** | PyPDF, NumPy, Gemini Embeddings | PDF extraction, recursive chunking, cosine vector similarity |
| **Voice Pipeline** | ElevenLabs STT/TTS + Microsoft Edge Neural TTS | Low-latency voice conversation with automatic fallback |

---

## 📐 Architecture & Execution Flow

```
Learner (Voice / Text)
  │
  ▼
Streamlit UI Controller
  │
  ├──► Concurrency Orchestrator (ThreadPoolExecutor max_workers=3)
  │      ├── [Thread 1] Asynchronously ingest turn into Zep Cloud
  │      ├── [Thread 2] Retrieve learner context graph (<USER_SUMMARY>, <FACTS>)
  │      └── [Thread 3] Embed query & search NumPy Vector Store (top-k)
  │
  ├──► Unified Context & Security Assembler (System Prompt + Untrusted Context)
  │
  ├──► Google Gemini Service (Streaming Generation with 3.8 -> 3.1 Lite failover)
  │      │
  │      ├── Token Stream ────────► Streamlit Live Chat UI
  │      │
  │      └── Sentence Stream ─────► Voice Pipe (ElevenLabs / Edge-TTS) ──► Audio Player
  │
  ▼
Post-turn Assistant Ingestion into Zep Cloud (Graph Memory building)
```

---

## 📂 Project Structure

```
d:/Antigravity Projects/AI agent/
│
├── app.py                      # Main Streamlit application entry point
├── DOCUMENTATION.md            # Comprehensive Technical Master Documentation
├── README.md                   # Repository overview & quickstart
├── requirements.txt            # Python dependencies
│
├── services/
│   ├── gemini_service.py       # Google GenAI service with automatic quota fallback
│   ├── openai_service.py       # OpenAI GPT-4o streaming service
│   ├── zep_service.py          # Zep Cloud SDK client (User, Thread, Memory Graph)
│   ├── memory_service.py       # High-speed concurrent orchestrator (ThreadPoolExecutor)
│   ├── rag_service.py          # High-level RAG indexer and query coordinator
│   └── voice/
│       ├── elevenlabs_stt.py   # Real-time WebSocket STT with Voice Activity Detection
│       ├── elevenlabs_tts.py   # Streaming TTS with VoiceStreamPipe
│       └── neural_tts.py       # Free, quota-free Microsoft Edge Neural TTS fallback
│
├── rag/
│   ├── ingestion.py            # PDF and text document loader
│   ├── chunking.py             # Recursive boundary-aware text chunker
│   ├── embeddings.py           # Gemini & OpenAI embedding adapters
│   ├── vector_store.py         # Local NumPy float32 cosine similarity engine
│   └── retriever.py            # Grounded chunk retriever
│
├── prompts/
│   ├── teacher_prompt.py       # Core pedagogical rules and 7-step explanation breakdown
│   └── rag_teacher_prompt.py   # RAG context injection and anti-hallucination constraints
│
├── components/
│   ├── avatar.py               # Dynamic SVG avatar (Idle, Thinking, Teaching)
│   ├── session_screen.py       # Learner onboarding and profile creation
│   └── chat_interface.py       # Chat feed, audio players, and source badges
│
├── knowledge_base/             # Drop educational PDFs/documents here
├── data/vector_store/          # Persisted index: embeddings.npy and chunks_metadata.json
└── scripts/                    # Ingestion and testing verification scripts
```

---

## 🚀 Quickstart Guide

### 1. Setup Virtual Environment
```bash
# Clone or navigate to project
cd "d:\Antigravity Projects\AI agent"

# Create and activate venv
python -m venv venv
.\venv\Scripts\Activate.ps1   # On Windows
# source venv/bin/activate    # On Linux/macOS
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Configure `.env` Secrets
Create a `.env` file in the root directory:
```env
LLM_PROVIDER=gemini
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-3.8-flash
GEMINI_FALLBACK_MODEL=gemini-3.1-flash-lite

ZEP_API_KEY=your_zep_api_key_here

EMBEDDING_PROVIDER=gemini
GEMINI_EMBEDDING_MODEL=gemini-embedding-001

# Optional Voice (ElevenLabs):
ELEVENLABS_API_KEY=your_elevenlabs_api_key_here
ELEVENLABS_VOICE_ID=JBFqnCBsd6RMkjVDRZzb
```

### 4. Index Knowledge Base (Optional)
Drop your course PDFs into `knowledge_base/` and run:
```bash
python scripts/index_knowledge_base.py
```

### 5. Launch the Application
```bash
streamlit run app.py
```
Open your browser at `http://localhost:8501`.

---

## 🧪 Testing & Verification Scripts

The `scripts/` directory provides diagnostic test scripts:

- `python scripts/verify_stage3_integration.py`: End-to-end test of Zep, RAG, Gemini streaming, and Voice fallback.
- `python scripts/test_retrieval.py`: Tests semantic search against the indexed vector store.
- `python scripts/test_voice_full_pipeline.py`: Tests the microphone STT and TTS speech output.

---

## 📄 License & Attribution
Designed for pair-learning and education. Built with modern Agentic AI design patterns. For complete architectural explanations, design rationale, and technical interview questions, please consult [DOCUMENTATION.md](file:///d:/Antigravity%20Projects/AI%20agent/DOCUMENTATION.md).

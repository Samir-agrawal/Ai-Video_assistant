# 🎬 AI Video Assistant — Meeting Intelligence RAG App

An end-to-end pipeline that ingests a YouTube URL or local audio/video file, transcribes it, and turns it into a searchable, chattable meeting record — complete with a title, summary, action items, key decisions, open questions, and a RAG-based chat interface to ask questions about the content.

## Features

- **Flexible input** — accepts a YouTube URL or a local audio/video file path.
- **Dual transcription engines**:
  - **Whisper** (local, OpenAI) for English audio.
  - **Sarvam AI** (speech-to-text-translate API) for Hinglish audio, translating to English while transcribing.
- **Automatic content generation** via **Mistral LLM** (through LangChain):
  - Meeting title
  - Bullet-point summary (using a map-reduce strategy for long transcripts)
  - Action items (with owner & deadline extraction)
  - Key decisions
  - Open/unresolved questions
- **RAG-based chat** — ask natural-language questions about the transcript, answered using a **ChromaDB** vector store + **HuggingFace sentence-transformer embeddings**, grounded strictly in the transcript content.
- **Streamlit UI** with real-time pipeline status tracking, plus a CLI entry point (`main.py`) for terminal use.

## Tech Stack

Python · LangChain (LCEL) · Streamlit · ChromaDB · HuggingFace sentence-transformers · OpenAI Whisper · Mistral AI · Sarvam AI · yt-dlp · pydub

## Architecture

```
Input (YouTube URL / local file)
        │
        ▼
 audio_processor.py   → download/convert to WAV, chunk into segments
        │
        ▼
 transcriber.py        → Whisper (English) or Sarvam AI (Hinglish)
        │
        ▼
 summarizer.py         → map-reduce summarization + title generation (Mistral)
        │
        ▼
 extractor.py          → action items / key decisions / open questions (Mistral)
        │
        ▼
 rag_engine.py          → build ChromaDB vector store → retriever → RAG chat (Mistral)
        │
        ▼
   Streamlit UI (app.py)  /  CLI (main.py)
```

## Setup

```bash
git clone <this-repo-url>
cd AI-Video_Assistant
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r Requirements.txt
```

You'll also need **FFmpeg** installed on your system (required by `pydub` / `yt-dlp`).

### Environment variables

Create a `.env` file in the project root:

```
MISTRAL_API_KEY=your_mistral_api_key
SARVAM_API_KEY=your_sarvam_api_key
WHISPER_MODEL=small
```

## Usage

**Streamlit UI:**
```bash
streamlit run app.py
```

**CLI:**
```bash
python main.py
```

## Known Limitations

- **YouTube ingestion (`yt-dlp`) may fail intermittently.** YouTube has been aggressively rate-limiting and blocking automated download tools at the IP level, which can cause `download_youtube_audio()` to return a "Forbidden" error regardless of the code itself. This is a known, external limitation of `yt-dlp`, not a bug in the pipeline.
  - **Local file input is fully functional** and recommended for reliable testing — pass a local `.mp4`/`.wav`/etc. file path instead of a YouTube URL.
- Whisper's `small` model (or larger) requires a reasonable amount of RAM/CPU; consider `tiny` or `base` for lighter environments.
- Not yet deployed — currently runs locally.
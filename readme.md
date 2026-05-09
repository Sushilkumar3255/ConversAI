# ConversAI 🤖

A multi-utility AI chatbot built with **LangGraph**, **LangChain**, and **Streamlit** that supports PDF document Q&A, web search, stock price lookup, and arithmetic — all within persistent, multi-threaded conversations.

---

## Features

- 📄 **PDF RAG (Retrieval-Augmented Generation)** — Upload a PDF and ask questions about its contents using FAISS-powered semantic search
- 🔍 **Web Search** — Answers grounded in real-time web results via DuckDuckGo
- 📈 **Stock Price Lookup** — Fetch live stock quotes by ticker symbol (powered by Alpha Vantage)
- 🧮 **Calculator** — Perform basic arithmetic (add, subtract, multiply, divide)
- 💬 **Multi-thread Conversations** — Each chat session is isolated with its own thread ID and persisted to a local SQLite database
- 🔁 **Conversation History** — Browse and reload past conversations from the sidebar

---

## Tech Stack

| Layer | Technology |
|---|---|
| LLM | OpenAI `gpt-4o-mini` |
| Embeddings | OpenAI `text-embedding-3-small` |
| Orchestration | LangGraph (StateGraph) |
| Vector Store | FAISS |
| Document Loading | LangChain PyPDFLoader |
| Persistence | SQLite via LangGraph `SqliteSaver` |
| Web Search | DuckDuckGo (`langchain-community`) |
| Stock Data | Alpha Vantage API |
| Frontend | Streamlit |

---

## Project Structure

```
.
├── ConversAI_backend.py      # LangGraph graph, tools, PDF ingestion, state logic
├── ConversAI_frontend.py     # Streamlit UI, session management, streaming
├── requirements.txt          # Python dependencies
├── chatbot.db                # SQLite checkpoint store (auto-created at runtime)
└── .env                      # Environment variables (not committed)
```

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-username/conversai.git
cd conversai
```

### 2. Set up a virtual environment

```bash
python -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_api_key_here
```

> **Note:** The Alpha Vantage API key is currently hardcoded in `ConversAI_backend.py`. For production use, move it to `.env` and load it via `os.getenv("ALPHAVANTAGE_API_KEY")`.

### 5. Run the app

```bash
streamlit run ConversAI_frontend.py
```

The app will open at `http://localhost:8501`.

---

## Usage

### Chat
Type any question in the chat input. The assistant will automatically decide which tools to use.

### PDF Q&A
1. Upload a PDF using the sidebar file uploader.
2. Once indexed, ask questions about the document in the chat.
3. Each chat thread maintains its own separate PDF index.

### Tools available to the assistant
- **`rag_tool`** — Queries the uploaded PDF for the current thread
- **`search_tool`** — Performs a DuckDuckGo web search
- **`get_stock_price`** — Fetches a stock quote by ticker (e.g. `AAPL`, `TSLA`)
- **`calculator`** — Evaluates arithmetic expressions

### Past Conversations
Click any thread ID in the sidebar under **"Past conversations"** to reload a previous session.

---

## Architecture Overview

```
User Input (Streamlit)
        │
        ▼
  HumanMessage added to state
        │
        ▼
   chat_node (GPT-4o-mini + tools)
        │
   ┌────┴────┐
   │         │
tools?      No tools → stream response to UI
   │
   ▼
tool_node (RAG / Search / Stock / Calculator)
   │
   └──────────► chat_node (loop until done)
```

State is persisted per `thread_id` using SQLite checkpointing, enabling full conversation resumability.

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `OPENAI_API_KEY` | ✅ Yes | OpenAI API key for LLM and embeddings |
| `ALPHAVANTAGE_API_KEY` | ⚠️ Optional | Move hardcoded key here for cleaner config |

---

## Known Limitations

- PDF indexes are stored **in-memory** and are lost on server restart. For persistence, consider replacing FAISS with a disk-backed vector store (e.g. Chroma, Qdrant).
- The Alpha Vantage free tier has rate limits (5 requests/min, 500 requests/day).
- DuckDuckGo search may occasionally be rate-limited under heavy use.

---

## License

MIT License. See [LICENSE](LICENSE) for details.
# Atlas AI Search Server

FastAPI backend for Atlas AI Search. This is an AI Engineering project that powers the chat stream, conversation memory, and web search flow used by the Next.js client.

## What it does

- Streams model output over Server-Sent Events (SSE)
- Uses LangGraph to manage chat state and tool routing
- Calls a local Ollama model through `langchain-ollama`
- Uses Tavily search when the model decides it needs web results
- Issues a conversation checkpoint ID so the client can continue the same thread

## Stack

- FastAPI
- LangGraph
- LangChain
- Ollama (`llama3.2` in the current code)
- Tavily Search

## Prerequisites

- Python 3.10+
- Ollama installed and running locally
- The `llama3.2` model pulled into Ollama
- A Tavily API key

## Environment

Create a `.env` file in this directory with:

```env
TAVILY_API_KEY=your_tavily_api_key
```

If you want to change the model, update the `model` value in [app.py](app.py).

## Install

From the `server` directory:

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

If you are on Windows, activate the virtual environment with:

```bash
venv\\Scripts\\activate
```

## Run

Start the API with Uvicorn:

```bash
uvicorn app:app --reload
```

By default, the service runs on `http://127.0.0.1:8000`.

## API

### `GET /chat_stream/{message}`

This endpoint returns an SSE stream.

Query parameter:

- `checkpoint_id` - optional conversation thread ID returned by the previous request

Example:

```bash
http://127.0.0.1:8000/chat_stream/What%20is%20Atlas%20AI%20Search%3F
```

To continue a prior conversation:

```bash
http://127.0.0.1:8000/chat_stream/Continue%20the%20thread?checkpoint_id=YOUR_CHECKPOINT_ID
```

## Stream Events

The frontend expects JSON payloads in SSE `data:` messages with these event types:

- `checkpoint` - emitted at the start of a new conversation with `checkpoint_id`
- `content` - streamed text from the model
- `search_start` - indicates the assistant decided to search, includes the search `query`
- `search_results` - includes the returned `urls` from Tavily
- `end` - marks the end of the stream

## Notes

- The backend uses permissive CORS so the client can call it from `localhost:3000`
- The current graph stores conversation state in memory through LangGraph’s memory checkpointer
- There is a small top-level demo call in [app.py](app.py) that invokes the model on startup; it is not part of the request flow

## Troubleshooting

- If the server cannot connect to Ollama, make sure the Ollama app/service is running locally
- If search does not work, verify that `TAVILY_API_KEY` is present in `.env`
- If the client cannot connect, confirm the backend is running on port `8000`

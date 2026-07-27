# Architecture Diagram

## System Overview

```mermaid
flowchart TD
    U[User] --> UI[Streamlit Web UI<br/>frontend.py]
    U --> CLI[Terminal CLI<br/>main.py]

    UI --> APP[LangGraph Travel App<br/>main.py]
    CLI --> APP

    APP --> STATE[(TravelState<br/>Message + Query + Results)]
    APP --> GRAPH[StateGraph Orchestrator]

    GRAPH --> FA[Flight Agent]
    GRAPH --> HA[Hotel Agent]
    GRAPH --> IA[Itinerary Agent]
    GRAPH --> FINA[Final Response Agent]

    FA --> FLT[Flight Tool<br/>search_flights]
    HA --> HOTEL[Tavily Tool<br/>tavily_search]
    IA --> LLM[Groq LLM<br/>Llama 3.3 70B]
    FINA --> LLM

    FLT --> AV[ AviationStack API ]
    HOTEL --> TV[Tavily Search API]
    LLM --> GROQ[Groq API]

    APP --> CKPT[Postgres Checkpointer<br/>LangGraph Memory]
    CKPT --> DB[(PostgreSQL Database)]

    ENV[.env Configuration<br/>API Keys + DATABASE_URL] --> APP
    ENV --> FLT
    ENV --> HOTEL

    FA --> RESULT[Flight Results]
    HA --> RESULT2[Hotel Results]
    IA --> ITIN[Itinerary Draft]
    FINA --> FINAL[Final Travel Response]

    RESULT --> IA
    RESULT2 --> IA
    ITIN --> FINA
    FINAL --> UI
    FINAL --> CLI
```

## Component Responsibilities

- User: submits a travel request through the web UI or terminal.
- Streamlit UI: provides the interactive chat/web experience.
- LangGraph App: orchestrates the multi-agent workflow.
- Agents:
  - Flight Agent: collects flight information.
  - Hotel Agent: gathers hotel recommendations.
  - Itinerary Agent: builds the travel plan.
  - Final Response Agent: composes the final answer.
- Tools:
  - Flight Tool: calls the AviationStack API.
  - Tavily Tool: performs web search for hotels and travel info.
- LLM Layer: uses Groq with Llama 3.3 to generate intelligent travel planning responses.
- Memory Layer: stores conversation/checkpoint state in PostgreSQL.

## Request Flow

1. The user enters a travel prompt.
2. The app starts the LangGraph workflow.
3. The Flight Agent and Hotel Agent gather external data.
4. The Itinerary Agent combines results and generates a trip plan.
5. The Final Agent creates the final polished response.
6. The result is shown to the user and persisted in PostgreSQL.

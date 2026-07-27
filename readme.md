# AI Travel Planning System using LangGraph

This project is a Real-World Multi-Agent AI System built using LangGraph.

The system uses 4 AI agents that work together to plan a complete trip automatically.

## Features

- ✈️ Flight Search Agent
- 🏨 Hotel Search Agent
- 🗓️ Itinerary Planning Agent
- 🤖 Final Response Agent
- 🧠 Memory using PostgreSQL
- 🌐 Real-time API Integration
- 💻 Streamlit Web Interface

---

# Tech Stack

- LangGraph
- LangChain
- Groq
- Llama 3.3 70B
- PostgreSQL
- Streamlit
- Tavily API
- AviationStack API

---

# Step 1: Create Python Environment

Open the terminal inside the project folder and run:

		python -m venv langgraph_env3


Now activate the environment:

#### Windows

		langgraph_env3\Scripts\activate


#### YouTube Tuturial (Hindi) - https://youtu.be/ctHby5vhDqg

#### YouTube Tuturial (English) -  https://youtu.be/_5XF5CCnbDk

---

# Step 2: Install Dependencies

Run the following command:

		pip install langgraph langchain langchain-openai langchain-groq langchain-community langchain-tavily psycopg[binary] psycopg_pool python-dotenv tavily-python requests streamlit

		pip install -U "psycopg[binary,pool]"  langgraph-checkpoint-postgres

---

# Step 3: Install PostgreSQL

Download and install PostgreSQL: https://www.postgresql.org/download/

⚠️ Important:
While installing PostgreSQL, remember:
- PostgreSQL Password
- Port Number

You will need them later while creating the database connection string.

---

# Step 4: Create Database

Open PostgreSQL and run:

CREATE DATABASE langgraph_memory_demo;


---

# Step 5: Setup `.env` File

Create a `.env` file inside the project folder.

Add the following keys:

GROQ_API_KEY=your_groq_api_key

TAVILY_API_KEY=your_tavily_api_key

AVIATIONSTACK_API_KEY=your_aviationstack_api_key

DATABASE_URL=postgresql://postgres:postgres@localhost:5433/langgraph_memory_demo


---

# Step 6: Get API Keys

## Get Groq API Key

https://console.groq.com

---

## Get Tavily API Key

https://tavily.com
  
---

## Get AviationStack API Key

https://aviationstack.com

---

# Step 7: Run the Application

#### Run Multi-Agent System in Terminal

		python main.py


This will test the multi-agent system through the terminal.

---

#### Run Streamlit Web App


		streamlit run frontend.py


This will launch the Multi-Agent AI web application.

---

#### Example Prompt

Plan a complete 7 days Japan trip including flights, hotels and sightseeing under 2 lakhs.


---

# Project Workflow

1. Flight Agent searches flights
2. Hotel Agent searches hotels
3. Itinerary Agent creates travel plan
4. Final Agent combines everything together
5. PostgreSQL stores conversation memory


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

---

## Technical Node and Edge Flow

This workflow is implemented using LangGraph in `main.py`. The graph is built as a StateGraph, where each node is a Python function and each edge describes the next execution step.

```mermaid
flowchart LR
    START([START]) --> N1[flight_agent]
    N1 --> N2[hotel_agent]
    N2 --> N3[itinerary_agent]
    N3 --> N4[final_agent]
    N4 --> END([END])

    N1 --> S[(TravelState)]
    N2 --> S
    N3 --> S
    N4 --> S
```

### Node Responsibilities

#### 1. flight_agent
- **Input:** `state["user_query"]`
- **Action:** calls the flight tool through `search_flights()`
- **Output updates:**
  - `flight_results` ← flight data from AviationStack API
  - `messages.append(AIMessage(...))`
  - `llm_calls += 1`

#### 2. hotel_agent
- **Input:** `state["user_query"]`
- **Action:** calls `tavily_search()` with hotel-focused query
- **Output updates:**
  - `hotel_results` ← hotel data from Tavily API
  - `messages.append(AIMessage(...))`
  - `llm_calls += 1`

#### 3. itinerary_agent
- **Input:** `state["user_query"]`, `state["flight_results"]`, `state["hotel_results"]`
- **Action:** builds prompt and calls Groq LLM (Llama 3.3 70B)
- **Output updates:**
  - `itinerary` ← generated travel itinerary
  - `messages.append(response)`
  - `llm_calls += 1`

#### 4. final_agent
- **Input:** `state["flight_results"]`, `state["hotel_results"]`, `state["itinerary"]`
- **Action:** synthesizes final response using Groq LLM
- **Output updates:**
  - `messages.append(response)`
  - `llm_calls += 1`

### Edge Connection Logic

The graph wiring in `main.py` follows this pattern:

```python
graph.add_edge(START, "flight_agent")
graph.add_edge("flight_agent", "hotel_agent")
graph.add_edge("hotel_agent", "itinerary_agent")
graph.add_edge("itinerary_agent", "final_agent")
graph.add_edge("final_agent", END)
```

This creates a strict pipeline where each node depends on the state produced by the previous node.

---

## Sequence Diagram: Agent Communication Over Time

```mermaid
sequenceDiagram
    participant User
    participant LangGraph as LangGraph<br/>StateGraph
    participant FA as Flight<br/>Agent
    participant FT as Flight<br/>Tool
    participant HA as Hotel<br/>Agent
    participant HT as Tavily<br/>Tool
    participant IA as Itinerary<br/>Agent
    participant FINALAGENT as Final<br/>Agent
    participant LLM as Groq<br/>LLM
    participant DB as PostgreSQL

    User->>LangGraph: invoke(user_query, config)
    LangGraph->>FA: execute node with TravelState
    FA->>FT: search_flights(query)
    FT->>FA: [flight data]
    FA->>LangGraph: return {flight_results, messages, llm_calls}
    LangGraph->>DB: checkpoint state
    
    LangGraph->>HA: execute node with updated TravelState
    HA->>HT: tavily_search(query)
    HT->>HA: [hotel data]
    HA->>LangGraph: return {hotel_results, messages, llm_calls}
    LangGraph->>DB: checkpoint state
    
    LangGraph->>IA: execute node with all prior results
    IA->>LLM: invoke(prompt with flights + hotels)
    LLM->>IA: [itinerary response]
    IA->>LangGraph: return {itinerary, messages, llm_calls}
    LangGraph->>DB: checkpoint state
    
    LangGraph->>FINALAGENT: execute node with all data
    FINALAGENT->>LLM: invoke(final synthesis prompt)
    LLM->>FINALAGENT: [final response]
    FINALAGENT->>LangGraph: return {messages, llm_calls}
    LangGraph->>DB: checkpoint state (final)
    
    LangGraph->>User: result (all messages + state)
```

### Sequence Explanation

1. **User Invocation:** User submits a query via CLI or Streamlit.
2. **Flight Agent Execution:** Fetches flight data and updates state.
3. **State Checkpoint:** PostgreSQL stores the intermediate state.
4. **Hotel Agent Execution:** Fetches hotel data using Tavily.
5. **State Checkpoint:** PostgreSQL persists the state.
6. **Itinerary Agent Execution:** Combines flights + hotels and generates itinerary via LLM.
7. **State Checkpoint:** State is persisted.
8. **Final Agent Execution:** Synthesizes all data into final response via LLM.
9. **Final Checkpoint:** State is stored for persistence and replay capability.
10. **Return to User:** Final response is sent back to the user interface.

---

## State Structure (TravelState)

```python
class TravelState(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]
    user_query: str
    flight_results: str
    hotel_results: str
    itinerary: str
    llm_calls: int
```

| Field | Type | Purpose |
|-------|------|---------|
| messages | list[AnyMessage] | Conversation history (accumulates with `operator.add`) |
| user_query | str | Original user request |
| flight_results | str | Flight data retrieved from AviationStack |
| hotel_results | str | Hotel data from Tavily search |
| itinerary | str | Generated travel itinerary |
| llm_calls | int | Counter for LLM API calls |

---

## How Agents Communicate

Agents do not call each other directly. They communicate through the shared **TravelState** object:

1. A node reads from the current state.
2. A node updates one or more state fields.
3. The graph passes the updated state to the next node.
4. The next node uses those values as input.

### Example Communication Flow

```
Flight Agent writes: state["flight_results"] = "Airline: Emirates, Departure: Mumbai..."
     ↓
Hotel Agent reads: state["user_query"], writes: state["hotel_results"] = "Hotel: Taj..."
     ↓
Itinerary Agent reads: state["flight_results"] + state["hotel_results"]
                 writes: state["itinerary"] = "Day 1: Arrive in Dubai..."
     ↓
Final Agent reads: all previous outputs, synthesizes final response
```

---

## Integration Points

- **Tools:**
  - `tools/flight_tool.py`: AviationStack API calls
  - `tools/tavily_tool.py`: Tavily API calls
- **LLM:** Groq API with Llama 3.3 70B model
- **Memory:** PostgreSQL with LangGraph checkpointer for persistence
- **Environment:** API keys and DATABASE_URL from `.env`

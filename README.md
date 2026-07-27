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


#### YouTube Tutorial (Hindi) - https://youtu.be/ctHby5vhDqg

#### YouTube Tutorial (English) - https://youtu.be/_5XF5CCnbDk

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

```sql
CREATE DATABASE langgraph_memory_demo;
```

---

# Step 5: Setup `.env` File

Create a `.env` file inside the project folder.

Add the following keys:

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
AVIATIONSTACK_API_KEY=your_aviationstack_api_key
DATABASE_URL=postgresql://postgres:postgres@localhost:5433/langgraph_memory_demo
```

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

```bash
python main.py
```

This will test the multi-agent system through the terminal.

---

#### Run Streamlit Web App

```bash
streamlit run frontend.py
```

This will launch the Multi-Agent AI web application.

---

#### Example Prompt

```text
Plan a complete 7 days Japan trip including flights, hotels and sightseeing under 2 lakhs.
```


---

# Project Workflow

1. Flight Agent searches flights
2. Hotel Agent searches hotels
3. Itinerary Agent creates travel plan
4. Final Agent combines everything together
5. PostgreSQL stores conversation memory

# Architecture Diagram

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

    FLT --> AV[AviationStack API]
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

## Node Responsibilities

### 1. flight_agent
- Input: state["user_query"]
- Action: calls the flight tool through search_flights
- Output updates:
  - flight_results
  - messages.append(AIMessage(...))
  - llm_calls + 1

### 2. hotel_agent
- Input: state["user_query"]
- Action: calls tavily_search with a hotel-focused query
- Output updates:
  - hotel_results
  - messages.append(AIMessage(...))
  - llm_calls + 1

### 3. itinerary_agent
- Input: state["user_query"], state["flight_results"], state["hotel_results"]
- Action: builds a travel itinerary prompt and calls the Groq LLM
- Output updates:
  - itinerary
  - messages.append(response)
  - llm_calls + 1

### 4. final_agent
- Input: state["flight_results"], state["hotel_results"], state["itinerary"]
- Action: synthesizes the final travel response using the LLM
- Output updates:
  - messages.append(response)
  - llm_calls + 1

## Edge Connection Logic

The graph wiring in [main.py](main.py) is:

- START → flight_agent
- flight_agent → hotel_agent
- hotel_agent → itinerary_agent
- itinerary_agent → final_agent
- final_agent → END
```


This means the workflow runs in a strict pipeline. Each node depends on the state produced by the previous node.

## How Agents Communicate

Agents do not call each other directly. Instead, they communicate through the shared state object called TravelState.

The communication pattern is:

1. A node reads from the current state.
2. A node updates one or more state fields.
3. The graph passes the updated state to the next node.
4. The next node uses those values as input.

### Example
- flight_agent writes flight_results
- hotel_agent reads user_query and writes hotel_results
- itinerary_agent reads both results and writes itinerary
- final_agent reads all of them and produces the final answer

## Technical State Structure

The shared state contains:

- messages: list of conversation messages
- user_query: original travel request
- flight_results: flight information from the flight tool
- hotel_results: hotel information from Tavily
- itinerary: generated itinerary text
- llm_calls: count of LLM executions

## External Tool Integration

- [tools/flight_tool.py](tools/flight_tool.py): calls the AviationStack API
- [tools/tavily_tool.py](tools/tavily_tool.py): calls the Tavily API
- Groq LLM: generates the itinerary and final response
- PostgreSQL checkpointer: stores the workflow state for persistence

## End-to-End Execution Flow

1. The user submits a travel prompt.
2. The graph starts at START.
3. flight_agent gathers flight data.
4. hotel_agent gathers hotel data.
5. itinerary_agent combines both sources and generates a plan.
6. final_agent produces the final polished response.
7. The workflow ends at END.



# Build a Multi-Agent Travel Planning System using LangGraph + MCP | Supervisor Agent + Guardrails + Human-in-the-Loop


This project extends the Multi-Agent Travel Planning System built in Part 1 and Part 2 by integrating Supervisor Agent + Guardrails + Human-in-the-Loop

## Part 1 of This Project

**GitHub Repository:**  
https://github.com/codewithaarohi/AI-Travel-Planning-System-using-LangGraph

**Video Tutorial:**  
Build a Real-World Multi-Agent AI System using LangGraph | Multi-Agent AI + Memory + APIs  
https://youtu.be/ctHby5vhDqg




## Part 2 of This Project

**GitHub Repository:**  
https://github.com/codewithaarohi/AI-Travel-Planning-App-using-LangGraph-and-MCP

**Video Tutorial:**  
Build a Real-World Multi-Agent AI System using LangGraph | Multi-Agent AI + Memory + MCP  
https://youtu.be/DjMX7o2EeV0

---

# Requirements

## APIs

- Groq API: https://console.groq.com
- Tavily API: https://www.tavily.com/
- AviationStack API: https://aviationstack.com/
- OpenWeatherMap API: https://openweathermap.org/

## Tools

- PostgreSQL: https://www.postgresql.org/download/
- Tavily MCP Server: https://docs.tavily.com/documentation/mcp

---

# Step 1: Create Python Environment


    python -m venv langgraph_env3


Activate:


    langgraph_env3\Scripts\activate


---

# Step 2: Install Dependencies


    pip install langgraph langchain langchain-openai langchain-groq langchain-community langchain-tavily psycopg[binary] psycopg_pool python-dotenv tavily-python requests streamlit



    pip install -U "psycopg[binary,pool]" langgraph-checkpoint-postgres


---

# Step 3: Install PostgreSQL

Download PostgreSQL:

https://www.postgresql.org/download/

**Important:** Note your PostgreSQL password and port number during installation.

---

# Step 4: Create Database


    CREATE DATABASE langgraph_memory_demo;


---

# Step 5: Setup .env File

Create a `.env` file:

    GROQ_API_KEY=your_groq_api_key

    TAVILY_API_KEY=your_tavily_api_key

    AVIATIONSTACK_API_KEY=your_aviationstack_api_key

    DATABASE_URL=postgresql://postgres:postgres@localhost:5433/langgraph_memory_demo


---

# Step 6: Get API Keys

- Groq: https://console.groq.com
- Tavily: https://tavily.com
- AviationStack: https://aviationstack.com
- OpenWeatherMap: https://openweathermap.org/

---

# Setup AviationStack MCP Server (Local MCP Server)

Repository:

https://github.com/Pradumnasaraf/aviationstack-mcp

Open PowerShell:

    E:

    cd E:\Multi_agent_system_with_MCP


Clone repository:


    git clone https://github.com/Pradumnasaraf/aviationstack-mcp.git

    cd aviationstack-mcp


## Install UV

Check:

    uv --version


Install:

    pip install uv


If installation fails:

    powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"


## Create .env File

    AVIATION_STACK_API_KEY=your_api_key_here


## Install Dependencies

    uv sync


This will:

- Create `.venv`
- Install dependencies
- Install AviationStack MCP package

## Activate Environment

    .venv\Scripts\activate


## Start MCP Server

    uv run -m aviationstack_mcp mcp run


or


    python -m aviationstack_mcp mcp run


The server will remain running and wait for MCP requests.

## Stop Server

    CTRL + C


---

# Setup Weather MCP Server

Get API key:

https://openweathermap.org/

Add the API key to your `.env` file.

Install dependencies:

    pip install mcp requests


---

# Run the Application

## Streamlit Web App
Run:

    streamlit run frontend.py


---

# Example Prompt

    Plan a complete 7 days Japan trip including flights, hotels and sightseeing under 2 lakhs.


---

# Features

- Multi-Agent Architecture using LangGraph
- PostgreSQL Memory
- Tavily Search Integration
- AviationStack MCP Integration
- Weather MCP Integration
- Streamlit Web App
- Real-Time Travel Planning
- Human in the loop
- Guardrails
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

## Request Flow

```mermaid
sequenceDiagram
    participant U as User
    participant M as main.py
    participant G as StateGraph
    participant F as flight_agent
    participant H as hotel_agent
    participant I as itinerary_agent
    participant R as final_agent

    U->>M: Enter travel request
    M->>G: app.invoke(user_query)
    G->>F: START → flight_agent
    F->>F: search_flights(query)
    F-->>G: flight_results

    G->>H: hotel_agent
    H->>H: tavily_search(query)
    H-->>G: hotel_results

    G->>I: itinerary_agent
    I->>I: LLM (ChatGroq) → generate itinerary
    I-->>G: itinerary

    G->>R: final_agent
    R->>R: LLM (ChatGroq) → final response
    R-->>G: final travel response

    G-->>M: compiled result
    M-->>U: Display FINAL RESPONSE
```

# AI Travel Orchestration System

A multi-agent AI travel planner built with **LangGraph, MCP, LangChain, Groq, PostgreSQL, and Streamlit**.

The system takes a natural-language travel request and coordinates specialized AI agents to generate a personalized travel plan covering flights, hotels, weather, budget, and itinerary.

---

## Architecture

```text
User Query
    ↓
Input Guardrail
    ↓
Supervisor Agent
    ↓
Specialist Agents
    ├── Flight Agent
    │       ↓
    │   AviationStack MCP
    │
    ├── Hotel Agent
    │       ↓
    │   Tavily MCP
    │
    ├── Weather Agent
    │       ↓
    │   Custom Weather MCP
    │
    ├── Budget Agent
    │
    └── Itinerary Agent
    ↓
Human-in-the-Loop Approval
    ↓
Final Response
    ↓
PostgreSQL Checkpointing
```

## Features

- Multi-agent architecture using LangGraph
- Supervisor-based agent routing
- Shared state between agents
- MCP-based external tool integration
  - AviationStack MCP integration
  - Tavily MCP integration
  - Custom Weather MCP server
- Flight information
- Hotel research
- Weather information
- Budget analysis
- AI-generated travel itinerary
- LLM input guardrails
- Human-in-the-loop approval
- PostgreSQL checkpointing
- Resumable LangGraph workflows
- Streamlit web interface

## Tech Stack

- Python
- LangGraph
- LangChain
- Groq
- MCP
- PostgreSQL
- Pydantic
- Streamlit
- AviationStack
- Tavily
- OpenWeather
- UV

## Project Structure

```text
AI-Travel-Orchestration-System/
│
├── aviationstack-mcp/
│   └── ...                     # Local AviationStack MCP server
│
├── agent.py                    # Travel agents and workflow logic
├── graph.py                    # LangGraph workflow and routing
├── state.py                    # Shared TravelState definition
├── mcp_client.py               # MCP client and tool integrations
├── weather_mcp_server.py       # Custom Weather MCP server
├── config.py                   # Configuration and LLM setup
├── frontend.py                 # Streamlit frontend
│
├── .env                        # API keys and database configuration
├── .gitignore
├── pyproject.toml
├── uv.lock
└── README.md
```

## Prerequisites

Make sure the following are installed:

- Python 3.13
- UV
- PostgreSQL

Check Python:

```bash
python --version
```

Check UV:

```bash
uv --version
```

If UV is not installed:

```bash
pip install uv
```

If installation fails on Windows:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

## Installation

### Step 1: Clone the Repository

```bash
git clone https://github.com/<your-username>/AI-Travel-Orchestration-System.git
cd AI-Travel-Orchestration-System
```

### Step 2: Create the Environment

Create the project environment using UV:

```bash
uv venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

Activate it on Linux/macOS:

```bash
source .venv/bin/activate
```

### Step 3: Install Dependencies

`pyproject.toml` is the single source of truth for dependencies, with `uv.lock` pinning exact versions — there's no separate `requirements.txt` to keep in sync.

```bash
uv sync
```

This creates/uses the project's virtual environment and installs the required dependencies.

### Step 4: Get API Keys

The application requires API keys for the following external services:

- Groq — https://console.groq.com
- Tavily — https://tavily.com
- AviationStack — https://aviationstack.com
- OpenWeatherMap — https://openweathermap.org/

### Step 5: Configure Environment Variables

Create a `.env` file in the project root:

```bash
GROQ_API_KEY=your_api_key_here
TAVILY_API_KEY=your_api_key_here
AVIATION_STACK_API_KEY=your_api_key_here
OPENWEATHER_API_KEY=your_api_key_here

DATABASE_URL=your_postgresql_connection_string
```

Do not commit the `.env` file to GitHub. Add the following to `.gitignore`:

```text
.env
.venv/
__pycache__/
```

## Setup AviationStack MCP Server

The project uses a local AviationStack MCP server.

Repository: https://github.com/Pradumnasaraf/aviationstack-mcp

From the project root:

```bash
git clone https://github.com/Pradumnasaraf/aviationstack-mcp.git
cd aviationstack-mcp
```

**Create AviationStack environment**

```bash
uv venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

Install the dependencies:

```bash
uv sync
```

**Configure API key**

Create a `.env` file inside the `aviationstack-mcp` directory:

```bash
AVIATION_STACK_API_KEY=your_api_key_here
```

**Start the MCP server**

```bash
uv run -m aviationstack_mcp mcp run
```

The server will remain running and wait for MCP requests. To stop it, press `CTRL + C`.

## Setup Weather MCP Server

The project includes a custom Weather MCP server that uses OpenWeather.

Add your OpenWeather API key to the main project's `.env` file:

```bash
OPENWEATHER_API_KEY=your_api_key_here
```

Install the required dependencies using UV:

```bash
uv add mcp requests
```

The Weather MCP server provides:

- Current weather
- Weather forecast

## PostgreSQL Setup

PostgreSQL is used for LangGraph checkpointing and persistent workflow state.

Add your PostgreSQL connection string to `.env`:

```bash
DATABASE_URL=your_postgresql_connection_string
```

The application initializes the required checkpoint tables when the graph is started.

## Run the Application

From the project root:

```bash
uv run streamlit run frontend.py
```

The Streamlit application will open in your browser.

### Example Prompt

```text
Plan a complete 7 days Japan trip including flights,
hotels and sightseeing under 2 lakhs.
```

Another example:

```text
Plan a 7-day trip to the Netherlands from Delhi
under a budget of 5 lakh. I prefer cultural attractions,
local food and a relaxed travel style.
```

## How It Works

1. **User Input** — The user provides a natural-language travel request containing information such as destination, origin, duration, budget, and preferences.
2. **Input Guardrail** — The system checks whether the request is related to travel planning before starting the main workflow.
3. **Supervisor Agent** — The Supervisor analyzes the request, extracts the trip constraints, and determines which specialist agents are required.
4. **Flight Agent** — Uses the AviationStack MCP integration to retrieve flight-related information.
5. **Hotel Agent** — Uses Tavily MCP to search for accommodation and relevant travel information.
6. **Weather Agent** — Uses the custom Weather MCP server to retrieve current weather and forecast information from OpenWeather.
7. **Budget Agent** — Analyzes the trip constraints and information collected from the other agents to provide a budget assessment.
8. **Itinerary Agent** — Combines the available information to generate a day-by-day travel itinerary.
9. **Human-in-the-Loop** — The workflow pauses and presents the generated itinerary for user approval. The user can approve the itinerary, provide feedback, or request changes.
10. **Final Response** — After the approval stage, the system generates the final travel response.
11. **PostgreSQL Checkpointing** — LangGraph state is persisted using PostgreSQL checkpointing, allowing workflow state to be maintained across interruptions and human-in-the-loop interactions.

## MCP Integrations

The project uses MCP to connect AI agents with external services.

```text
                    MCP Client
                   /    |     \
                  /     |      \
                 ↓      ↓       ↓
        AviationStack  Tavily  Weather MCP
              ↓          ↓          ↓
        Flight Data   Web Search  OpenWeather
```

**AviationStack MCP**

- Local MCP server
- Flight-related tools
- Runs using stdio transport

**Tavily MCP**

- Remote MCP integration
- Web search capabilities
- Used for travel and hotel research

**Custom Weather MCP**

- Local MCP server
- Developed for this project
- Current weather and forecast tools
- Uses OpenWeather API

## Human-in-the-Loop

The system includes a human approval step before finalizing the generated itinerary. This allows the user to review the proposed plan and provide feedback before the workflow continues.

## PostgreSQL Checkpointing

PostgreSQL is used with LangGraph's checkpointing mechanism to persist workflow state. This is particularly useful for human-in-the-loop workflows where execution can be interrupted and resumed using the stored state.

## Future Improvements

- Real-time flight search and filtering
- Structured Pydantic outputs across agents
- Automatic validation and retry mechanisms
- Improved human-in-the-loop revision routing
- Agent-level logging and observability
- Authentication and user-specific sessions
- Production deployment
- Scalable infrastructure

## Disclaimer

Travel information such as flight availability, hotel prices, and weather conditions can change. Always verify important travel information with the respective service provider before making bookings.

## Author

**Aditya Kumar Arya**

- GitHub: https://github.com/Shadow-Knight-803
- Portfolio: https://shadow-knight-803.github.io/portfolio/
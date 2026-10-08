# AI Travel Planner

An AI-powered travel planning application that uses multiple specialized agents to turn a travel request into a structured travel plan.

Instead of asking a single AI agent to handle the entire task, the application uses a supervisor-based multi-agent workflow. Different agents handle different parts of the trip, such as flights, hotels, weather, budget, and itinerary planning.

The project is built around LangGraph and MCP, with guardrails and human-in-the-loop approval added to make the workflow more controlled and practical.

---

## What It Does

A user can provide a request such as:

> Plan a 5-day trip from Delhi to Dubai for two people with a budget of ₹1,50,000.

The system processes the request and coordinates different agents to build the travel plan.

Depending on the request, the system can work with:

- Flight information
- Hotel information
- Weather information
- Budget planning
- Daily itinerary generation

The generated plan can also go through a human approval step before the final response is produced.

---

## Architecture

The application follows a supervisor-based multi-agent architecture.

```text
                         User
                           │
                           ▼
                    Guardrail Agent
                           │
                           ▼
                   Supervisor Agent
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
     Flight Agent      Hotel Agent     Weather Agent
          │                │                │
          └────────────────┼────────────────┘
                           │
                           ▼
                     Budget Agent
                           │
                           ▼
                    Itinerary Agent
                           │
                           ▼
                   Human Approval
                           │
                           ▼
                    Final Travel Plan

                    Main Components
Supervisor Agent
The supervisor acts as the central coordinator.
It interprets the user's request, identifies the travel requirements, and decides which specialized agents should be involved.
Flight Agent
Handles flight-related information and communicates with aviation-related tools through MCP.
Hotel Agent
Handles hotel-related information based on the destination and travel requirements.
Weather Agent
Retrieves weather information for the destination using the weather MCP server.
Budget Agent
Organizes the estimated travel expenses and considers the user's budget when preparing the plan.
Itinerary Agent
Combines the information collected by the other agents and produces the final day-by-day travel plan.
Guardrails
The application includes a guardrail layer that checks incoming requests before the main travel workflow is executed.
Human-in-the-Loop
The application can generate a draft travel plan and wait for user approval before continuing with the final response.
This allows the user to review the generated plan instead of relying completely on an autonomous workflow.
MCP
The project uses the Model Context Protocol (MCP) to connect agents with external tools.
The current implementation includes MCP integrations for services such as:
- Aviation data
- Weather information
- Web search
A custom weather MCP server is also included in the project.
AI Agents
    │
    ▼
MCP Client
    │
    ├── Aviation MCP
    ├── Weather MCP
    └── Search MCP

This keeps tool access separate from the core agent logic.
Tech Stack
Backend
- Python
- FastAPI
- LangGraph
- LangChain
- PostgreSQL
AI / Agents
- Groq
- LangGraph multi-agent workflow
- MCP
- Guardrails
- Human-in-the-loop
External Services
- Tavily
- AviationStack
- OpenWeather
Frontend
- HTML
- CSS
- JavaScript
Deployment
- Docker
- Render
Project Structure
travel_planner-AI/
│
├── static/
│   ├── style.css
│   └── script.js
│
├── templates/
│   └── index.html
│
├── app.py
├── backend.py
├── mcp_client.py
├── custom_weather_mcp_server.py
├── requirements.txt
├── Dockerfile
├── .dockerignore
├── .gitignore
├── LICENSE
└── README.md

How It Works
A typical request goes through the following process:
1. User enters a travel request
             ↓
2. FastAPI receives the request
             ↓
3. Guardrail checks the request
             ↓
4. Supervisor analyzes the request
             ↓
5. Required agents are selected
             ↓
6. Agents use MCP tools / external services
             ↓
7. Information is combined
             ↓
8. Travel plan is generated
             ↓
9. User reviews the draft
             ↓
10. User approves or requests changes
             ↓
11. Final travel plan is returned

Running Locally
1. Clone the repository
git clone https://github.com/SharmaRaj-0605/travel_planner-AI.git

cd travel_planner-AI

2. Create the Python environment
Using Conda:
conda create -n travel-agent python=3.11

Activate it:
conda activate travel-agent

3. Install dependencies
pip install -r requirements.txt

4. Configure environment variables
Create a .env file in the project root.
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
AVIATIONSTACK_API_KEY=your_aviationstack_api_key
OPENWEATHER_API_KEY=your_openweather_api_key
DATABASE_URL=your_postgresql_connection_string
DEFAULT_ORIGIN_IATA=your_default_origin

If LangSmith tracing is enabled, add the corresponding LangSmith variables as well.
Do not commit the .env file to GitHub.
5. Start the application
python app.py

The application will be available at:
(https://tavel-planner-ai.onrender.com)

Open the URL in your browser.

# ✈️ TripMate AI — Multi-Agent Travel Planner

An AI travel planner that turns a plain-English request into a complete, day-by-day trip plan. Instead of one big prompt, it uses a **supervisor-based multi-agent workflow** built with **LangGraph**, where specialized agents handle flights, hotels, weather, budget, and itinerary. Real-world data comes in through **MCP (Model Context Protocol)** tools, and you get to **review and approve the draft** before the final plan is produced.

🌐 **Live demo:** [tavel-planner-ai.onrender.com](https://tavel-planner-ai.onrender.com)

---

## 📖 Table of Contents

- [Features](#-features)
- [How It Works](#-how-it-works)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Environment Variables](#-environment-variables)
- [Run with Docker](#-run-with-docker)
- [API Reference](#-api-reference)
- [Example Prompts](#-example-prompts)
- [License](#-license)

---

## ✨ Features

- 🧠 **Supervisor agent**: reads the request, extracts constraints (destination, duration, budget, etc.), and picks only the agents needed
- 🛡️ **Input guardrail**: checks every request and blocks anything that isn't a travel-planning query
- ✈️ **Flight agent**: pulls airport and airline data through the AviationStack MCP server
- 🏨 **Hotel agent**: searches live hotel information through the Tavily MCP server
- 🌤️ **Weather agent**: current weather and forecast via a **custom OpenWeather MCP server** included in this repo
- 💰 **Budget agent**: breaks down estimated costs against your budget
- 🗓️ **Itinerary agent**: combines everything into a day-by-day plan
- 🙋 **Human-in-the-loop**: approve the draft or send feedback to revise it before the final answer
- 💾 **Persistent state**: conversations are checkpointed in PostgreSQL, so approvals can resume by `thread_id`
- 📄 **PDF export**: download the final plan as a PDF from the browser
- 🐳 **Docker-ready** and deployed on Render

---

## 🔄 How It Works

1. You describe your trip, e.g. *"Plan a 5-day trip from Delhi to Dubai for two people under ₹1,50,000."*
2. **FastAPI** receives the request and starts the LangGraph workflow.
3. The **guardrail** validates the request (non-travel requests are blocked).
4. The **supervisor** analyzes it and selects which agents to run.
5. Selected agents run in order and call their MCP tools (Tavily, AviationStack, Weather).
6. The **itinerary agent** compiles a draft plan.
7. The workflow **pauses** and asks you to review the draft.
8. You **approve**, or **reject with feedback** to get a revised plan.
9. The **final agent** returns the finished travel plan.

---

## 🏗️ Architecture

```mermaid
flowchart TD
    U[User] --> API[FastAPI]
    API --> S["Supervisor + Input Guardrail"]
    S -- blocked --> B[Guardrail Blocked Response]
    S -- allowed --> F[Flight Agent]
    F --> H[Hotel Agent]
    H --> W[Weather Agent]
    W --> BU[Budget Agent]
    BU --> I[Itinerary Agent]
    I --> HA{{"Human Approval (interrupt)"}}
    HA -- approve / feedback --> FA[Final Agent]
    FA --> R[Final Travel Plan]

    F -. MCP .-> AV[AviationStack MCP]
    H -. MCP .-> TV[Tavily MCP]
    W -. MCP .-> WM[Custom Weather MCP]
```

> The supervisor only routes through the agents your request needs. Any agent not selected is skipped, and the flow goes straight on to the next one.

**MCP integrations** (all managed in `mcp_client.py`):

| Server | Transport | Used for |
|--------|-----------|----------|
| Tavily | Streamable HTTP | Hotel / web search |
| AviationStack | stdio (via `uvx aviationstack-mcp`) | Airports and airlines data |
| Weather (custom) | stdio (`custom_weather_mcp_server.py`) | `get_current_weather`, `get_forecast` |

Each MCP server is loaded independently, so one failing server won't crash the others.

---

## 🧰 Tech Stack

| Layer | Technology |
|-------|------------|
| Backend | Python 3.11, FastAPI, Uvicorn |
| Agents | LangGraph, LangChain |
| LLM | Groq (`openai/gpt-oss-120b`) |
| Tools | Model Context Protocol (`mcp`, `langchain-mcp-adapters`) |
| Database | PostgreSQL (LangGraph `PostgresSaver` checkpointer) |
| External APIs | Tavily, AviationStack, OpenWeather |
| Frontend | HTML, CSS, JavaScript (Jinja2 templates, `marked`, `html2pdf.js`) |
| Deployment | Docker, Render |

---

## 📁 Project Structure

```
tavel_planner-AI/
├── static/
│   ├── script.js                     # Frontend logic (requests, approval flow, PDF export)
│   └── style.css                     # UI styling
├── templates/
│   └── index.html                    # Main UI
├── app.py                            # FastAPI app and routes
├── backend.py                        # LangGraph workflow, agents, guardrail, HITL, Postgres checkpointer
├── mcp_client.py                     # MCP client setup and tool wrappers
├── custom_weather_mcp_server.py      # Custom OpenWeather MCP server
├── requirements.txt
├── Dockerfile
├── LICENSE
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- **Python 3.11**
- A **PostgreSQL** database (e.g. a free Render Postgres instance)
- **[uv](https://docs.astral.sh/uv/)** installed (the AviationStack MCP server runs through `uvx`)
- API keys for **Groq**, **Tavily**, **AviationStack**, and **OpenWeather**

### Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/SharmaRaj-0605/tavel_planner-AI.git
   cd tavel_planner-AI
   ```

2. **Create a virtual environment**

   ```bash
   # Conda
   conda create -n travel-agent python=3.11
   conda activate travel-agent

   # or venv
   python -m venv venv
   source venv/bin/activate        # Windows: venv\Scripts\activate
   ```

3. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

4. **Add your environment variables**: create a `.env` file in the project root (see below).

5. **Start the app**

   ```bash
   python app.py
   ```

6. Open **http://127.0.0.1:8000** in your browser.

---

## 🔐 Environment Variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
AVIATIONSTACK_API_KEY=your_aviationstack_api_key
OPENWEATHER_API_KEY=your_openweather_api_key
DATABASE_URL=postgresql://user:password@host:5432/dbname
```

| Variable | Required | Description |
|----------|:--------:|-------------|
| `GROQ_API_KEY` | ✅ | Powers all agents (LLM) |
| `DATABASE_URL` | ✅ | PostgreSQL connection string; `sslmode=require` is added automatically if missing |
| `TAVILY_API_KEY` | ✅ | Hotel and web search via Tavily MCP |
| `AVIATIONSTACK_API_KEY` | ✅ | Flight data (`AVIATION_STACK_API_KEY` is also accepted) |
| `OPENWEATHER_API_KEY` | ✅ | Weather data for the custom MCP server |

> ⚠️ The app **fails at startup** if `GROQ_API_KEY` or `DATABASE_URL` is missing. Never commit your `.env` file.

---

## 🐳 Run with Docker

```bash
docker build -t tripmate-ai .

docker run -p 8000:8000 --env-file .env tripmate-ai
```

Then open **http://localhost:8000**.

---

## 🔌 API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Web UI |
| `POST` | `/api/travel` | Start a new travel request |
| `POST` | `/api/travel/approve` | Approve or reject (with feedback) the draft plan |
| `GET` | `/health` | Health check |

**Start a request**

```http
POST /api/travel
Content-Type: application/json

{
  "message": "Plan a 5 day Dubai trip from Delhi with flights, hotels and sightseeing.",
  "thread_id": null
}
```

**Approve or revise the draft**

```http
POST /api/travel/approve
Content-Type: application/json

{
  "thread_id": "<thread_id from the previous response>",
  "approved": false,
  "feedback": "Make it more budget friendly and add more food spots."
}
```

> `feedback` is required when `approved` is `false`.

---

## 💬 Example Prompts

- *Plan a complete 7 days Japan trip including flights, hotels and sightseeing under 2 lakhs.*
- *Plan a 5 days Dubai trip from Delhi with flights, hotels and sightseeing.*
- *Plan a 7 days Thailand trip with budget hotels and sightseeing.*

---

## 📄 License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

---

⭐ If you like this project, consider giving it a star!

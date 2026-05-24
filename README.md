# HokieHomes (VT Housing Advisor)

Welcome to **HokieHomes** – the most over-engineered way to get Virginia Tech students on-campus or off-campus housing recommendations (and maybe revolutionize finding a dorm room along the way).

## Project Purpose

HokieHomes is a full-stack AI-powered housing advisor. It lets students converse naturally (“cheapest with AC”, “private bath, under 4500”) and get personalized, ranked matches for university dorms or local apartments. 

Everything goes through a conversational AI layer tuned to minimize friction, maximize helpfulness, and (almost) never say “no results.” Extreme clarity, ranking, and reasoning-per-explanation are built into every response.

---

## Codebase Architecture


### Major Components

- **Python Flask API backend** (`server.py`)
  - Hosts `/api/chat`, a POST endpoint for client-to-AI chat.
  - Handles CORS for local development by default.
  - Authenticates with Databricks serving endpoints via Bearer tokens.
  - Handles message relay, schema wrangling, and error normalization.
- **Databricks LLM Gateway Logic**
  - `chat()` functions in Python (shared in both `server.py` and CLI) assemble, ship, and parse requests to a Databricks LLM endpoint.
  - Multiple return schema variants handled: OpenAI-style or flat output.
- **Databricks Prompt Orchestration** (`databricx_prompt.py`)
  - Houses a truly "insane" system prompt that governs the AI's behavior.
  - Prompt enforces role (VT housing advisor), expected fields, constraints, defaults, fallback logic, presentation format, and clarification style.
  - Contains internal logic for interpreting natural-language student requests into normalized, ranked queries and answers.
- **CLI Chatbot** (`chat_cli2.py`)
  - For the real hackers: talk to your housing bot from the terminal.
  - Relays your messages to the same `chat()` function for uniform LLM access.
- **Frontend (React + Vite, in `/VTHACKS`)**
  - (See notes in `/VTHACKS/README.md`)
  - Designed for speed and modern developer experience.

---

## APIs & Technologies Used

- **Databricks Foundation Model Serving API**
  - Custom AI chat endpoint taking OpenAI-like chat payloads.
  - Requires `DATABRICKS_HOST`, `DATABRICKS_TOKEN`, and `ENDPOINT` environment variables.
- **Python stack**
  - Flask (API server, routing)
  - requests (HTTP client to Databricks)
  - python-dotenv (local env var loading)
  - Flask-CORS (local dev support)
- **JavaScript/React (frontend)**
  - Vite for local/front-end build experience.

---

## Architecture Diagram (Text)

```
  +------------+         HTTP/JSON         +-----------------+         HTTPS/JSON         +-------------------+
  | Frontend   |  <──POST /api/chat──>     | Flask API (Py)  |  <──POST messages──>      | Databricks LLM    |
  | (React)    |-------------------------> | server.py       |-------------------------> | /serving-endpoint |
  +------------+                          +-----------------+                          +-------------------+
          ▲                                                        ▲
          |                                                        |
          |                                                        |
   [ Vite, hot reload ]                                      [ SYSTEM PROMPT ]
          |                                                        |
          ▼                                                        ▼
  +------------+                                      +--------------------------+
  | CLI Chat   |  <──(same API/chat.py logic)──>      | databricx_prompt.py      |
  +------------+                                      | "Advisor" role definition|
                                                     +--------------------------+
```

---

## Features and Philosophy

- **Natural Language Understanding:** The LLM parses all sorts of student lingo (“as cheap as possible with AC,” “I want a suite, co-ed, close to gym”).
- **Failback and Ladder Logic:** If a query is too strict (“female-only, suite, AC, under $3000, honors, top floor”), the AI *relaxes* criteria in order – never just saying "nope." It'll state what concessions it made.
- **Extreme Prompt Engineering:** Nearly 1000 lines of system prompt govern ranking, field mapping, relaxation, friendly tone, and refusal to ever output SQL/internal structure.
- **Multi-modal input:** Works from the web or terminal CLI.
- **Explainability:** Every recommendation includes “why it matched” and what, if any, filters were relaxed.
- **Privacy/Safety:** Never dumps raw SQL, schema, or internal queries.

---

## What Could Be Even More Insane (Improvements)

- **Persistent User Context:** Remember preferences, prior conversations, or user IDs for smoother follow-ups.
- **True Multi-Turn Dialogue:** Maintain chat history and context seamlessly rather than only last-turn logic.
- **Real Housing Listings Data:** Instead of static/dummy content, integrate with VT APIs or local housing aggregators for dynamic, up-to-date listings.
- **Testing/CI:** Add tests, lint, type-checks (current repo is adventure-driven development).
- **Frontend Polish:** The JS/React frontend is “minimal” – upgrade to MUI, Chakra, or custom design.
- **Containerization/Deployment:** Dockerize everything for reproducible deployment.
- **Monitoring & Error Reporting:** Add logging and reporting for both API+AI failures.

---

## Language Composition

- JavaScript: 47.2%
- CSS: 31.4%
- Python: 20.5%
- Other: 0.9%

**In other words:** It's a triple-stack JS/React + AI backend + Pro-level prompt engineering playground which acts as a proof of concept.

---

## Getting Started

1. Clone the repo and set up your `.env` with Databricks host, endpoint, and access token.
2. (Optional) Use `chat_cli2.py` to chat from your terminal.
3. Run the Flask API: `python server.py`
4. See `/VTHACKS` for React frontend.
5. [Push, extend, and break things — this is HokieHomes!](https://github.com/ifzaal4dev/HokieHomes)

---

*Created for VT Hacks 13, rewritten for the future of housing bots and hackathons alike.*

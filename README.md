# 🤖 Boundless AI Grading Service

An AI-powered REST API that automatically grades hackathon submissions using [Claude](https://www.anthropic.com/claude) (Anthropic). Built for the **Boundless** platform, it evaluates Stellar/Soroban blockchain projects by analyzing GitHub repositories, uploaded documents, and on-chain activity.

---

## What It Does

When a hackathon submission is sent to this service, it:

1. **Clones & analyzes the GitHub repo** — counts lines of code, languages used, and measures code complexity using [Radon](https://radon.readthedocs.io/).
2. **Extracts content from uploaded files** — reads PDF, DOCX, and Markdown documents linked in the submission.
3. **Verifies Stellar on-chain activity** — checks whether a Stellar wallet address exists and has real transactions, and confirms if a Soroban smart contract ID is deployed on-chain.
4. **Grades using Claude AI** — builds a structured prompt from all gathered evidence and asks Claude to score the submission across 5 criteria.
5. **Returns a structured JSON result** — scores, reasoning, strengths/weaknesses, flags, and a final recommendation.

### Grading Criteria

| Criterion | Weight | Description |
|---|---|---|
| Innovation | 25% | Uniqueness and creativity of the solution |
| Technical Execution | 25% | Code quality and completeness |
| Stellar Integration | 20% | How well it uses Stellar / Soroban smart contracts |
| UX / Design | 15% | User experience and visual polish |
| Completeness | 15% | Whether the project is a working demo |

### Recommendation Levels

- `STRONG_ACCEPT` — Top-tier submission
- `ACCEPT` — Good submission worth rewarding
- `BORDERLINE` — Has merit but significant gaps
- `REJECT` — Does not meet minimum bar

---

## Project Structure

```
ai-grading-service/
├── app/
│   ├── main.py              # FastAPI app entry point
│   ├── engine.py            # Core grading logic (orchestrates all steps)
│   ├── models.py            # Pydantic data models (input/output schemas)
│   ├── prompts.py           # Claude prompt builder
│   ├── routers/
│   │   └── grading.py       # POST /grading/hackathon endpoint
│   └── services/
│       ├── extractor.py     # GitHub repo cloning & analysis (Radon)
│       ├── file_reader.py   # PDF / DOCX / Markdown text extraction
│       └── stellar_verifier.py  # Stellar wallet & contract verification
├── test_grading.py          # Quick end-to-end test script
├── requirements.txt         # Python dependencies
├── Dockerfile               # Docker setup
└── .env                     # Your API keys (create from .env.example)
```

---

## Prerequisites

- **Python 3.11+** — [Download here](https://www.python.org/downloads/)
- **An Anthropic API key** — [Get one here](https://console.anthropic.com/)

---

## Setup & Installation

### 1. Clone the repository (if you haven't already)

```bash
git clone <your-repo-url>
cd ai-grading-service
```

### 2. Create a virtual environment

A virtual environment keeps this project's dependencies isolated from your system Python.

```bash
python3 -m venv venv
```

### 3. Activate the virtual environment

**macOS / Linux:**
```bash
source venv/bin/activate
```

**Windows:**
```bash
venv\Scripts\activate
```

> You'll see `(venv)` appear at the start of your terminal prompt — this means it's active.

### 4. Install dependencies

```bash
pip install -r requirements.txt
```

### 5. Create your `.env` file

```bash
cp .env.example .env
```

Then open `.env` and add your Anthropic API key:

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

### 6. Run the server

```bash
uvicorn app.main:app --reload
```

The API will be available at **http://localhost:8000**.

Interactive API docs (Swagger UI) are at **http://localhost:8000/docs**.

---

## Running the Test Script

To quickly verify everything works end-to-end (requires a valid `ANTHROPIC_API_KEY`):

```bash
python3 test_grading.py
```

This will grade a sample Stellar project and print the scores to the terminal.

---

## API Usage

### `POST /grading/hackathon`

Grades a hackathon submission.

**Request body:**

```json
{
  "submission": {
    "submission_id": "sub_001",
    "team_name": "Team Stellar",
    "project_name": "My DeFi App",
    "tagline": "Decentralized lending on Stellar",
    "description": "A full DeFi lending protocol built with Soroban smart contracts.",
    "github_url": "https://github.com/my-org/my-defi-app",
    "live_demo_url": "https://my-defi-app.vercel.app",
    "stellar_address": "GBXXXX...",
    "contract_id": "CCXXXX...",
    "file_urls": ["https://example.com/pitch-deck.pdf"],
    "hackathon_context": {
      "name": "Stellar DeFi Sprint 2026",
      "description": "Build innovative DeFi applications on Stellar.",
      "judging_criteria": "1. Innovation (25%) ...",
      "duration_hours": 48
    }
  }
}
```

**Response:**

```json
{
  "success": true,
  "processing_time_seconds": 12.4,
  "result": {
    "overall_score": 7.8,
    "recommendation": "ACCEPT",
    "confidence_level": "HIGH",
    "innovation": { "score": 8.0, "reasoning": "...", "strengths": [], "weaknesses": [] },
    "technical_execution": { "score": 7.5, "reasoning": "...", "strengths": [], "weaknesses": [] },
    "stellar_integration": { "score": 8.0, "reasoning": "...", "strengths": [], "weaknesses": [] },
    "ux_design": { "score": 7.0, "reasoning": "...", "strengths": [], "weaknesses": [] },
    "completeness": { "score": 8.0, "reasoning": "...", "strengths": [], "weaknesses": [] },
    "standout_features": ["Live demo deployed", "Soroban contract verified on-chain"],
    "improvement_suggestions": ["Add more unit tests"],
    "red_flags": [],
    "graded_at": "2026-03-24T08:00:00"
  }
}
```

---

## Running with Docker

If you prefer Docker instead of setting up Python manually:

```bash
# Build the image
docker build -t ai-grading-service .

# Run the container
docker run -p 8000:8000 --env-file .env ai-grading-service
```

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | ✅ Yes | Your Anthropic API key for Claude access |

---

## Tech Stack

| Library | Purpose |
|---|---|
| [FastAPI](https://fastapi.tiangolo.com/) | Web framework & API |
| [Anthropic](https://pypi.org/project/anthropic/) | Claude AI SDK |
| [GitPython](https://gitpython.readthedocs.io/) | Git repo cloning |
| [Radon](https://radon.readthedocs.io/) | Code complexity metrics |
| [PyPDF2](https://pypdf2.readthedocs.io/) | PDF text extraction |
| [python-docx](https://python-docx.readthedocs.io/) | DOCX text extraction |
| [stellar-sdk](https://stellar-sdk.readthedocs.io/) | Stellar network verification |
| [Pydantic](https://docs.pydantic.dev/) | Data validation |
| [python-dotenv](https://pypi.org/project/python-dotenv/) | `.env` file loading |

---

## License

See [LICENSE.md](LICENSE.md).

# AI Research Agent — Parallel Multi-Agent Pipeline

A research assistant built with LangChain 1.x `create_agent` and LangGraph `StateGraph`.
Given a question, it researches the topic from two angles **in parallel**, analyzes the
findings, and writes a beginner-friendly report with sources.

## Architecture (non-sequential: parallel fan-out / fan-in)

    START ──┬──> researcher_facts   ──┐
            └──> researcher_context ──┴──> analyst ──> writer ──> END

| Node | Job | Tools |
|------|-----|-------|
| `researcher_facts` | Finds definitions, key facts, and data | `web_search`, `read_webpage` |
| `researcher_context` | Finds use cases, advantages, and disadvantages | `web_search`, `read_webpage` |
| `analyst` | Waits for both branches, compares and combines them | none |
| `writer` | Turns the analysis into a clear report with sources | none |

- Both researchers use the same `create_agent` researcher, but with different tasks.
- Each branch writes to its **own State key** (`research_facts`, `research_context`),
  so the parallel branches never overwrite each other.
- The analyst runs only after **both** branches finish (fan-in).

## Reliability

- **Search fallback:** `web_search` tries DuckDuckGo first and automatically falls back
  to the free Wikipedia API when DuckDuckGo is blocked or returns nothing.
- **Safe agent calls:** `safe_run` catches agent errors and empty answers and passes a
  clear fallback message to the next stage instead of crashing the pipeline.
- **Step budget:** `ToolCallLimitMiddleware` caps the researcher's tool calls.
- **Observability (given):** every agent run prints a trace with tokens and real OpenRouter cost.

## Setup and run (local)

    uv sync
    copy .env.example .env      # Windows (use cp on Mac/Linux), then paste your OPENROUTER_API_KEY
    uv run jupyter lab research_agent.ipynb

Then run all cells top to bottom (skip the Colab `pip install` cell, it is commented out).
The last code cells draw the graph and run a live query: `Compare RAG and fine-tuning`.

Never commit your `.env` file. It holds your API key and is already in `.gitignore`.

## Project structure

    ├── research_agent.ipynb   # setup, tools, agents, parallel graph, live run
    ├── EVALUATION.md          # grading rubric
    ├── pyproject.toml         # dependencies (managed with uv)
    ├── uv.lock                # locked dependency versions
    ├── .env.example           # environment variable template
    └── .gitignore             # keeps .env out of git

Submitted by: Amal Alotaibi — academy: @SDAIAAcademy
[github.com/SDAIAAcademy](https://github.com/SDAIAAcademy).
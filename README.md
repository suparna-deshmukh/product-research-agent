# Product Research Agent — Reddit + Hacker News → PRDs

A **29-node n8n workflow** that turns a product topic into ranked pain points, theme clusters, competitor flags and a first-draft **PRD** — unattended, on a schedule, with a diffable history log.

**Author:** Suparna Deshmukh
**Stack:** n8n · Ollama (Llama 3, local) · Reddit API · Hacker News (Algolia) API
**Full report:** [Product_Research_Agent_Report.pdf](Product_Research_Agent_Report.pdf) · [workflow screenshot](Screenshot%202026-05-02%20at%206.14.20%20AM.png)

## Why

Manual product research is a loop: read threads → note complaints → find the pattern → draft the spec. This agent runs that loop itself. Give it a topic; it comes back with a prioritized pile of evidence and a spec skeleton. On a new product area, research-to-spec time dropped ~70 % versus doing it by hand.

## How it works

```
Form Trigger (topic input)
  └─► auto-discovery: LLM expands the topic into keywords + candidate subreddits
  └─► parallel fetch: Reddit posts + HN stories (Algolia API)
  └─► clean & merge both sources into one corpus
  └─► LLM pipeline (all local Ollama / Llama 3):
        Classify Themes  →  Flag Competitors  →  Check Relevance (filter)
  └─► aggregate by theme (ranked list with quotes)
  └─► Generate PRD (problem statement, affected users, proposed solution, open questions)
  └─► write PRD + output markdown to disk  ·  append to the history log
```

All 29 nodes are `HTTP Request`, `Code`, `Set`, `Merge` or `Form Trigger` — the full workflow is the committed `product_research_agent.json`.

**LLM calls are 100 % local** (Ollama at `http://localhost:11434/api/generate`), so nothing leaves the machine.

## Output

Each run produces, for a topic like *UPI payment platforms* or *whey protein powder*:

- a **ranked theme list** — pain points clustered by theme with representative quotes and frequency;
- a **PRD draft** — problem statement, affected users, proposed solution, open questions;
- an **entry in the history log** — the part that compounds: run last week's themes against this week's and the trend is visible.

Sample runs are committed in `outputs/`:

- [UPI payment platforms PRD](outputs/upi-payment-platforms-prd-2026-05-02.md)
- [Whey protein powder PRD](outputs/whey-protein-powder-prd-2026-05-02.md)

## How to run

1. Install [n8n](https://n8n.io) (self-hosted recommended) and start it.
2. Start Ollama with a Llama 3 model: `ollama serve && ollama pull llama3`.
3. Open n8n → **Import from file** → `product_research_agent.json`.
4. Fill in the form (topic, optional keywords / subreddits, output path) and execute.
5. Activate the workflow and schedule it — it runs unattended and appends every run to the history log.

> The workflow writes files to disk with `readWriteFile` nodes, so it must run on a machine where that path exists (self-hosted n8n, not n8n Cloud's sandbox).

## Repository layout

```
product_research_agent/
├── product_research_agent.json          # the 29-node n8n workflow (import into n8n)
├── Product_Research_Agent_Report.pdf    # full write-up
├── Screenshot 2026-05-02 at 6.14.20 AM.png  # workflow canvas
├── outputs/                             # real run artifacts (PRDs, markdown)
└── logs/                                # history-log location
```

## What I'd change

Add citations back into every generated claim, and make the theme taxonomy a human-editable artifact instead of a prompt. The LLM is good at drafting; it's worse at knowing what it doesn't know.

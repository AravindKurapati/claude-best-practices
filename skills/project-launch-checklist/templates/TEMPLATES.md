# README Template — For Hiring Manager Speed-Read (2 min)

```markdown
# [Project Name]

## What This Does
[One sentence explaining what it does]

## The Problem
[Why this matters — what problem does it solve? Real, not theoretical.]

## Live Demo
[Link to Streamlit/Gradio/HF Spaces, screenshot, or Loom video]

## Key Features
- [Feature 1]
- [Feature 2]
- [Feature 3]

## Quick Start
\`\`\`bash
# Clone and setup
git clone [repo]
cd [project]
pip install -r requirements.txt

# Run (or run the hosted demo)
python main.py
\`\`\`

## Architecture
See [`ARCHITECTURE.md`](./documents/ARCHITECTURE.md) for design decisions.

## Evaluation (if applicable)
Results on [dataset]: [metric] = [number]
See [`EVAL.md`](./documents/EVAL.md) for full benchmarks.

## Config
Customize in `config.yaml`. See `.env.example` for environment variables.

## Tests
\`\`\`bash
pytest tests/
\`\`\`

## Known Limitations
[Be honest about what this doesn't do.]

## Future Work
[What's next?]
```

---

# GitHub Actions Template — CI/CD in 5 Minutes

Save as `.github/workflows/ci.yml`:

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest tests/ -v --cov=src

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pip install flake8
      - run: flake8 src/ --max-line-length=100
```

---

# config.yaml Template

```yaml
# Model & API
model:
  name: "claude-3.5-sonnet"
  temperature: 0.7
  max_tokens: 2000

# Data
data:
  input_path: "data/input"
  golden_path: "data/golden"
  batch_size: 32

# Evaluation
eval:
  metrics: ["precision@k", "mrr", "token_cost"]
  k: 10

# Deployment
deployment:
  host: "0.0.0.0"
  port: 8000
  workers: 4
```

---

# .env.example

```bash
# API Keys
ANTHROPIC_API_KEY=your_key_here
OPENAI_API_KEY=your_key_here

# Config
MODEL_NAME=claude-3.5-sonnet
BATCH_SIZE=32
LOG_LEVEL=INFO

# Data paths
INPUT_PATH=./data/input
OUTPUT_PATH=./data/output
```

---

# ARCHITECTURE.md Template

```markdown
# Architecture

## Overview
[Diagram or ASCII art of the system]

## Components

### [Component 1]: [Purpose]
- Responsibility: [what it does]
- Dependencies: [what it needs]
- Design choice: [why this approach]

### [Component 2]: [Purpose]
- Responsibility: [what it does]
- Dependencies: [what it needs]
- Design choice: [why this approach]

## Data Flow
[Explain how data moves through the system]

## Key Tradeoffs
| Approach | Pros | Cons |
|---|---|---|
| A | + [pro] | - [con] |
| B | + [pro] | - [con] |
| **Chosen: A** | Chose A because [reasoning] | |

## Evaluation Strategy
[How you measure success]

## Known Limitations
[Be honest about constraints]
```

---

# EVAL.md Template

```markdown
# Evaluation

## Setup
Dataset: [name, size, source]
Baseline: [what you're comparing against]
Metrics: [how you measure success]

## Results

| Approach | Metric 1 | Metric 2 | Cost |
|---|---|---|---|
| Baseline | X | Y | $Z |
| **Proposed** | **X** | **Y** | **$Z** |

## Analysis
[Breakdown of why proposed approach wins/loses]

## Golden Dataset
See [`data/golden/`](../data/golden/) for examples.

## Running Evals
\`\`\`bash
python scripts/eval.py --model [model] --dataset golden
\`\`\`
```

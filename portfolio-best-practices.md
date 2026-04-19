# Portfolio Best Practices — Hiring Visibility & Project Launch

## The Reality of How Your Work Gets Seen
- Hiring managers spend **5–10 minutes** on your GitHub repo. They scan README, check if project exists, move on.
- Recruiters spend **< 2 minutes** total. They care that a project exists; they don't read code.
- For take-home assignments, hiring managers actually run the code and read carefully.
- Nobody looks at commit history. Projects are judged on their current state, not historical development.

## What Hiring Managers Look For (In Order)
1. **Clear description** — what it does, why it exists, what problem it solves
2. **Production signals** — tests, evaluation harness, CI/CD, deployment link
3. **Accessibility** — live demo (Streamlit, Gradio, HF Spaces) beats static README
4. **Code quality** — less important than clarity and tests, but each adds a plus

## The README — Your Highest-Value Asset
The README is the **only thing most people read**. Optimize for two audiences:

### Audience 1: Impatient Hiring Manager (5 min)
- What does this project do? (one sentence)
- What problem does it solve? (real, not theoretical)
- Can I see it live? (link or screenshot)
- Is it legit? (tests, eval, CI/CD badge)

### Audience 2: Peer Reviewer (thorough)
- Architecture diagram or explanation
- How to run it locally
- Key prompts or config highlighted
- Evaluation results with numbers
- Links to deepest components (eval harness, prompts, cost analysis)

## AI-Generated READMEs Are Dead Giveaways
- Don't use auto-generated READMEs; write or heavily edit them
- Filler kills signal. Clarity is the goal
- One well-written README beats five mediocre ones

## What to Actually Build (Avoid Over-Engineering)
### Easy Wins (Always Include)
- Tests (unit, integration, ideally e2e)
- GitHub Actions CI/CD (runs tests on every push)
- Config file (so people can experiment with parameters)
- A live deployment link or demo video

### Worth Building
- Evaluation harness (golden datasets, metrics, before/after comparisons)
- Cost analysis (token counts, API call breakdowns, monthly projection)
- Smart routing logic (show decision-making, not just calling one model)
- Deployment diagram (Docker, K8s, Prometheus/Grafana — only if it genuinely exists)

### Probably Over-Engineering
- Production-grade infrastructure on a personal project
- Enterprise-level monitoring for toy code
- Multiple deployment regions for a solo project

> If you genuinely need production-level infrastructure, you don't need a job — you have one.

## The Checklist — Before Making a Project Public
Use this before every `git push` to public:

- [ ] **README clarity** — can someone understand what this does in 2 minutes?
- [ ] **Problem statement** — "solves X" beats "demonstrates Y"
- [ ] **Live demo link** — Streamlit, Gradio, HF Spaces, or Loom video walking through it
- [ ] **Unit tests** — at least cover the core logic
- [ ] **CI/CD** — GitHub Actions that run tests on push (5-minute setup)
- [ ] **Config file** — `config.yaml` or `.env.example` so people can tune it
- [ ] **Eval harness** — for RAG/ML projects, include golden datasets + metrics
- [ ] **Cost breakdown** — for LLM projects, show tokens/cost per operation
- [ ] **Architecture doc** — in `/documents/ARCHITECTURE.md` for peer reviewers
- [ ] **Open issues or TODOs** — shows you know what's incomplete (credible)

Not all projects need all of these. But the more you check, the stronger the signal.

## Multi-Project Strategy (More Signals > Fewer Perfect Projects)
Build **5–6 small focused projects** (1–2 weeks each) instead of one large project:
- Project 1: Multi-agent orchestration + error handling
- Project 2: RAG with evaluation framework
- Project 3: Cost optimization (routing, caching, compression)
- Project 4: End-to-end deployment (Docker, CI/CD, monitoring)
- Project 5: Real-time feature engineering or recommendation engine

At the end: you have a portfolio showing breadth + depth across the AI stack.

## Domain-Specific Projects (Higher Signal)
When targeting specific companies or domains:
1. Read their engineering blog — understand their problems
2. Build 2–3 projects in that domain using relevant datasets
3. At interviews, you have concrete things to discuss
4. Reference their tech choices in your README ("we use Postgres like company X does for...")

## For Take-Home Assignments
Hiring managers actually run these and read code carefully:
- Write tests (unit, integration, ideally end-to-end)
- Aim for code coverage
- Follow engineering best practices (logging, error handling, clear variable names)
- Include comments on tricky logic
- Ship a Loom video walking through design + running the code

## What NOT to Do
- Don't build perfect code on a useless project
- Don't include filler in README (AI generates this automatically)
- Don't over-engineer — clarity beats production polish on personal projects
- Don't hope people dig into your code — put the best signals in the README
- Don't make projects with no eval or evaluation strategy (RAG without eval is incomplete)

## Key Insight: Tests + CI/CD Are Force Multipliers
- Tests are easy to write (especially with Claude Code)
- CI/CD is a 5-minute GitHub Actions wrap around tests
- But together, they signal "this person ships real code"
- A messy project with tests > a clean project with none

## Red Flags You Want to Avoid
- Broken links in README
- No way to actually run the code
- Claims that don't match reality ("production-ready" but no tests)
- Generic project (another RAG tutorial with no unique angle)
- README that needs 10 minutes to understand what the project does

## Green Flags That Impress
- ✅ Real problem statement
- ✅ Live demo link (not just code)
- ✅ Evaluation metrics with numbers
- ✅ Tests + CI/CD badge
- ✅ Config file for experimentation
- ✅ Cost breakdown (for LLM projects)
- ✅ Architecture diagram
- ✅ Known limitations / future work (shows self-awareness)
- ✅ Open-source contributions linked (signal of depth)

## The Progression
```
Incomplete project (no tests, no eval)
    ↓ (10 min to add tests + CI/CD)
Solid project (tests, CI/CD, clear README)
    ↓ (add eval harness, cost analysis)
Strong project (all above + deployment proof)
    ↓ (add domain expertise + multi-project breadth)
Portfolio that gets offers
```

---

## Reference
- Based on interviews with recruiters, hiring managers, and interview prep companies
- Consensus: a single high-quality project with eval + tests + README > multiple certifications
- Live demo + clear README > pristine code that nobody sees

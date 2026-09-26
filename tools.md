# tools.md

every tool trainery uses, what it is for, budget, and usage rules.

---

## ingestion tools

### firecrawl
**role:** documentation ingestion — turns urls into clean markdown
**budget:** 10,000 credits
**credit rates:**
- /scrape — 1 credit per page
- /crawl — 1 credit per page discovered
- /map — 1 credit per call (returns all urls on a domain)
- /search — 2 credits per 10 results
- /interact (browser session) — 2 credits per browser minute
- enhanced / stealth mode — +4 credits per page (total: 5)

**usage rules:**
1. always run /map first before /crawl — costs 1 credit, saves hundreds
2. review the url map before scraping — do not crawl everything
3. avoid enhanced mode unless the page fails in normal mode
4. store every scraped page with: source url, date, credit cost
5. do not re-scrape pages that have not changed

**budget allocation:**
- p5.js ingestion (reference + learn + examples): 800–1,200 credits
- react ingestion (selected pages): 400–600 credits
- vite ingestion (selected pages): 100–200 credits
- python ingestion (selected pages): 400–600 credits
- examples + counterexamples + tavily follow-up scrapes: 1,000–2,000
- reserve for three.js and openscad (later): 4,000–5,000
- buffer: 1,000

**p5.js priority urls:**
```
https://p5js.org/reference/        — full api reference
https://p5js.org/learn/            — concept tutorials
https://p5js.org/examples/         — working code examples
```

---

### tavily
**role:** discovery — finds canonical examples, counterexamples,
         debugging guides, community knowledge
**budget:** standard api tier (check current limits)

**use tavily for:**
- "p5.js common mistakes beginners"
- "p5.js coordinate system confusion"
- "p5.js vs processing differences"
- "p5.js for react developers"
- "p5.js map function vs javascript map"
- specific concept searches when official docs are thin
- finding stack overflow threads that encode real user confusion

**do not use tavily as an authority.**
it is a discovery tool. everything found via tavily needs
to be verified against official sources before entering the graph.

---

## graph store

### falkordb cloud
**role:** knowledge graph — stores concepts and transfer relationships
**plan:** free tier
**limits:** 100mb ram, max graph dataset size 100mb
**hosting:** aws or gcp (choose gcp to align with other gcp credits)

**critical constraints:**
- free instances stopped after 1 day of inactivity
- deleted after 7 days of inactivity
- no persistence, no backups on free tier
- data cannot be recovered after deletion

**mitigation:**
- run falkordb locally via docker during development
- use the cloud instance only during active sessions
- export graph snapshots to gcp storage regularly
- automate a daily ping if leaving the cloud instance running

**local docker:**
```bash
docker run -p 6379:6379 falkordb/falkordb:latest
```

**schema:** see graph.md for current node types, relationship types,
and property definitions.

**connection:**
```python
from falkordb import FalkorDB
db = FalkorDB(host='localhost', port=6379)
g = db.select_graph('trainery')
```

---

## inference

### nebius token factory
**role:** llm inference for curriculum generation, graph population,
         and evaluation
**budget:** $25 credits
**pricing (approximate, check current rates):**
- llama 3.1 8b instruct: $0.02/m input, $0.06/m output
- llama 3.3 70b instruct: $0.12/m input, $0.30/m output
- qwen2.5 coder 7b: $0.01/m input, $0.03/m output
- deepseek v3: check current pricing

**use nebius for:**
- bulk curriculum example generation (use a smaller model)
- graph population from ingested docs (structured extraction)
- evaluation scoring where lm-as-judge is used as a fallback

**do not use nebius for:**
- casual exploration or testing prompts (use watsonx free tier instead)
- any call where you do not need to record the output

**budget estimate:**
at $0.06/m output tokens for a 70b model,
$25 buys roughly 400m output tokens.
at ~500 tokens per curriculum example, that is ~800,000 examples.
in practice, budget for 50,000–100,000 quality examples with
overhead for graph population and evaluation.

**record every call:**
- model used
- purpose
- approximate tokens
- date

---

## agentic layer

### backboard.io
**role:** stateful agent layer — persistent memory, model routing,
         built-in rag, unified api across models
**budget:** $30 credits
**models available:** 17,000+ including all major frontier and open models

**use backboard for:**
- the transfer engine (stateful multi-step graph traversal + generation)
- curriculum generation pipelines that need memory across steps
- any workflow where session state matters between calls
- agent coordination (e.g., ingest agent + graph agent + eval agent)

**key features relevant to trainery:**
- built-in rag (index docs, retrieve context, ground responses)
- persistent memory across sessions
- model routing (route cheap tasks to small models, hard tasks to large)
- openai-compatible api (easy to swap models)

**do not use backboard for:**
- one-off inference calls (use nebius directly)
- anything where you need per-token cost accounting
  (backboard abstracts this; use nebius for tracked bulk generation)

---

## browser evaluation

### browserbase developer plan
**role:** real browser execution for p5.js and three.js evaluation
**plan:** developer — $20/month
**concurrency:** 25 concurrent sessions
**session limit:** 25 new sessions per 60-second window

**the p5.js evaluation loop:**
```
generated p5.js code
  ↓
browserbase playwright session
  ↓
load html page with p5 cdn:
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.min.js"></script>
  <script>/* inject generated sketch here */</script>
  ↓
wait for setup() to complete
wait for N draw() frames (use frameCount)
  ↓
page.screenshot() — capture canvas
  ↓
pixel analysis or lm-as-judge on screenshot
  ↓
evaluation result
```

**session hygiene:**
- always close sessions after use
- use context managers / try-finally to ensure cleanup
- do not leave sessions open overnight (they count against your quota)

**what browserbase enables that nothing else does:**
- javascript execution (canvas renders, animation frames)
- real browser environment (same as p5 users see)
- screenshot capture for visual evaluation
- dom inspection for checking rendered state

**for openscad evaluation:** browserbase is not useful.
openscad requires a native binary. use e2b or a gcp cloud run
container with openscad installed.

---

## development infrastructure

### google cloud platform
**role:** orchestration, storage, notebooks, cloud run
**use for:**
- gcs bucket: store ingested markdown, training data, experiment records
- cloud run: host evaluation runners (openscad container, eval api)
- vertex ai notebooks: exploratory graph analysis, data inspection
- cloud scheduler: daily falkordb ping to prevent inactivity deletion

**storage structure:**
```
gs://trainery-data/
  raw/              — ingested pages
  curriculum/       — training examples
  experiments/      — experiment records and results
  graph-snapshots/  — regular falkordb exports
  eval/             — evaluation task definitions and results
```

---

### ibm watsonx.ai (free tier)
**role:** supplemental inference, prompt lab experimentation
**limits:** 300k tokens free for new trials, 10 cuh/month
**models:** ibm granite, llama, mistral (via ibm)

**use watsonx for:**
- testing prompts before spending nebius credits
- prompt lab exploration (no code needed)
- lightweight inference tasks that do not require scale

**do not use watsonx for:**
- fine-tuning or weight updates (not available on free tier)
- bulk generation (use nebius)

---

### ibm bob 2.0
**role:** coding assistant with full repository context
**access:** via lablab.ai / ibm developer

**use bob for:**
- navigating the trainery codebase
- generating boilerplate (ingestion scripts, graph queries)
- understanding how existing code is structured
- debugging during development

**bob is a dev tool, not research infrastructure.**
it does not replace the experiment. it helps build it faster.

---

## budget summary

| tool | budget | primary use | phase |
|------|--------|-------------|-------|
| firecrawl | 10,000 credits | ingestion | 1 |
| tavily | api tier | discovery | 1 |
| falkordb cloud | free (100mb) | graph store | 2 |
| nebius token factory | $25 | bulk inference | 3, 5 |
| backboard.io | $30 | agentic pipeline | 3, 4 |
| browserbase developer | $20/mo | browser eval | 4, 5 |
| gcp credits | varies | storage + compute | all |
| watsonx.ai | 300k tokens | supplemental | 1–3 |
| ibm bob 2.0 | dev access | coding | all |

---

## tool hierarchy for inference decisions

```
is this a test / exploration?
  → watsonx.ai free tier (cost: $0)

is this a stateful multi-step agent task?
  → backboard.io ($30 budget)

is this bulk generation at scale (curriculum, graph population)?
  → nebius token factory ($25 budget)

does this require a real browser (p5.js, three.js eval)?
  → browserbase developer plan ($20/mo)

does this require documentation ingestion?
  → firecrawl (10k credits) + tavily (discovery)
```

# trainery

> teaching an ai agent new programming environments  
> without making it forget that it already knows how to program.

---

## what this is

trainery is a research project investigating one question:

> can an ai agent learn a new programming environment more efficiently  
> when its training is informed by programming knowledge it already has?

the hypothesis is that programming knowledge transfers.

if an agent already understands functions, composition, iteration,  
abstraction, and coordinate systems — it should not have to learn  
those concepts from zero every time it encounters a new environment.

the syntax may be new. the api may be new. the domain may be new.  
but some of the underlying ideas are already there.

trainery tries to make that transfer explicit, structured, and measurable.

---

## the research question

**primary**

> can structured transfer of existing programming concepts reduce  
> the data or compute required for target-domain proficiency?

**secondary**

> does learning one new environment create useful prior knowledge  
> for learning another?

**eventual**

> can an agent's programming knowledge become cumulative —  
> where each new environment learned makes the next one easier?

---

## current domains

### source domains — what the agent already knows

| domain | provides |
|--------|----------|
| react | components, composition, props, state, event handling, declarative construction |
| vite | javascript/typescript projects, modules, imports, build environments, project structure |
| python | functions, classes, iteration, collections, control flow, modules, scripting |

### target domains — what we are teaching

| domain | status | why |
|--------|--------|-----|
| p5.js | **active — first experiment** | closest transfer case: javascript → javascript, new domain abstractions |
| three.js | planned | familiar language, unfamiliar 3d domain |
| openscad | planned | new language + new domain — most interesting transfer case |
| cadquery | planned later | familiar language (python), unfamiliar cad domain |

---

## how it works

conventional approach:

```
target documentation
        ↓
   training data
        ↓
      model
        ↓
  "learn p5.js"
```

trainery approach:

```
what does the agent already know?
        ↓
  existing concepts
        ↓
  transfer graph
  (what is shared, what differs, where analogies fail)
        ↓
  what is actually new here?
        ↓
  curriculum built around prior knowledge
        ↓
  target-domain training
        ↓
    proficiency
```

the difference is that prior knowledge is used as training infrastructure,  
not just retrieved at inference time.

---

## the transfer graph

the core of trainery is a knowledge graph in falkordb.

nodes represent: languages, frameworks, concepts, constructs, apis,  
examples, tasks, constraints, and failure modes.

relationships represent structured assertions:

```
(react:component_lifecycle)
  -[PARTIAL_ANALOGY {
      shared: "initialization + recurring update phase",
      different: "react re-renders on state change; p5 loops unconditionally",
      failure_case: "assuming draw() only runs when something changes"
  }]->
(p5js:setup_draw_loop)
```

```
(python:map_builtin)
  -[CONTRASTS_WITH {
      relationship: "same function name",
      difference: "python map() applies fn to iterable; p5 map() remaps a number between ranges",
      confusion_risk: "high"
  }]->
(p5js:map_fn)
```

encoding where analogies **fail** is as important as encoding where they work.  
a system that learns only "these things are similar" — without learning  
"this is where the similarity ends" — may perform worse than no transfer at all.

---

## the three experimental conditions

every target domain is evaluated under three conditions:

**a — target only**
```
p5.js data only → model → p5.js evaluation
```

**b — source + target**
```
react + vite + python data + p5.js data → model → p5.js evaluation
```

**c — transfer-aware (the trainery intervention)**
```
source knowledge + transfer graph + p5.js data → model → p5.js evaluation
```

the comparison isolates the contribution of structured transfer.  
a better score from condition c is only meaningful if compute is held constant.

---

## evaluation

trainery requires execution-based evaluation where possible.  
lm-as-judge is a fallback, not a primary signal.

for p5.js:

```
generated p5.js sketch
        ↓
browserbase (playwright session)
        ↓
canvas renders in real browser
        ↓
screenshot
        ↓
pixel check / visual evaluation
        ↓
pass / fail / score
```

for three.js:

```
generated three.js code → browser → 3d scene → evaluation
```

for openscad:

```
generated openscad → openscad binary → geometry → geometric evaluation
```

---

## what success looks like

the project is successful if it produces **evidence that answers the research question** —  
not merely if it produces a model, a graph, a dashboard, or a demo.

the ideal outcome is something like:

> given equivalent compute, transfer-aware training required fewer  
> target-domain examples to reach proficiency in p5.js.

the opposite result is equally valuable if measured rigorously.  
a failed transfer is a result.

---

## what this is not

| this is not | why |
|-------------|-----|
| a rag system | rag retrieves at inference time. trainery uses prior knowledge at training time. |
| a prompt engineering project | we are not getting a model to answer one question better. we are changing how it learns. |
| a demo | a beautiful demo with an invalid experimental comparison is not a successful contribution. |
| an analogize clone | analogize discovers transfer relationships. trainery uses them to train and evaluates whether transfer actually helped. |

---

## infrastructure

| tool | role |
|------|------|
| firecrawl | documentation ingestion — urls to clean markdown |
| tavily | discovery — examples, counterexamples, community knowledge |
| falkordb | knowledge graph and transfer graph store |
| nebius token factory | llm inference — curriculum generation, graph population |
| backboard.io | stateful agent layer — persistent memory, model routing |
| browserbase | real browser evaluation for p5.js and three.js |
| google cloud | orchestration, storage, notebooks, cloud run |
| aws trainium | training backend (phase 7 — deferred) |

---

## repository structure

```
/
├── readme.md          ← you are here
├── explain.md         ← what trainery is and why
├── scope.md           ← what is in and out of scope
├── plan.md            ← implementation plan, phase by phase
├── agent.md           ← instructions for ai agents working on this repo
├── tools.md           ← tool reference and budget tracking
├── graph.md           ← falkordb schema reference
├── p5js.md            ← everything specific to the p5.js target domain
├── journal.md         ← research log (append only)
│
├── data/
│   ├── raw/           ← ingested markdown, one file per source page
│   ├── curriculum/    ← generated training examples
│   └── experiments/   ← experiment records and results
│
├── eval/
│   └── p5js/          ← evaluation tasks and browserbase runner
│
├── graph/
│   ├── schema.md      ← current schema (mirrors graph.md)
│   └── migrations/    ← schema change records
│
└── scripts/
    ├── ingest/        ← firecrawl + tavily ingestion
    ├── graph/         ← falkordb population and queries
    ├── curriculum/    ← training example generation
    └── eval/          ← evaluation runners
```

---

## current status

**phase 0 — foundations** ✓ in progress

- [x] research specification written
- [x] graph schema designed (v0.1)
- [x] tool stack confirmed and budgeted
- [ ] repository structure initialized
- [ ] falkordb instance running
- [ ] first firecrawl map of p5js.org

**phase 1 — ingestion** upcoming

**phase 2 — concept graph** upcoming

**phase 3 — curriculum generation** upcoming

**phase 4 — evaluation benchmarks** upcoming

**phase 5 — baseline experiments** upcoming

---

## for ai agents working on this repo

read `agent.md` before modifying anything.

the short version:

- this is research software. reproducibility matters more than features.
- do not silently change graph schema, training data, or evaluation logic.
- do not encode hard equivalences between concepts (`==`). use qualified relationships.
- do not replace the graph with vector similarity.
- do not let llm-generated content become canonical knowledge without labeling it.
- failed transfer is a result. record it.

---

## related

**analogize** — [analogize.annecrypted.com](https://analogize.annecrypted.com)  
a sibling project focused on representation and discovery of conceptual relationships between technical domains.

analogize asks: *what can transfer?*  
trainery asks: *does transferring it actually help an agent learn?*

---

## research discipline

statements in this repository should be clearly labeled as one of:

- **hypothesis** — what we think might be true
- **measurement** — what an experiment actually observed
- **interpretation** — what we think the measurement means
- **speculation** — informed guessing, not yet tested

avoid statements like "transfer makes models learn faster"  
unless supported by a specific experiment result.

prefer: "we hypothesize that transfer-aware training will require  
fewer target-domain examples to reach equivalent proficiency."

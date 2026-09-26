# agent.md

instructions for ai agents working on trainery.
read this before modifying anything.

---

## what you are working on

trainery is research software.
the goal is evidence, not features.

before you write any code, ask:
> does this change protect or undermine the experiment?

if you are not sure, check scope.md and plan.md before proceeding.

---

## the files you need to read first

| file | read when |
|------|-----------|
| explain.md | before anything — what trainery is |
| scope.md | before building anything — what is in and out |
| plan.md | before coding — what phase we are in |
| graph.md | before touching falkordb or schema |
| tools.md | before using any external service |
| journal.md | before starting a session — what happened last |

---

## hard rules

### 1. do not silently change experimental definitions

if you change any of these, create a new versioned artifact
and document the change:
- graph schema
- training data
- evaluation tasks or scoring logic
- baseline definitions
- experiment configuration

a better benchmark score obtained by changing the benchmark
is scientific fraud. do not do it.

### 2. do not encode hard equivalences between concepts

never write:
```
react.component == openscad.module
```

always use qualified relationships:
```
analogous_to
partial_analogy
conceptually_related_to
contrasts_with
generalizes
specializes
```

and always record what is shared, what differs, and where it fails.

### 3. do not replace the graph with vectors

vectors are for semantic retrieval.
the graph is for explicit structural relationships.

do not replace a graph query with vector cosine similarity.
they answer different questions.

### 4. do not let llm output become canonical knowledge

if you use an llm to generate an explanation, label it:
```
provenance: llm-generated
model: [model name]
date: [date]
review_status: unverified
```

do not merge llm-generated content into the graph as if it
came from official documentation.

### 5. do not regenerate training data silently

if you regenerate a training dataset:
- give it a new version identifier
- record what changed and why
- do not overwrite the previous version
- do not run the same experiment on new data and call it a replication

### 6. do not jump to trainium

trainium is phase 7.
do not optimize training code, write nki kernels, or
spend time on training hardware until phases 1–6 are working.

---

## before running firecrawl

1. use /map first — 1 credit, returns all urls on a domain
2. review the url list — scrape only what is needed
3. avoid enhanced/stealth mode where possible — costs 5x credits
4. record what was scraped: url, date, credit cost

firecrawl budget: 10,000 credits total.
p5.js ingestion target: 3,000–5,000 credits.
reserve: 5,000–7,000 for re-ingestion, examples, other targets.

---

## before writing to falkordb

check graph.md for the current schema.
do not invent new node types or relationship types without documenting them.

if a new relationship type is needed, answer first:
> what question does this relationship type answer
> that existing types cannot?

if you cannot answer that, use an existing type.

---

## before calling nebius / backboard

record the call:
- model used
- prompt (or prompt version if templated)
- purpose (curriculum gen / graph population / evaluation / other)
- approximate token cost
- date

this goes in the session log or journal.md.

---

## before building evaluation

evaluation tasks must be defined before training begins.
do not design or modify evaluation tasks after seeing training results.

for p5.js evaluation:
- the browserbase loop must be tested with a simple known-good sketch first
- record the test: what sketch, what was expected, what was observed
- only scale the evaluation after the loop is verified

---

## what a good session looks like

**start:**
- read journal.md — what was the last state?
- check which checkpoint in plan.md we are at
- identify one concrete deliverable for this session

**during:**
- one thing at a time
- record decisions as you make them
- if something surprises you, note it immediately

**end:**
- write a journal.md entry:
  - what was attempted
  - what worked
  - what failed or surprised
  - what changed
  - what is next
- commit with a message that describes what actually changed

---

## what to do when an analogy seems wrong

if you are encoding a transfer relationship and it feels off:

1. write down what feels wrong specifically
2. find a concrete example where the analogy breaks
3. encode that as a failure_case on the relationship
4. do not delete the relationship — a qualified relationship
   is more valuable than no relationship
5. add a note to journal.md

failed transfer is a result.
a system that learns only where analogies work,
without learning where they break,
may perform worse than no transfer at all.

---

## what to do when you are unsure about scope

stop. read scope.md.

if still unsure, write a note in journal.md under "open questions"
and move to something you are sure about.

do not make assumptions about scope in code.
scope decisions belong in documents, not buried in implementation.

---

## file structure conventions

```
/data/
  raw/           — ingested markdown, one file per source page
  curriculum/    — generated training examples by type
  experiments/   — experiment records by id

/eval/
  p5js/          — evaluation tasks and runner for p5.js
  three/         — (later)
  openscad/      — (later)

/graph/
  schema.md      — current graph schema
  migrations/    — schema change records

/scripts/
  ingest/        — firecrawl + tavily ingestion scripts
  graph/         — falkordb population and query scripts
  curriculum/    — training example generation scripts
  eval/          — evaluation runners

/journal.md      — research log (append only, do not rewrite history)
/agents.md       — original research spec (do not modify)
/explain.md      — what trainery is
/scope.md        — what is in and out of scope
/plan.md         — implementation plan
/agent.md        — this file
/tools.md        — tool reference and budget tracking
/graph.md        — graph schema reference
```

---

## the test for any change

before committing:

> does this change make the experiment more reproducible,
> more interpretable, or more measurable?

if no — reconsider.
if yes — proceed and document.

# plan.md

the implementation plan for trainery, p5.js first.

---

## guiding principle

build the loop before optimizing any part of it.

a working end-to-end pipeline that produces weak results is more
valuable than a perfect ingestion system with no evaluation.
get signal first. optimize what the signal says matters.

---

## phase 0 — foundations
*status: in progress*

set up the project structure, tooling, and connections.

- [ ] repository structure established
- [ ] falkordb cloud instance running (free tier)
- [ ] firecrawl api key connected
- [ ] browserbase developer plan connected
- [ ] tavily api key connected
- [ ] nebius token factory connected ($25 credits)
- [ ] backboard.io connected ($30 credits)
- [ ] gcp project set up (storage bucket for artifacts)
- [ ] graph schema designed (see graph.md)
- [ ] journal.md started

deliverable: all tools reachable, schema on paper, nothing built yet.

---

## phase 1 — ingestion
*target: p5.js + source domains*

ingest documentation and examples into clean structured form.
output is markdown files ready to become graph nodes and training data.

### step 1a — map before crawling

use firecrawl /map on each domain to inventory urls.
do not crawl blindly. review the map, select what matters.

target urls:
- https://p5js.org/reference/
- https://p5js.org/learn/
- https://p5js.org/examples/
- react.dev/reference (selected pages)
- vitejs.dev/guide (selected pages)
- docs.python.org (selected pages: functions, classes, iteration, collections)

estimated firecrawl credits: 3,000–5,000
budget remaining after: 5,000–7,000

### step 1b — scrape

targeted scrape of selected urls.
store as: domain / section / page.md
record source url, scrape date, and page title in frontmatter.

### step 1c — examples and counterexamples

use tavily to find:
- canonical p5.js examples by concept
- common p5.js beginner mistakes
- stack overflow threads on p5.js confusion points
- "p5.js vs processing" comparisons (good transfer contrast material)
- "p5.js for react developers" tutorials (pre-existing transfer framing)

these are especially valuable because they encode failure modes
that pure documentation does not.

### step 1d — provenance check

before moving to phase 2, verify:
- every ingested page has a source url
- every ingested page has a date
- no llm-generated summaries have been added as if they were source material
- storage structure is clear and consistent

deliverable: /data/raw/ directory with structured markdown,
one file per source page, provenance in frontmatter.

---

## phase 2 — concept graph
*the core of trainery*

build the falkordb graph from ingested material.

### step 2a — p5.js concept nodes

create nodes for every p5.js concept, construct, and api.
priority concepts for first pass:

**environment**
- setup() — runs once on start
- draw() — runs every frame
- frameRate(), frameCount
- width, height (canvas dimensions)
- createCanvas()

**2d primitives**
- ellipse(), circle(), rect(), line(), point(), triangle()
- beginShape(), endShape(), vertex()

**color**
- fill(), stroke(), noFill(), noStroke()
- background()
- colorMode() — rgb vs hsb (important: not obvious from js background)

**transforms**
- translate(), rotate(), scale()
- push(), pop() — the transform stack

**coordinate system**
- origin at top-left
- y increases downward
- this is a known failure mode for python/math-background developers

**interaction**
- mouseX, mouseY, mouseIsPressed
- keyIsPressed, key
- mousePressed(), keyPressed() (event functions)

**typography**
- text(), textSize(), textAlign()

**math**
- map() — IMPORTANT: name collision with python map(), different semantics
- random(), noise()
- dist(), lerp(), constrain()

**control flow**
- standard js (for, while, if) — direct transfer from js/python
- no special p5 control flow

### step 2b — source domain nodes

create nodes for react, vite, and python concepts
that are relevant to p5.js transfer.

do not ingest all of react. ingest what connects to p5.js targets.

react concepts relevant to p5.js:
- component lifecycle (maps to setup/draw)
- event handlers (maps to mousePressed etc.)
- state (partial map to draw loop variables)
- the dom as rendering target (contrasts with canvas)

python concepts relevant to p5.js:
- functions (direct transfer)
- loops (direct transfer)
- map() built-in (contrast — different from p5 map())
- coordinate systems from math (contrast — y-axis flipped in p5)

vite/js concepts relevant to p5.js:
- javascript syntax (direct transfer)
- browser environment (direct transfer)
- module imports (p5 as a library)

### step 2c — transfer relationships

encode relationships between source and target nodes.
every relationship must include:
- type (analogous_to / partial_analogy / contrasts_with / etc.)
- shared: what is in common
- different: what diverges
- failure_case: where the analogy breaks down
- evidence: why we believe this relationship exists
- confidence: low / medium / high

priority relationships for p5.js:

| source | relationship | target | notes |
|--------|-------------|--------|-------|
| react component lifecycle | partial_analogy | setup() + draw() | react re-renders on state change; p5 loops unconditionally |
| react event handlers | analogous_to | mousePressed(), keyPressed() | p5 uses global functions, not callbacks on elements |
| js for loop | analogous_to | p5 for loop | identical syntax, same semantics — strong transfer |
| python functions | analogous_to | p5 functions | same concept, minor syntax diff |
| python map() | contrasts_with | p5 map() | same name, completely different semantics — encode as failure mode |
| math y-axis (up) | contrasts_with | p5 y-axis (down) | common source of confusion — explicit failure mode |
| react dom rendering | contrasts_with | p5 canvas rendering | different model entirely |
| react state | partial_analogy | draw loop variables | p5 has no state system; variables in outer scope serve this role |
| push/pop (call stack) | partial_analogy | push() / pop() (transforms) | same name, different domain — encode carefully |

### step 2d — verify graph structure

before moving to curriculum generation:
- query the graph to check node counts and relationship counts
- manually inspect 10 random relationships for quality
- check that every node has a source provenance
- check that failure mode nodes exist and are connected

deliverable: falkordb graph with p5.js + source concepts and
transfer relationships, queryable and inspectable.

---

## phase 3 — curriculum generation

use the graph to generate training examples.

### three example types

**direct target examples**
p5.js code with explanation. tasks. corrections.
generated from ingested documentation and examples.

**transfer examples**
show source concept → target representation.
example:
> you know react component props.
> in p5.js, parameterized behavior is achieved through function arguments.
> here is what that looks like...

**contrastive examples**
explicitly show where analogies break.
example:
> in python, map(fn, iterable) applies a function to a sequence.
> in p5.js, map(value, start1, stop1, start2, stop2) remaps a number
> from one range to another. same name, completely different function.
> do not confuse these.

### curriculum ordering

use graph structure to determine ordering:
1. concepts with strong transfer from source (easiest — build confidence)
2. concepts with partial transfer (require careful contrast)
3. target-specific concepts with no good prior (require most training)
4. failure modes (explicitly teach what not to transfer)

### generation

use nebius token factory (llama 3.1 70b or similar) to generate
examples at scale. keep generation prompts versioned.
record: model used, prompt version, graph version, date.

deliverable: /data/curriculum/p5js/ with typed training examples,
provenance metadata, ready for training.

---

## phase 4 — evaluation benchmarks

build evaluation tasks before running any training.
do not design evaluations after seeing results.

### task categories for p5.js

**syntax** — does the code run without errors?
**static output** — does a static sketch produce the right shape/color?
**animation** — does the draw loop behave correctly over N frames?
**interaction** — does mouse/key input produce expected behavior?
**composition** — can the model combine multiple concepts?
**modification** — can the model change existing p5.js code correctly?
**debugging** — can the model identify and fix an error in p5.js code?
**unseen tasks** — tasks not present in training data

### evaluation loop (p5.js via browserbase)

```
prompt
  ↓
generated p5.js sketch
  ↓
browserbase session (playwright)
  ↓
inject sketch into html page with p5 cdn
  ↓
wait for setup() + N draw() frames
  ↓
screenshot canvas
  ↓
pixel-level check OR lm-as-judge on screenshot
  ↓
pass / fail + score
```

build this loop end-to-end with one test case before scaling.

deliverable: /eval/p5js/ with task definitions, expected outputs,
and a working browserbase evaluation runner.

---

## phase 5 — baselines

run the three comparison conditions.

**baseline a: target-only**
train on p5.js data only.
no source knowledge. no transfer structure.
this is the minimum bar.

**baseline b: source + target**
train on react + vite + python + p5.js data.
no transfer structure.
this tells us whether more data alone helps.

**experiment c: transfer-aware**
train with the transfer graph informing curriculum ordering
and contrastive examples.
this is the trainery intervention.

hold constant: model initialization, evaluation set, compute budget.
if anything changes between conditions, document it.

deliverable: three experiment records with full provenance,
results on the p5.js evaluation benchmark.

---

## phase 6 — analyze and iterate

- compare conditions on all evaluation metrics
- look especially at: examples required to reach proficiency,
  compute required, failure modes, transfer errors
- record cases where analogy helped, did nothing, or hurt
- update journal.md with findings
- decide what to test next (three.js? different transfer structure?)

---

## phase 7 — trainium (later)

once the loop works on cpu/small gpu, move to trainium.
run fixed-budget experiments: given the same compute,
does transfer-aware training produce a better p5.js agent?

this phase is explicitly deferred until phases 1–6 are solid.

---

## timeline heuristic

do not set hard deadlines on research phases.
set checkpoints instead:

- checkpoint 1: graph queryable with p5.js concepts
- checkpoint 2: one evaluation task running end-to-end via browserbase
- checkpoint 3: 100 curriculum examples generated with provenance
- checkpoint 4: baseline a running
- checkpoint 5: all three conditions run and compared

each checkpoint should produce a journal.md entry.

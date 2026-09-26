# scope.md

what is in scope, what is out, and why.

---

## in scope

### knowledge representation
- a concept graph in falkordb covering source and target domains
- nodes for: languages, frameworks, concepts, constructs, apis,
  operations, abstractions, examples, tasks, constraints, failure modes
- relationships with metadata: what is shared, what differs,
  confidence, evidence, source, failure cases
- vector embeddings alongside graph structure for hybrid retrieval

### ingestion
- source domain ingestion: react, vite, python
- target domain ingestion: p5.js (first), then three.js, openscad
- canonical sources only: official docs, official references,
  source repositories, primary technical material
- provenance recorded for every piece of knowledge entering the graph

### curriculum generation
- distinguishing known / transferable / target-specific / novel
- generating training examples: direct, transfer, contrastive
- contrastive examples are first-class: teaching where analogies fail
  is as important as teaching where they work

### training data
- generated datasets with full provenance records:
  source knowledge, target knowledge, transfer relationships,
  generation method, model, date, graph version
- three baseline conditions:
  a) target-only
  b) source + target (no transfer structure)
  c) transfer-aware (source + transfer graph + target)

### evaluation
- execution-based where possible
- for p5.js: code → browser (browserbase) → rendered canvas → measurement
- for three.js: code → browser → 3d scene → measurement
- for openscad: code → openscad binary → geometry → measurement
- lm-as-judge only as a fallback, not a primary signal

### experiment tracking
- every experiment records: id, date, model, domains, data versions,
  graph version, training config, evaluation version, results,
  failure modes, notes
- no silent regeneration of training sets
- no silent modification of evaluation benchmarks

---

## out of scope (for now)

### visual polish
the system does not need a beautiful ui to be scientifically valid.
a simple interface is fine. ui work does not take priority over
reproducibility, provenance, or evaluation integrity.

### benchmark chasing
optimizing for a leaderboard score is not the goal.
a higher score obtained by changing the evaluation is not progress.

### nki / trainium optimization
trainium is a later-phase backend.
nki kernel optimization is out of scope until profiling shows it matters.
do not make the core research pipeline dependent on trainium.

### cadquery
cadquery is a planned target but not part of the first experiment.
it will be added after p5.js results are in.

### multi-agent orchestration
the current scope is a single training loop, not a swarm of agents.
multi-agent patterns may emerge later but are not planned now.

### production deployment
trainery is research software. production hardening, slas, and
enterprise features are explicitly not a priority.

---

## constraints that must not be violated

1. do not silently change source/target domain assignments.
   if a domain moves, document why.

2. do not encode hard equivalences between concepts.
   use analogous_to, partial_analogy, contrasts_with — not same_as.

3. do not replace the graph with vector similarity.
   vectors are for retrieval. the graph is for explicit structure.

4. do not let llm-generated explanations become canonical facts.
   firecrawl and tavily are acquisition tools, not authorities.

5. do not overwrite an experiment's training set and call it
   the same experiment.

6. do not modify an evaluation benchmark to improve a result.

7. do not optimize the demo at the expense of the research.

---

## the current priority boundary

we are currently at step 1–5 of the implementation sequence:

1. research specification ← done (agents.md, this document)
2. source-domain ingestion ← starting
3. target-domain ingestion ← starting (p5.js first)
4. graph schema ← designing now
5. concept graph ← building next

steps 6–16 come after these foundations are solid.

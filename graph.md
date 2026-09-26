# graph.md

the falkordb graph schema for trainery.
current version: 0.1 (p5.js focus)

do not change this schema without creating a new version entry
at the bottom of this file.

---

## design principles

the graph represents **concepts and their relationships**,
not documentation chunks.

a node is not a paragraph. it is a concept, construct, api, or idea
that can be related to other concepts.

a relationship is not "these things are similar."
it is a structured assertion about how two concepts are connected,
what they share, how they differ, and where the connection breaks.

vectors are stored alongside the graph for semantic retrieval,
but vector similarity does not establish a graph relationship.
graph relationships are explicit, authored, and justified.

---

## node types

### Language
a programming language.

properties:
- name (string) — e.g. "javascript", "python"
- version (string, optional) — e.g. "es2022", "3.11"
- paradigm (string[]) — e.g. ["imperative", "functional", "oop"]
- source (string) — url of canonical reference
- ingested_at (string) — iso date

examples: JavaScript, Python

---

### Framework
a framework built on a language.

properties:
- name (string) — e.g. "react", "p5.js"
- version (string, optional)
- language (string) — parent language
- domain (string) — e.g. "ui", "creative-coding", "3d"
- source (string)
- ingested_at (string)

examples: React, p5.js, three.js

---

### Concept
an abstract programming idea that can appear across environments.

properties:
- name (string) — e.g. "iteration", "composition", "event handling"
- description (string) — one or two sentences
- domain (string) — "general" or a specific domain
- source (string) — where this definition comes from
- ingested_at (string)

examples: iteration, composition, parameterization, abstraction,
          coordinate-system, render-loop, event-handling

---

### Construct
a specific language or framework construct.
more concrete than Concept — has actual syntax.

properties:
- name (string) — e.g. "for loop", "setup()", "mousePressed()"
- syntax (string) — canonical syntax example
- description (string)
- framework (string) — which framework or language this belongs to
- source_url (string) — link to official docs page
- ingested_at (string)
- embedding (vector) — for semantic retrieval

examples: p5js.setup(), react.useEffect(), python.for_loop,
          p5js.push(), p5js.map()

---

### API
a specific callable function, method, or property.
more specific than Construct — includes signature and parameters.

properties:
- name (string) — e.g. "ellipse"
- full_name (string) — e.g. "p5js.ellipse"
- signature (string) — e.g. "ellipse(x, y, w, [h])"
- parameters (string) — description of parameters
- returns (string) — what it returns
- description (string)
- category (string) — e.g. "2d-shapes", "color", "transforms"
- framework (string)
- source_url (string)
- ingested_at (string)
- embedding (vector)

examples: p5js.ellipse, p5js.map, p5js.colorMode, react.useState

---

### FailureMode
a known pattern of incorrect transfer or common mistake.
these are first-class nodes, not footnotes.

properties:
- name (string) — short label
- description (string) — what the mistake is
- source_concept (string) — what prior knowledge causes the mistake
- target_concept (string) — what is incorrectly applied to
- example_wrong (string) — code or description of the error
- example_correct (string) — the right approach
- why_it_happens (string) — explanation of the confusion
- source (string) — where this failure mode was documented
- ingested_at (string)

examples:
- python_map_vs_p5_map (same name, different semantics)
- y_axis_inverted (math y-up vs canvas y-down)
- draw_loop_not_reactive (p5 loops unconditionally vs react re-renders on state)
- push_pop_transforms_vs_callstack (transform stack vs execution stack)

---

### Example
a concrete code example demonstrating a concept or construct.

properties:
- name (string)
- code (string) — the actual code
- language (string)
- framework (string)
- concepts_demonstrated (string[]) — list of concept names
- description (string)
- type (string) — "direct", "transfer", "contrastive", "debugging"
- source (string)
- ingested_at (string)

---

### Task
an evaluation or training task.

properties:
- name (string)
- prompt (string) — the task prompt
- framework (string) — target framework
- category (string) — "syntax", "static-output", "animation",
                       "interaction", "composition", "modification",
                       "debugging", "unseen"
- difficulty (string) — "easy", "medium", "hard"
- expected_behavior (string) — what correct output looks like
- evaluation_method (string) — "pixel-check", "execution", "lm-judge"
- created_at (string)

---

## relationship types

### ANALOGOUS_TO
strong conceptual correspondence between two constructs or concepts.
not identity. similarity with differences.

properties:
- shared (string) — what the two things have in common
- different (string) — how they diverge
- failure_case (string) — where the analogy breaks down
- confidence (string) — "low", "medium", "high"
- evidence (string) — why we believe this relationship
- direction (string) — "bidirectional" or "source→target"

example:
```
(react:component_lifecycle)-[ANALOGOUS_TO {
  shared: "initialization phase + recurring update phase",
  different: "react re-renders on state change; p5 loops unconditionally at frameRate",
  failure_case: "assuming draw() only runs when something changes",
  confidence: "high",
  evidence: "p5.js learn docs: draw loop section"
}]->(p5js:setup_draw_loop)
```

---

### PARTIAL_ANALOGY
weaker correspondence — some aspects map, others do not.
use this when ANALOGOUS_TO would overstate the relationship.

properties: same as ANALOGOUS_TO, plus:
- maps (string) — which specific aspects transfer
- does_not_map (string) — which aspects do not

---

### CONTRASTS_WITH
explicit contrast — these concepts are related but work differently.
especially important for failure modes.

properties:
- relationship (string) — how they are related (e.g. "same name")
- difference (string) — what is actually different
- confusion_risk (string) — "low", "medium", "high"
- failure_mode_ref (string) — name of FailureMode node if one exists

example:
```
(python:map_builtin)-[CONTRASTS_WITH {
  relationship: "same function name",
  difference: "python map() applies fn to iterable; p5 map() remaps a number between ranges",
  confusion_risk: "high",
  failure_mode_ref: "python_map_vs_p5_map"
}]->(p5js:map_fn)
```

---

### IMPLEMENTS
a construct implements a concept.

properties:
- how (string) — description of how the construct implements the concept
- notes (string, optional)

example:
```
(p5js:for_loop)-[IMPLEMENTS]->(concept:iteration)
(p5js:setup_draw)-[IMPLEMENTS]->(concept:render_loop)
```

---

### GENERALIZES / SPECIALIZES
hierarchical concept relationships.

GENERALIZES: this node is the more general case
SPECIALIZES: this node is the more specific case

example:
```
(concept:render_loop)-[GENERALIZES]->(p5js:setup_draw)
(p5js:ellipse)-[SPECIALIZES]->(concept:2d_primitive)
```

---

### DEPENDS_ON
this construct requires another to function.

properties:
- required (boolean) — is this a hard dependency?
- reason (string)

example:
```
(p5js:draw)-[DEPENDS_ON]->(p5js:setup)
(p5js:translate)-[DEPENDS_ON]->(p5js:push) — soft: push/pop recommended around transforms
```

---

### DEMONSTRATES
an example demonstrates a concept or construct.

```
(example:grid_of_circles)-[DEMONSTRATES]->(p5js:for_loop)
(example:grid_of_circles)-[DEMONSTRATES]->(p5js:ellipse)
```

---

### CAUSES
a concept or pattern causes a failure mode.

```
(python:map_builtin)-[CAUSES]->(failuremode:python_map_vs_p5_map)
(concept:y_axis_up)-[CAUSES]->(failuremode:y_axis_inverted)
```

---

## initial graph queries (useful for verification)

count all nodes:
```cypher
MATCH (n) RETURN labels(n), count(n)
```

find all transfer relationships from react to p5:
```cypher
MATCH (a:Construct {framework:'react'})-[r:ANALOGOUS_TO|PARTIAL_ANALOGY]->(b:Construct {framework:'p5js'})
RETURN a.name, type(r), b.name, r.shared, r.different
```

find all high-risk failure modes:
```cypher
MATCH (f:FailureMode)
WHERE f.confusion_risk = 'high' OR f.name IS NOT NULL
RETURN f.name, f.description, f.why_it_happens
```

find all p5.js constructs with no transfer relationship:
```cypher
MATCH (c:Construct {framework:'p5js'})
WHERE NOT (c)<-[:ANALOGOUS_TO|PARTIAL_ANALOGY]-()
AND NOT (c)<-[:CONTRASTS_WITH]-()
RETURN c.name, c.category
```
(these are the "genuinely novel" concepts — the ones that need the most training data)

---

## schema version history

### v0.1 — initial schema
date: 2026-09
scope: p5.js first pass
node types: Language, Framework, Concept, Construct, API,
            FailureMode, Example, Task
relationship types: ANALOGOUS_TO, PARTIAL_ANALOGY, CONTRASTS_WITH,
                    IMPLEMENTS, GENERALIZES, SPECIALIZES,
                    DEPENDS_ON, DEMONSTRATES, CAUSES
changes: initial design, no previous version

### adding a new version

when the schema changes:
1. document the change here under a new version heading
2. write a migration script in /graph/migrations/
3. update the version number at the top of this file
4. note which experiments used which schema version

# journal.md

research log for trainery.
append only. do not rewrite history.
entries are dated and sequential.

---

## how to write an entry

each entry should cover:
- what was attempted
- what worked
- what failed or surprised
- what changed (code, schema, understanding)
- what is next
- open questions

honest entries are more valuable than clean entries.
a note that says "this didn't work and we don't know why"
is more useful than silence.

---

## 2026-09 — project start

### context

trainery is starting with p5.js as the first target domain.
source domains: react, vite, python.

rationale for starting with p5.js:
- closest transfer case (javascript → javascript, new domain abstractions)
- browserbase developer plan enables real browser evaluation
- bounded target: p5.js is smaller and more constrained than three.js
- if transfer does not help even here, we find out cheaply and early

### stack confirmed

all tools accessible:

| tool | status | budget |
|------|--------|--------|
| firecrawl | connected | 10,000 credits |
| tavily | connected | standard tier |
| browserbase | connected | developer plan ($20/mo) |
| falkordb cloud | connected | free tier (100mb) |
| nebius token factory | connected | $25 credits |
| backboard.io | connected | $30 credits |
| gcp | connected | credits available |
| watsonx.ai | connected | 300k free tokens |
| ibm bob 2.0 | accessible | dev plan |

### documents created

- explain.md — what trainery is and why
- scope.md — in/out of scope, constraints
- plan.md — phase-by-phase implementation plan
- agent.md — instructions for ai agents
- tools.md — tool reference and budget tracking
- graph.md — falkordb schema v0.1
- journal.md — this file

### decisions made

1. starting with p5.js, not three.js or openscad.
   reason: closest transfer case, fastest to validate or invalidate.

2. graph schema v0.1 designed with 8 node types and 9 relationship types.
   key decision: FailureMode is a first-class node type.
   reason: encoding where transfer fails is as important as where it works.

3. firecrawl strategy: map before crawl.
   reason: avoid spending 5x credits on stealth mode pages discovered mid-crawl.

4. evaluation loop target: browserbase → canvas screenshot → pixel check.
   reason: agents.md explicitly requires execution-based evaluation.
   lm-as-judge is a fallback, not the primary method.

5. falkordb free tier: run locally via docker during dev,
   cloud instance only during active sessions.
   reason: free tier deletes instances after 7 days inactivity, no backups.

### open questions

- what is the right granularity for graph nodes?
  should "push() + pop() transform stack" be one node or two?
  current answer: one Construct node (push_pop_stack) + two API nodes.
  revisit after first graph population pass.

- how do we handle p5.js examples from openprocessing.org?
  they are community-created, not official.
  current answer: ingest separately, label provenance as "community",
  use for training data but not as source of truth for graph relationships.

- what is the minimum viable evaluation task set for a first baseline run?
  current thinking: 20 tasks across 5 categories (syntax, static-output,
  animation, composition, debugging). revisit when eval runner is working.

### what is next

1. run firecrawl /map on p5js.org — inventory all urls
2. select and scrape reference + learn + examples
3. begin populating graph with p5.js Construct and API nodes
4. encode the first 10 transfer relationships (priority list in plan.md)
5. build the browserbase evaluation loop with one test sketch

---

*entries below this line are added as work progresses*

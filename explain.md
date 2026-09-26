# explain.md

what trainery is and why it exists.

---

## the core idea

trainery is a research project that investigates one question:

> can an ai agent learn a new programming environment more efficiently
> if its training is informed by programming knowledge it already has?

the hypothesis is that programming knowledge transfers.

if an agent already understands functions, composition, iteration,
abstraction, and coordinate systems — it shouldn't have to learn
those concepts from zero every time it encounters a new environment.

the syntax may be new. the api may be new. the domain may be new.
but some of the underlying ideas are already there.

trainery tries to make that transfer explicit and measurable.

---

## what this is not

trainery is not:

- a rag system. rag retrieves information at inference time.
  trainery uses prior knowledge at training time.

- a prompt engineering project. we are not trying to get a model
  to answer one question better. we are trying to make a model
  learn a new environment more efficiently.

- a demo. a working demo with an invalid experimental comparison
  is not a successful contribution.

- a model. trainery produces evidence, not just a model.

---

## the transfer problem

the central unit is not the language. it is the concept.

a react component and an openscad module are not the same thing.
but both can function as reusable units of construction.

that relationship — what is shared, what differs, where the analogy
fails — is what trainery encodes in a knowledge graph and uses to
build training curricula.

encoding "react.component == openscad.module" is wrong.
encoding "react.component is analogous_to openscad.module,
shared: reusable composition, different: ui vs geometry" is useful.

---

## the research question

**primary:**
can structured transfer of existing programming concepts reduce
the data or compute required for target-domain proficiency?

**secondary:**
does learning one new environment create useful prior knowledge
for learning another?

**eventual:**
can an agent's programming knowledge become cumulative, where
each new environment learned makes the next one easier to acquire?

---

## current scope

source domains (what the agent already knows):
- react
- vite
- python

first target domain (what we are teaching first):
- p5.js

later target domains:
- three.js
- openscad
- cadquery (planned)

we start with p5.js because it is the closest transfer case.
the language is still javascript. the concepts are new but bounded.
if transfer doesn't help even here, that is critical signal.

---

## what success looks like

the project is successful if it produces evidence that answers
the research question — not merely if it produces a model,
a graph, a dashboard, or a demo.

the ideal outcome is something like:

> given equivalent compute, transfer-aware training required fewer
> target-domain examples to reach proficiency in p5.js.

but the opposite result is equally valuable if measured rigorously.
a failed transfer is a result.

# p5js.md

everything trainery needs to know about p5.js as a target domain.

---

## what p5.js is

p5.js is a javascript library for creative coding.
it is the javascript successor to processing.

it makes drawing, animation, and interaction in the browser
accessible through a simple, function-based api.

the mental model is:
- a canvas (the drawing surface)
- setup() — runs once, set up your canvas and initial state
- draw() — runs every frame (default 60fps), draw everything

p5.js is deliberately not a framework with components or reactivity.
it is imperative. you tell it what to draw, every frame.

---

## why p5.js is a good first target

1. **language continuity** — still javascript.
   syntax transfer from vite/js is direct.
   no new language syntax to learn.

2. **bounded scope** — smaller api than three.js or openscad.
   the full reference is manageable.

3. **browser-native** — evaluation via browserbase is natural.
   a canvas renders in a real browser; screenshot it and inspect.

4. **rich example ecosystem** — openprocessing.org has thousands
   of community sketches. excellent training data source.

5. **interesting transfer cases** — react, python, and general js
   all have partial analogies to p5.js concepts.
   the analogies are real but imperfect — which makes them
   scientifically interesting.

6. **meaningful failure modes** — the y-axis flip, the map() name
   collision, the "draw loop is not reactive" mistake.
   these are concrete, testable, and important to encode.

---

## the p5.js mental model (what makes it different)

### the draw loop

```javascript
function setup() {
  createCanvas(400, 400);  // runs once
}

function draw() {
  background(220);         // runs every frame
  ellipse(200, 200, 50);  // draw a circle in the center
}
```

**transfer note (react):**
react re-renders when state changes.
p5 calls draw() unconditionally at the frameRate.
there is no "update trigger."
if you stop wanting to redraw, you call noLoop().

**transfer note (game loops):**
if the agent knows game loops (requestAnimationFrame), this maps well.
if it only knows react/event-driven patterns, it needs explicit teaching.

---

### the coordinate system

```
(0,0) ─────────────────→ x increases right
  │
  │
  │
  ↓
y increases DOWN
```

**failure mode: y_axis_inverted**
- source of confusion: mathematics (y-axis points up)
- result: agent places things upside-down or mirrors geometry
- fix: explicitly teach and show coordinate system diagrams early

---

### color

```javascript
fill(255, 0, 0);      // rgb red
fill('#ff0000');       // hex
fill(0, 100, 50);     // hsb if colorMode(HSB) is set

colorMode(HSB, 360, 100, 100);  // switch to hsb
colorMode(RGB, 255);            // back to rgb (default)
```

**transfer note:** the concept of fill + stroke is similar to svg.
the colorMode switch is genuinely novel.

---

### transforms and the push/pop stack

```javascript
push();              // save current transform state
translate(100, 100); // move origin
rotate(PI / 4);      // rotate 45 degrees
rect(0, 0, 50, 50);  // draws relative to new origin
pop();               // restore transform state
```

**failure mode: push_pop_transforms_vs_callstack**
- "push" and "pop" in programming usually mean a call stack or array operation
- in p5.js, push() and pop() save and restore transform + style state
- agents may confuse these with stack operations
- always teach push/pop with a visual example showing what changes

---

### the map() function

```javascript
// p5.js map — remaps a number from one range to another
let x = map(mouseX, 0, width, 0, 255);
// if mouseX is 0, x = 0. if mouseX is width, x = 255.
```

**failure mode: python_map_vs_p5_map**
- python: map(function, iterable) — applies a function to each element
- p5.js: map(value, start1, stop1, start2, stop2) — range remapping
- same name. completely different semantics.
- this is one of the highest-priority failure modes to encode

---

### interaction

```javascript
// system variables (updated every frame)
mouseX, mouseY        // current mouse position
mouseIsPressed        // boolean
keyIsPressed          // boolean
key                   // last key pressed (string)
keyCode               // last key pressed (code)

// event functions (called automatically by p5)
function mousePressed() { ... }   // called once on click
function mouseMoved() { ... }     // called every frame mouse moves
function keyPressed() { ... }     // called once on key down
```

**transfer note (react):**
react uses onClick={handler} — attaching callbacks to elements.
p5.js uses globally-defined functions — mousePressed() exists in
the sketch and p5 calls it automatically.
the concept (event handling) transfers. the mechanism does not.

---

## canonical ingestion sources

### official (highest priority)

| source | url | what it contains | firecrawl strategy |
|--------|-----|------------------|--------------------|
| reference | https://p5js.org/reference/ | full api, every function | map then scrape all |
| learn | https://p5js.org/learn/ | concept tutorials | map then scrape all |
| examples | https://p5js.org/examples/ | categorized code examples | map then scrape all |

### community (secondary)

| source | how to get it | label |
|--------|---------------|-------|
| openprocessing.org | tavily search by category | community |
| p5.js discourse / forum | tavily search for common mistakes | community |
| "p5.js for beginners" tutorials | tavily search | community |
| stack overflow p5.js tag | tavily search for confusion points | community |

### specifically useful searches for failure modes

```
tavily: "p5.js map function python confusion"
tavily: "p5.js y axis direction coordinate system"
tavily: "p5.js draw loop vs react rendering"
tavily: "p5.js push pop what does it do"
tavily: "p5.js vs processing differences javascript"
tavily: "p5.js for react developers"
tavily: "p5.js common mistakes beginners"
```

---

## priority concept nodes (phase 2 graph population)

**tier 1 — teach first (clear transfer or critical foundation)**

| concept | transfer from | relationship |
|---------|--------------|--------------|
| setup() | react componentDidMount | partial_analogy |
| draw() | react render / raf loop | partial_analogy |
| createCanvas() | dom element creation | analogous_to |
| for loops (drawing grids) | python/js for | analogous_to |
| functions as reusable drawing units | python functions | analogous_to |
| mouseX, mouseY | dom mousemove event | analogous_to |

**tier 2 — teach with contrast (partial transfer, failure modes)**

| concept | transfer from | relationship |
|---------|--------------|--------------|
| push() / pop() | call stack / array push | contrasts_with |
| map() | python map() | contrasts_with |
| y-axis direction | math convention | contrasts_with |
| mousePressed() | react onClick | partial_analogy |
| draw loop always runs | react conditional re-render | contrasts_with |

**tier 3 — teach as novel (limited or no useful prior)**

| concept | why novel |
|---------|-----------|
| colorMode(HSB) | no prior equivalent |
| noise() | perlin noise — domain-specific |
| frameCount, frameRate() | not present in general js patterns |
| noLoop() / loop() / redraw() | unique to p5 draw loop control |
| beginShape() / endShape() / vertex() | domain-specific |
| image(), loadImage() | differs from html img handling |

---

## evaluation task examples

### syntax tasks (easy)
- "draw a red circle in the center of a 400x400 canvas"
- "draw 5 blue rectangles evenly spaced horizontally"
- "write a setup() that creates a 600x400 canvas with a white background"

### static output tasks (medium)
- "draw a grid of 10x10 circles, each circle's color based on its position"
- "draw a triangle pointing upward in the upper-left quadrant"
- "draw concentric circles from the center outward"

### animation tasks (medium)
- "make a circle follow the mouse"
- "make a rectangle move from left to right and wrap around"
- "make the background color change over time using frameCount"

### composition tasks (hard)
- "draw a clock face with hands that update each frame"
- "create a particle system where clicking adds a new particle"
- "draw a spiral using trigonometry and a loop"

### debugging tasks (hard)
- provide broken code (y-axis confusion, wrong map() usage,
  missing push/pop around transforms) and ask the model to fix it

---

## the browserbase test sketch

first thing to verify before any training run.
this is the baseline "does our evaluation loop work?" test.

```javascript
// test-sketch.js
// expected output: red circle, center of canvas

function setup() {
  createCanvas(400, 400);
  noLoop();
}

function draw() {
  background(255);
  fill(255, 0, 0);
  noStroke();
  ellipse(200, 200, 100, 100);
}
```

evaluation check:
- pixel at (200, 200) should be approximately rgb(255, 0, 0)
- pixel at (0, 0) should be approximately rgb(255, 255, 255)
- canvas should be 400x400

if this passes, the evaluation loop is working.
if this fails, fix the loop before doing anything else.

# Story Dojo — Beginner UX Audit & Proposals

_A design brief for the next implementation pass. Written to be handed to an
implementing model (Sonnet). No code here — decisions, specs, data shapes, and
priorities. Grounded in the current `index.html` component (state model,
lenses, `seedStory`, fields, focus sheet, export)._

---

## 0. TL;DR

Story Dojo is a **beautiful tool for people who already think like John Truby.**
For a first-timer who has never read *The Anatomy of Story*, it is closer to
being handed a cockpit than a canvas. The structure is the product — but the
structure is also the barrier. Nothing in the app teaches the vocabulary,
suggests an order, shows a worked example next to your own, or tells you that
"I don't know yet" is fine and you're still making progress.

The five problems, in priority order:

1. **Cold start / jargon wall** — a new work is 21 empty boxes labelled with
   terms a beginner can't define. (P0)
2. **No worked examples** — the concepts are abstract; famous stories would make
   them click instantly. (P0)
3. **No guided path** — everything is available at once; nothing says "start
   here, then this." (P1)
4. **No motivation loop** — no sense of progress, momentum, or permission to be
   imperfect; easy to feel lost and quit. (P1)
5. **Single-player only** — localStorage means no real collaboration. (P2)

Plus one experiment: a **Monologue Studio** that turns the character structure
into raw, punchy first-person voice (Section 6).

Everything below is written so each item can be picked up independently.

---

## 1. Current-state audit (what a beginner actually hits)

**The good (keep and lean on):**
- Opens on a filled seed story ("Escape the Inheritance") — a beginner can *see*
  a finished shape before touching anything. This is the single strongest
  onboarding asset and is currently underused.
- The **state grammar** (Question / Hypothesis / Decision) is genuinely
  beginner-friendly *in principle* — it legitimizes not knowing. It's just never
  explained.
- **Drafts / Memory** (nothing is overwritten) removes the fear of "ruining" work.
- **Lenses** are a great advanced feature.

**The friction, walking in cold:**

| # | Moment | What happens now | Why a beginner stalls |
|---|--------|------------------|-----------------------|
| 1 | First new work | `seedBlank` → 21 elements, all `OPEN`, all empty | No entry point; no "do this first" |
| 2 | Reads a node label | "Designing Principle", "Basic Action", "New Equilibrium" | Undefined jargon; no tooltip, no definition |
| 3 | Opens an element | Empty textarea + placeholder | No prompt scaffolding, no example, blank-page freeze |
| 4 | Doesn't know an answer | Leaves it blank | Blank ≠ progress; feels like failure (vs. marking it a Question) |
| 5 | Wants to know if they're "doing it right" | Only signal is `n decided · n open` in the Works drawer | No feedback, no encouragement, no milestone |
| 6 | Sees Plot/Scenes empty | "Nothing placed here yet." | Dead-end tone; no next action |
| 7 | Relationships / verbs | Powerful, but advanced | Overwhelming before the basics exist |

**Complexity verdict for a beginner:** the *conceptual* load is high (Truby's
full 7-step + premise chapter), the *interface* load is medium (map, drag,
lenses, relationships). The interface is fine; the **conceptual scaffolding is
missing**. Fix the teaching, not the tool.

**Time-to-first-story estimate today:** a Truby reader can seed a rough premise
in ~10 min. A cold beginner likely bounces in <5 min without ever completing one
element they feel good about.

---

## 2. P0 — Onboarding & the cold start

**Goal:** in the first 3 minutes, a beginner understands what the app is, writes
*one* real sentence about *their* story, and feels they've started.

### 2.1 First-run "Start" flow (new state: `ui.onboarding`)
When a **brand-new work** is created (or on very first app load with no saved
stories), route to a lightweight guided intro instead of the bare map:

- **Step 1 — One sentence.** "Forget structure. In one line, what's your story
  about? (A person who ___.)" Save it straight into `premise` as a `HYPOTHESIS`.
  This is the whole first win.
- **Step 2 — The three questions that matter most.** Ask only: *Who's it about?*
  (`bestCharacter`), *What do they want / what's in the way?* (`conflict`),
  *How might they be different by the end?* (`characterChange`). Everything else
  is deferred.
- **Step 3 — "The rest can wait."** Reveal the full map, with the 4 answered
  elements lit and the other 17 explicitly marked *optional for now*.

Make it skippable ("I know Truby → open the full map") and re-runnable from a
`?` menu. Keep it as an overlay flow reusing existing `setFieldText` /
`setStatus`; don't fork the data model.

### 2.2 Reframe the blank state
- Change `seedBlank` so new works are **not** 21 equal empty boxes. Introduce a
  notion of **Essential (5)** vs **Deepen (the rest)**:
  Essential = `premise`, `bestCharacter`, `conflict`, `basicAction`,
  `characterChange`. Visually de-emphasize the rest until the essentials have
  content (dim, smaller, or collapsed under a "Go deeper" reveal).
- Replace discouraging empty copy ("Nothing placed here yet") with an inviting
  next action ("When your character is clear, drop the first thing that happens
  to them here — one line is enough.").

### 2.3 A persistent "Start here" affordance
A small always-available **guide button** (`?` or "Guide") in the masthead that
opens: the definition panel (2 below), the onboarding flow, and a "what should I
do next?" suggestion (Section 4.2).

---

## 3. P0 — Teach the concepts with famous examples

This is the highest-leverage change. Each element should be able to answer, in
place: **"What is this?"**, **"Why does it matter?"**, and **"Show me this in a
story I already know."**

### 3.1 A concept dictionary (new static data, e.g. `this.concepts`)
For each field key, author: a one-line **plain-language definition** (no Truby
jargon in the definition itself), a **prompt** (already partially exists as
`cardPrompts` — reuse/extend), and **2–3 worked examples** from widely known
films/books. Surface it in the focus sheet as a collapsible "What is this? /
Examples" strip, and as a hover tooltip on the map label.

**Authoring rules for examples:**
- Use loglines / short factual descriptions only (1 sentence). These are not
  copyrightable and keep us safe. Never paste dialogue or long passages.
- Prefer stories with near-universal recognition, and vary era/genre so
  different beginners find an anchor.
- Where Truby uses a canonical example in the book, prefer it (continuity for
  readers who later pick up the book).

### 3.2 Starter example bank (seed content — verify before shipping)

> Implementer: treat these as a first draft to refine. Two examples per concept;
> add a third where helpful.

- **Premise** — the story in one sentence, as a person + situation.
  - *The Godfather*: The reluctant youngest son of a mafia family becomes the
    ruthless don he swore he'd never be.
  - *The Lion King*: A young prince flees his kingdom after his father's death,
    then returns to claim who he really is.
- **Designing Principle** — the single deep process/metaphor that organizes the
  whole telling (Truby's term).
  - *Harry Potter*: A boy becomes a man as he journeys through seven years of
    magical school.
  - *Moby-Dick*: A man goes on a hunt for a whale that is really a hunt for the
    meaning of life.
- **Best Character / Protagonist** — whose need and change carry the story.
  - *Breaking Bad*: A dying chemistry teacher who wants to provide for his
    family — and discovers he likes power.
  - *Toy Story*: A favorite toy whose place is threatened by a shinier newcomer.
- **Conflict** — the fundamental clash of values, not just the plot obstacle.
  - *Pride and Prejudice*: Pride and social judgment vs. honest feeling.
  - *The Dark Knight*: Order and rules vs. chaos and fear.
- **Basic Action** — the one thing the hero does, over and over, to get what
  they want.
  - *Finding Nemo*: A father crosses an ocean, refusing to give up on his son.
  - *The Shawshank Redemption*: A man quietly, patiently refuses to be broken.
- **Character Change (W × A = C)** — weakness, put through action, becomes a new
  person.
  - *Groundhog Day*: A cynic (W) relives one day helping others (A) and becomes
    someone capable of love (C).
  - *A Christmas Carol*: A miser confronts his life and becomes generous.
- **Moral Choice** — the final decision that proves who they've become, with a
  real cost.
  - *Casablanca*: He gives up the woman he loves so she can do greater good.
  - *The Hunger Games*: She threatens a double suicide rather than kill her ally.
- **Desire** — the concrete, external thing pursued.
  - *The Wizard of Oz*: To get home to Kansas.
  - *Raiders of the Lost Ark*: To find the Ark before the Nazis.
- **Opponent** — who wants the same goal and forces the hero to grow (not just a
  villain).
  - *Amadeus*: Salieri, whose envy of Mozart drives the story.
  - *Whiplash*: The teacher who both breaks and forges the student.
- **Self-Revelation** — what the hero finally learns about themselves.
  - *Star Wars*: Luke learns to trust himself and the Force over the machine.
  - *Frozen*: Love, not fear, is what controls her power.

(Author the remaining fields — Audience Appeal, Wish List, Possibilities,
Challenges, Moral Argument, Story World, Weakness, Need, Plan, Battle, New
Equilibrium — the same way.)

### 3.3 "Compare to a master" mode
Let a beginner temporarily overlay one full worked example story (e.g. a bundled
*Finding Nemo* or *The Lion King* structure, stored like a normal `story`
object) **beside** their own field, so they can see a finished answer next to
their blank. Reuse the seed mechanism: ship 1–2 fully-filled example stories as
read-only "reference works" in the Works drawer (flagged `reference: true`,
non-editable, openable in the new View mode).

---

## 4. P1 — A guided creation path (reduce "everything at once")

The map is the *expert* surface. Add a **Guided lane** that runs the same data
through a linear, one-question-at-a-time flow.

### 4.1 "Guided" as a first lens (new `LENSES` entry, `kind: 'guided'`)
- Presents Essential elements in a sensible order (premise → character →
  desire → conflict → opponent → basic action → change → moral choice), one
  card at a time, each with: the plain definition, the prompt, an example, the
  textarea, and the Question/Hypothesis/Decision control.
- "Next" moves forward; "I don't know yet" marks it a Question and advances
  (explicitly framed as legitimate progress, not skipping).
- Writes to the exact same `fields` — Guided and Map are two views of one truth.
- At the end: a short **synthesis card** that stitches their answers into a
  paragraph (like the existing MD export's premise synthesis) so they *see a
  story emerge from their own words*. This is the payoff moment.

### 4.2 "What should I do next?" suggestion
A tiny rules engine (no ML): look at `fields`, find the highest-priority element
that is still `OPEN`/empty following the Essential order, and surface it as a
one-click suggestion in the Guide menu and the empty states. E.g. "Your
character is clear — next, what do they *want*?"

---

## 5. P1 — Motivation & retention (so beginners don't quit)

Beginners quit when they feel lost, judged, or like they've made no progress.
Counter each.

### 5.1 Make progress visible and kind
- A **progress meter** for the Essential 5 (and a secondary one for the full
  set): "Your story's spine: 3 of 5." Show it in the masthead in Guided mode.
- Celebrate the **first Decision**, the **Essential 5 complete**, and the
  **first Plot beat** with a light, non-cheesy moment (a quiet toast, not
  confetti spam). Tone should match the app's calm, literary voice.

### 5.2 Reframe uncertainty as momentum
- Count Questions as *engagement*, not gaps: "6 things written, 4 still open
  questions — that's a healthy early draft." Never show a bare "17 open" that
  reads as debt.
- Micro-encouragement copy tied to milestones, in the app's existing voice
  ("You've got a spine. The rest is play.").

### 5.3 Lower the cost of returning
- On reopen, land the user on **"Continue where you left off"** — the last-edited
  element or the next suggested one — instead of a static map they have to
  re-orient to.
- Optional: a gentle streak/"you've shaped this story N days" counter. Keep it
  ambient; never punitive.

### 5.4 Session-sized goals
- Offer a "10-minute session" mode: "Answer just 3 questions today." Small,
  finishable units beat an infinite canvas for retention.

---

## 6. Experiment — Monologue Studio (character as backdrop → punchy voice)

_Early-stage, may not fit. Framed as a separate module that reads the finished
structure. The bet: the character work the app already captures is exactly the
raw material a strong monologue needs._

### 6.1 Why the structure is the fuel
A monologue/soliloquy lands when a character, alone or cornered, collides their
**Weakness/Need** with their **Desire**, presses on a **Conflict**, and either
resists or reaches a **Self-Revelation**. Story Dojo already holds all of these
per character. A monologue is that gap, spoken aloud.

### 6.2 A working framework (the "PIVOT" method)
Give the writer a scaffold, not a fill-in-the-blanks template:

- **P — Pressure**: pick the moment/relationship squeezing the character (draw
  from `challenges`, `conflict`, `char.opponent`, a Plot beat).
- **I — I want**: state the raw desire in the character's own words
  (from `char.desire` / `char.need`).
- **V — Verboten**: the forbidden truth they'd normally never say
  (often the `char.weakness` or the inherited-morality tension in `conflict`).
- **O — One turn**: the monologue must *pivot* once — a realization, a decision,
  a reversal (the seed of `char.selfRevelation` or `moralChoice`).
- **T — The hard line**: end on a short, concrete, unsoftened final beat.

### 6.3 Craft rules for raw, punchy voice (surface as coaching)
- Short sentences. Then one long one when it earns it.
- Concrete nouns and active verbs; cut adjectives and adverbs.
- One image, reused — not five metaphors.
- Contradiction on the page (wants X, says Y) creates subtext.
- Escalate: each beat costs more than the last.
- Speak *at* someone (even if absent) — direct address beats abstraction.
- Present tense, first person, no stage directions.
- Read it aloud; if you can't breathe it, cut it.

### 6.4 Structural exemplars (describe the shape, never reproduce text)
- **Deliberation** — a mind arguing with itself (cf. Hamlet's "To be or not to
  be"): thesis → counter → unresolved dread.
- **Escalating rant** — a grievance building to a demand (cf. *Network*, "mad as
  hell"): naming the wound → widening it → the call to act.
- **Confession/reckoning** — quiet admission that turns (cf. many final-act
  soliloquies): small truth → bigger truth → the line they can't take back.

Ship these as *structural diagrams*, not quotations.

### 6.5 Product shape (minimal first cut)
- A **Monologue Studio** surface (new lens `kind: 'monologue'`, or a section in
  Scenes) where the writer: picks a **character**, picks a **mode** (Deliberation
  / Rant / Confession / Plea), picks a **pressure** (auto-suggested from their
  conflict/opponent/challenges), and gets the PIVOT scaffold pre-filled with
  their own structure data as prompts.
- Drafts are stored like everything else (versions kept — reuse `mk`/`commitVersion`
  so monologue drafts get the same Memory treatment).
- Keep it **assistive, not generative-by-default**: the value is the frame +
  their voice. If AI drafting is added later, feed it the character fields + the
  chosen mode + the craft rules as the system prompt, and always keep the human
  draft alongside.
- Position it in the flow as a **post-structure** activity ("Once the character
  holds, hear them speak") — matches the user's instinct that it comes "after
  story."

---

## 7. Collaboration — multiple people on one story

**Reality check:** persistence is `localStorage` (single browser, single user).
Collaboration needs the backend already on the roadmap (Cloudflare Worker +
storage). Sequence it so value ships before real-time complexity.

### 7.1 The data model is already collaboration-friendly
- Every element carries **versions + timestamps + status**, and notes have
  **types** (note/question/ai). This is most of what async collaboration and
  review need. Lean on it.

### 7.2 Phased plan
- **Phase A — Async share (small).** Extend the new read-only **View mode** into
  a real **share link**: export the story to the backend, get a URL, others open
  it in View mode. (Today Export→JSON already enables manual hand-off; this just
  removes the copy-paste.)
- **Phase B — Comments/review (medium).** Let viewers leave **margin notes**
  (the note system already exists) attributed to a name/identity. Add an "ai/
  external suggestion" style thread per element. Owner can **Settle** (the ripple
  Settle pattern already exists) or accept a suggested draft into versions.
- **Phase C — Multiplayer edit (large).** Real-time co-editing. Given per-element
  versioning, a pragmatic model is **element-level ownership + last-write-wins
  with a conflict draft** (a conflicting save becomes a new version to reconcile,
  not a lost edit). If true concurrent typing is needed, use a CRDT (Yjs) per
  textarea. Add lightweight **presence** (who's viewing which element).
- **Roles:** Owner / Editor / Commenter / Viewer. View mode = Viewer today.

### 7.3 Recommendation
Ship **A + B** first (share link + attributed comments). It delivers 80% of the
felt collaboration ("my writing partner / mentor / editor can see this and
respond") at a fraction of the cost of real-time, and it fits the app's
reflective, non-frantic character.

---

## 8. Suggested build order (for the next passes)

| Priority | Item | Rough size | Section |
|----------|------|-----------|---------|
| P0 | Concept dictionary + examples in focus sheet & tooltips | M | 3.1–3.2 |
| P0 | First-run onboarding flow (one sentence → 3 questions) | M | 2.1 |
| P0 | Essential vs Deepen framing + kinder empty states | S | 2.2 |
| P1 | Guided lens (linear Q&A over same data) + synthesis card | L | 4.1 |
| P1 | "What next?" suggestion engine | S | 4.2 |
| P1 | Progress meter + milestone toasts + continue-where-left-off | M | 5 |
| P1 | Bundled reference stories (read-only, "compare to a master") | M | 3.3 |
| P2 | Monologue Studio (assistive, PIVOT scaffold) | L | 6 |
| P2 | Share link (View mode → URL) | M | 7.2A |
| P2 | Attributed comments / review | M | 7.2B |
| P3 | Real-time multiplayer | XL | 7.2C |

**Guiding principle for the implementer:** add *teaching and guidance layers on
top of* the existing data model and lenses. Do **not** replace the map or fork
the state — Guided mode, onboarding, the dictionary, and View mode are all
alternate renderings of the same `fields`. The tool is good; it just needs to
explain itself and cheer the beginner on.

---

## 9. Notes on what already shipped (context for the implementer)

- **View mode** exists: a read-only, shareable **flashcard grid** of all Chapter
  One + Character elements (`ui.viewMode`, `deckSections` in `renderVals`,
  masthead "View ▸" / "‹ Edit"). This is the natural substrate for the share
  link (7.2A) and "compare to a master" (3.3).
- **Dark mode** exists app-wide via CSS variables (`:root` light +
  `:root[data-theme="dark"]` blue-gray), toggled from the masthead and persisted
  (`storyDojo_theme`). SVG map colors come from `pal()`. New UI should use the
  `var(--*)` tokens, never hardcoded hex, so both themes keep working.
- `cardPrompts` (per-field study prompts) is already authored and is a ready
  starting point for the dictionary prompts in 3.1.

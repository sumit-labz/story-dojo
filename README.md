# Story Dojo

A workspace for the thinking that happens *before* you can write a scene —
figuring out what your story is actually about, who your character is, and
what forces them to change.

It's built around the idea that a story premise isn't one sentence you nail
down and move past. It's a dozen interlocking pieces (premise, designing
principle, conflict, moral choice, character weakness, desire, opponent,
plan, self-revelation...) that keep revising each other as you understand
the story better. Story Dojo is a place to hold all of that at once, watch
how the pieces pull on each other, and keep every earlier draft instead of
overwriting your thinking.

No account, no cloud, no subscription — it runs entirely in your browser and
saves to your own machine.

## What it's for

If you've worked with story structure books — Truby's *22 Steps*, McKee,
Save the Cat, or your own homebrew method — you know the frustration of
doing this thinking in a document or a stack of index cards: you change the
premise, and now three other things are quietly out of date and you have to
remember to go check them by hand. Story Dojo tracks that for you.

Concretely, it gives you:

- **A structure to fill in, not a blank page.** Premise, Designing Principle,
  Best Character, Conflict, Basic Action, Character Change, Moral Choice,
  Audience Appeal, plus the character's Weakness, Need, Desire, Opponent,
  Plan, Battle, Self-Revelation, and New Equilibrium — the pieces that make a
  story hang together, laid out as elements you can open, edit, and connect.
- **Room to not know yet.** Every element carries a state — Question,
  Hypothesis, or Decision — so "I have no idea who the opponent is yet" is a
  real, visible state, not a blank you feel bad about.
- **A memory of your own thinking.** Editing an element doesn't erase what
  it used to say. Every version is kept, so you can see how your
  understanding of a character moved, and go back to an earlier idea if the
  new one isn't actually better.
- **Ripple warnings.** Change something causal — your Designing Principle,
  say — and everything downstream of it gets flagged for a second look, so
  you don't publish a plot built on a premise you've since abandoned.
- **Different lenses on the same material.** Story, Character, Moral, and
  Causal views re-arrange the same elements around a different question —
  "what's the architecture," "who's changing and why," "what am I arguing,"
  "what causes what" — instead of forcing you into one fixed diagram.
- **A place to move past outlining.** Once the structure feels solid, Plot
  and Scenes give you a running list to place beats and scenes against it.
- **An export you can actually use.** Pull the whole thing out as Markdown
  (to read, print, or paste into a manuscript doc) or JSON (to feed into
  another tool, including an AI writing assistant, with real structure
  intact) whenever you're ready to move on.
- **More than one story.** Keep several works in progress side by side,
  duplicate one to try a different direction, archive the ones you've set
  aside.

## Using it

1. **Start in the Story lens.** It opens on a seed example ("Escape the
   Inheritance") so you can see the shape of a filled-in structure before
   you touch anything — click any element to read it, or start a new work
   from the Works drawer (top left) for a blank slate.
2. **Click any element to open it.** Write freely in the text box. Mark it
   Question, Hypothesis, or Decision depending on how sure you are — that's
   not a formality, it changes how the element reads on the map (a decision
   looks solid; a hypothesis looks lighter; a question reads as unresolved).
3. **Drag elements to arrange them your way.** The layout isn't fixed —
   pull things closer together, spread them out, whatever matches how you
   think about the story. It stays where you put it.
4. **Draw relationships.** From inside an element, pick another element and
   a verb — *causes*, *contradicts*, *requires*, *transforms*, and others —
   and connect them. This is where the tool starts doing work for you:
   causal relationships trigger the ripple warning on the element downstream
   whenever the upstream one changes.
5. **Leave notes as you go.** Pin a plain note, a standing question, or an
   idea from an outside reader/editor/AI directly on the element it's about,
   instead of losing it in a separate document.
6. **Switch lenses when you're stuck.** If the Story view isn't unlocking
   anything, jump to Character or Moral — same material, different
   question, often enough to loosen a knot.
7. **Move to Plot or Scenes once the structure holds.** Place provisional
   beats; nothing here is final, it's just the next layer on top of a
   structure you now trust.
8. **Check Memory** to see the whole evolution of every element at once —
   useful when you want to remember *why* you moved away from an earlier
   idea, not just that you did.
9. **Export whenever you want a copy** — Markdown for reading or printing,
   JSON if you're handing the structure to another tool.

Everything saves automatically to your browser's local storage as you type
— there's no save button, and nothing leaves your machine unless you export
it yourself.

## Running it

Requires [Node.js](https://nodejs.org/) (no other dependencies — nothing to
`npm install`).

```bash
npm start
```

Then open http://localhost:8080. (You can also just open `index.html`
directly in a browser — it's a single static page and works the same way.)

## How it's built

- `index.html` — the app: markup + client-side logic, using a small template
  runtime (`support.js`) that compiles `{{ }}` bindings and `sc-if`/`sc-for`
  control flow into React at load time.
- `support.js` — the template runtime. Loads React/ReactDOM from a CDN and
  boots the app on page load. Do not hand-edit; generated by the design tool
  this prototype came from.
- `server.js` — a zero-dependency static file server for local self-hosting.
- `design/` — reference material from the design phase (not used by the app
  at runtime): the visual design rationale, the design-logic proposal page,
  and the seed story content baked into `index.html`.

## Roadmap

This is stage 1: get the working prototype running as a self-hosted app.
Future iterations will move persistence off `localStorage`, add multi-device
sync, and deploy to Cloudflare (Pages for the static app, with a Worker +
storage once sync is needed).

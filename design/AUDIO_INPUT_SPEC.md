# Audio Input — Implementation Spec

**For:** the model implementing this feature in `index.html`.
**Feature:** Let authors *speak* instead of type, at two scopes — one field, or one whole chapter/arc — using their own OpenRouter key. Nothing auto-writes; everything lands as a new draft version through a review step.

---

## 0. Read this first — how this app is built

Do not fight the existing architecture. Match it exactly.

- **Single static page.** All app code lives in `index.html`. There is **no build step** and **no backend**. `server.js` is a dumb static file server. Do not add a bundler, npm deps, or server routes.
- **Custom template runtime.** `support.js` compiles a `{{ }}` / `sc-if` / `sc-for` template into React at load time. **Do not hand-edit `support.js`.** New UI is authored as markup inside the `<script type="text/x-dc" data-dc-script>` block in `index.html` (starts ~line 571), using the same `{{ binding }}` and `onInput="{{ handler }}"` conventions already used everywhere in that block.
- **Where values come from.** `renderVals()` (Component method, ~line 1108) returns one big object; every `{{ name }}` in the template resolves against it. Handlers are arrow functions bound to `this`. To add UI you: (1) add markup in the template block, (2) add the values/handlers it needs into the object returned by `renderVals()`, (3) add supporting methods on the `Component` class.
- **State + persistence.** `this.state.stories` (array) and `this.state.currentId`. `this.cur()` returns the active story. UI-only state lives in `this.state.ui`; patch it with `this.setUI({...})`. Persist story data with `this.commit()` (writes `localStorage['storyDojo_v2']`). `freshUI()` (~line 876) defines the default UI shape — add any new UI fields there.
- **Reference/worked-example stories are read-only.** Every mutator guards with `if (this.isRefLocked()) return;`. All new mutators must do the same.

### The field model (critical — this is what you write into)

Each element is keyed in `this.cur().fields[key]`. Keys come from two ordered lists on the constructor:

- `this.metaP` — **Premise Chapter**, 13 fields:
  `premise, designingPrinciple, bestCharacter, conflict, basicAction, characterChange, moralChoice, audienceAppeal, wishList, possibilities, challenges, moralArgument, storyWorld`
- `this.metaC` — **Character Structure / the arc** (Truby's steps), 8 fields:
  `char.weakness, char.need, char.desire, char.opponent, char.plan, char.battle, char.selfRevelation, char.newEquilibrium`

`this.labels[key]` gives the human label. `this.concepts[key].def` gives the plain-language definition of what that field *is* (use these in the extraction prompt — they're the field descriptions).

A field object (see `mk()`, ~line 752) looks like:

```js
{ status, versions: [{ id, text, createdAt }], current, updatedAt, discoveredAt, review, notes: [] }
```

Two existing mutators are the ONLY way you should write text — do not touch `.versions` directly:

- `setFieldText(key, text)` — overwrites the *current* version's text.
- `commitVersion(key)` — snapshots current text into a new version and advances `current`.

**The rule for this feature:** a dictation/extraction result must never silently overwrite existing work. When a target field already has non-empty text, first `commitVersion(key)` (to preserve the old draft), then `setFieldText(key, newText)`. When the field is empty, just `setFieldText`. Encapsulate this as one helper `applyFieldText(key, text)` and use it everywhere.

---

## 1. Scope of what to build

Two entry points, one shared review UI, one shared OpenRouter client.

### Scope A — Dictate one field ("one of the 22 steps")
A 🎤 button on the open element editor (the focus panel — the textarea at ~line 382, `f.text` / `f.onText`) and on the Guided single-field view (~line 200). Record → transcribe → **no LLM extraction** → open the review UI pre-targeted to this one field.

### Scope B — Dictate a whole chapter/arc in one go
A "🎤 Dictate this chapter" affordance at the group level. Two groups only:
- **Premise Chapter** → fans out across `metaP` (13 fields)
- **Character Structure / arc** → fans out across `metaC` (8 fields)

Record → transcribe → **one extraction call** that sorts the transcript into that group's fields → open the review UI listing every proposed field fill.

Good places to put the Scope-B trigger: the Guided lens header, and/or a small control near each lens. Pick the least intrusive spot that's discoverable; a single button in the top toolbar with a "which chapter?" choice is acceptable. Do not add it to the map canvas nodes.

### Shared: Review UI (the heart of the feature)
A modal/panel (model it on the existing export modal, ~line 545+, and its `showExport` pattern). It shows one row per proposed field:

- Field label (`this.labels[key]`) + its definition (`this.concepts[key].def`) as helper text.
- The proposed text, in an **editable** textarea (the user can fix the LLM/transcript before accepting).
- If the field already has text, show the current text collapsed above it labeled "current draft — will be kept as a version," so the user knows nothing is lost.
- Per-row **Accept / Skip** toggle (default Accept for non-empty proposals; default Skip for empty ones).
- Footer: "Apply N fields" (runs `applyFieldText` for each accepted row) and "Cancel."

For Scope A there is exactly one row. Reuse the same component.

---

## 2. OpenRouter integration

### 2.1 Key handling & consent (respect the privacy promise)
The README's headline promise is "nothing leaves your machine." Audio breaks that, so it must be **explicit and opt-in**:

- Store the user's own OpenRouter API key in `localStorage['storyDojo_openrouter_key']`. Never ship a key in the app.
- Store the chosen models in `localStorage` too (see 2.4).
- First time the user triggers any audio action with no key set, show a one-time consent + setup panel: plain-language notice that "recording and transcript will be sent to OpenRouter using your own API key; audio is not stored by Story Dojo," a link to https://openrouter.ai/keys, a key input, and the model dropdowns.
- Add a small **Settings** entry point (gear near the toolbar) to change key/models later and to clear the key.
- Wrap the key read in `try/catch` like the existing `localStorage` reads (see `freshUI`, `loadState`).

### 2.2 Recording
Use the browser `MediaRecorder` API — no library.

```js
// getUserMedia({audio:true}) -> MediaRecorder -> collect chunks ->
// on stop, Blob(type:'audio/webm'). Show elapsed time + a Stop button while recording.
```

Handle: permission denied, no mic, and a hard cap (e.g. warn approaching the 25 MB / long-recording limit). Free the stream tracks on stop.

### 2.3 Transcription call
OpenRouter has a dedicated OpenAI-compatible endpoint:

```
POST https://openrouter.ai/api/v1/audio/transcriptions
Authorization: Bearer <user key>
```

Send the audio as multipart/form-data (`file` + `model`) **or** JSON with base64 `input_audio: { data, format }`. Default transcription model: `openai/whisper-1` (user-overridable). Returns `{ text: "..." }`. Supported input up to ~25 MB; formats include webm/wav/mp3/flac.

### 2.4 Extraction call (Scope B only) — model is user-chosen
The user picked **"let user pick"** for the extraction model. So:

- Add an **extraction-model dropdown** in Settings, stored in `localStorage['storyDojo_extract_model']`. No hard-coded default is forced on them — but pre-populate the dropdown with a sensible short list and a free-text "custom slug" option. Suggested list:
  - `google/gemini-flash-1.5` (cheap/fast — good default highlight)
  - `anthropic/claude-3.5-sonnet` (stronger reasoning)
  - `openai/gpt-4o-mini`
  - custom (free text OpenRouter slug)
- If nothing is chosen yet, prompt them to choose on first Scope-B use rather than silently picking.

Extraction request → `POST https://openrouter.ai/api/v1/chat/completions` (OpenAI-compatible), `response_format: { type: 'json_object' }` when the model supports it.

**Prompt construction (build dynamically from the code, do not hardcode field lists):**
- System: the app is a story-structure tool; the author has spoken freely about one chapter of their story; extract their thinking into the specific fields; only fill a field if the transcript genuinely supports it; leave others empty; do not invent; preserve the author's own words where possible; return JSON.
- Provide the target schema by iterating the group's keys and emitting, for each: the key, `this.labels[key]`, and `this.concepts[key].def` (the field's meaning). This is the same set of fields `genJSON()` already knows about, so the shape is consistent with export.
- Ask for `{ "<key>": "<text or empty string>", ... }` using the exact keys (including the `char.` prefix for `metaC`).
- User content: the transcript.

Parse defensively (models sometimes wrap JSON in prose): extract the first `{...}` block, `JSON.parse` in a `try/catch`, and on failure fall back to showing the raw transcript in a single "couldn't auto-sort — here's your transcript, paste it where it belongs" review row so the recording is never lost.

### 2.5 Network resilience
All calls: loading state (the review UI opens in a "transcribing…/sorting…" state), timeout, and a visible error with the raw transcript preserved for manual use. Never lose a transcript because a later step failed.

---

## 3. UI state to add

Add to `freshUI()` (all UI-only, not persisted with story data):

```js
audio: {
  open: false,          // review panel visible
  phase: 'idle',        // 'recording' | 'transcribing' | 'extracting' | 'review' | 'error'
  scope: null,          // 'field' | 'chapter'
  targetKey: null,      // for scope 'field'
  group: null,          // 'premise' | 'character' for scope 'chapter'
  seconds: 0,
  transcript: '',
  rows: [],             // [{ key, label, def, currentText, proposed, accept }]
  error: null,
},
needsAudioSetup: false, // show consent/key panel
```

Persist key/models in their own `localStorage` items (§2.1/2.4), **not** inside `storyDojo_v2`.

---

## 4. Methods to add on `Component`

Sketch (names indicative):

- `hasOpenRouterKey()`, `getOpenRouterKey()`, `setOpenRouterKey(k)`, `getExtractModel()`, `getTranscribeModel()`.
- `applyFieldText(key, text)` — the commit-then-set helper from §0. Guards `isRefLocked()`.
- `startDictation(scope, arg)` — entry point; if no key → set `needsAudioSetup`; else begin recording. `arg` is a field key (scope 'field') or group id (scope 'chapter').
- `beginRecording()` / `stopRecording()` — MediaRecorder lifecycle + `seconds` ticker.
- `transcribe(blob)` → returns text (§2.3).
- `extractChapter(transcript, group)` → returns `{key:text}` (§2.4); builds the schema from `metaP`/`metaC` + `labels` + `concepts`.
- `buildReviewRows(...)` — from a single field (scope A) or the extraction map (scope B). Skip empty proposals or include them as default-Skip.
- `toggleRow(key)`, `editRow(key, text)`, `applyReview()` (loop accepted rows → `applyFieldText`), `cancelAudio()`.
- Reuse `this.showToast(...)` for success ("Chapter drafted — 6 fields updated") — the toast mechanism already exists.

Wire every new value/handler through `renderVals()`.

---

## 5. Interaction flow (summary)

**Scope A (field):** open element → 🎤 → record → Stop → transcribing → review (1 row, editable) → Apply → `applyFieldText(targetKey, text)` → toast. Old text (if any) preserved as a prior version.

**Scope B (chapter):** trigger with chapter choice → record (talk through the whole chapter/arc) → Stop → transcribing → extracting → review (N rows, each editable, Accept/Skip) → Apply → each accepted field `applyFieldText` (commit-then-set) → toast "N fields updated." Fields the transcript didn't cover stay untouched.

Both: any failure leaves the transcript visible and copyable.

---

## 6. Guardrails / definition of done

- [ ] No new runtime dependency; still runs by opening `index.html` or `npm start`.
- [ ] `support.js` untouched. New UI lives in the x-dc template + `renderVals()` + class methods, matching existing style.
- [ ] No key shipped in code. Audio is opt-in behind a clear consent panel. Key/models in `localStorage`, `try/catch`-wrapped.
- [ ] Nothing auto-writes. Every applied change routes through the review panel and through `applyFieldText` (commit-then-set), so **no existing draft is ever overwritten** — prior text becomes a kept version. Verify in the Memory lens that old versions survive.
- [ ] Reference/worked-example story stays read-only (`isRefLocked()` honored).
- [ ] Extraction targets only the chosen group's keys, uses exact keys incl. `char.` prefix, and degrades gracefully to a raw-transcript review row on parse/API failure.
- [ ] Extraction model is user-selectable (dropdown + custom slug) per the user's decision; do not force a silent default.
- [ ] Mic permission denial, no-mic, network error, and oversized recording are all handled with visible, recoverable messaging.

---

## 7. References

- OpenRouter audio/transcription: https://openrouter.ai/docs/guides/overview/multimodal/stt and https://openrouter.ai/blog/announcements/announcing-audio-apis/
- Whisper on OpenRouter: https://openrouter.ai/openai/whisper-1
- Chat completions (extraction): https://openrouter.ai/docs

*Key anchors in `index.html` as of writing: template block ~L571; field lists `metaP`/`metaC` ~L575–585; `concepts` ~L653+; `mk()` ~L752; `freshUI()` ~L876; `setFieldText`/`commitVersion` ~L1003–1016; export/modal pattern ~L545+ and `genJSON` ~L1055; `renderVals()` ~L1108. Verify line numbers before editing — they drift.*

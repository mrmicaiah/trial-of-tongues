# Trial of Tongues

A puzzle box for one apprentice. The wizard is cocky, dismissive, and occasionally — rarely — rattled.

This document is the complete specification. Build to it. There is no prior version.

---

## What it is

A single-page web app. One user. The user opens a URL only they have, and meets a wizard who refuses to teach an apprentice he hasn't measured. The wizard administers seven **trials** to calibrate the apprentice's mind across different cognitive domains. Once calibration is complete, the wizard begins issuing **daily puzzles** tuned to the apprentice's profile, escalating in difficulty as the wizard's frustration grows.

The wizard is a character. Everything the apprentice sees is in the wizard's voice. There is no mention of AI, models, Claude, Anthropic, or the underlying technology anywhere in the UI. The illusion is the product.

---

## The three surfaces

The app has three views. Only one is visible at a time.

### 1. The Chamber

The primary surface. Where puzzles happen.

- A scene illustration at the top — the wizard at his desk, lit by candlelight, his expression shifting with mood.
- The puzzle text below, set in a serif display face.
- The apprentice's input area at the bottom — single text input, plain, no chat-bubble framing.
- The wizard's reactions appear above the input, in prose, in his voice.

The Chamber is where the apprentice spends 95% of their time. Trials and daily puzzles both render here.

### 2. The Dossier

The wizard's notes on the apprentice. Accessed via a small affordance in the corner of the Chamber — a quill icon, or the word "DOSSIER" in small caps.

The dossier is *written by the wizard*. It is not a clinical readout. It contains:

- **Standing observations** — the wizard's running assessment of the apprentice across the seven domains. Updated after every puzzle.
- **Behavioral notes** — what the wizard has noticed about *how* the apprentice thinks. Probes carefully. Refuses to guess. Bargains. Etc.
- **Grudges** — specific moments the wizard has not forgotten. Suspiciously fast solves. Unintended valid answers. Caught contradictions.
- **The verdict** — once calibration completes, a final sealed assessment. Re-opens and revises only on rare events.

The dossier may, on rare occasions, contain things that are unfair, contradictory, or quietly self-aggrandizing. This is deliberate. The wizard is the author.

### 3. The Hearth

The quiet surface. Where the apprentice goes when the wizard isn't around — between sessions, after a daily puzzle is complete, or when the wizard has stormed off.

- A simple view: today's date, the apprentice's streak, the next puzzle's available time ("the wizard returns at dawn"), and a short note from the wizard, if he left one.
- No input. The Hearth is read-only.
- If the apprentice is mid-trial or mid-daily-puzzle, the Hearth is replaced by the Chamber.

---

## The arc

### Phase A: The Trials (calibration)

The apprentice's first ~week. Seven trials, one cognitive domain each. Each trial is a short session (3–7 minutes) with 2–3 puzzles inside. The apprentice can do one trial per session, or burn through several in a day if they're hooked. Progress is saved between trials.

The seven trials, in the order the wizard prefers to administer them (the order is part of the wizard's strategy):

1. **The Trial of Reason** — deductive logic, rule-chaining
2. **The Trial of Sight** — pattern recognition, sequences
3. **The Trial of Tongues** — wordplay, anagrams, linguistic reframing
4. **The Trial of Shadows** — lateral thinking, the "aha" puzzles
5. **The Trial of Shapes** — spatial reasoning, mental rotation
6. **The Trial of Memory** — information held across turns, callbacks
7. **The Trial of Cunning** — open-ended; the wizard sets a scenario and watches how the apprentice probes it

Full trial designs live in [`trials.md`](trials.md).

At the end of all seven, the wizard delivers a **verdict** — a sealed paragraph in the dossier, in his voice, summing up the apprentice. The verdict is the gate between Phase A and Phase B.

### Phase B: The Daily Puzzles

From day 8 onward, the wizard issues one puzzle per day. Each puzzle is generated live by the wizard based on the apprentice's current dossier — which domains they're strong in, which they're weak in, what the wizard most wants to exploit today, and what the wizard's current mood is.

The wizard's mood drifts over time. He starts cocky. As the apprentice succeeds, his mood escalates: frustrated, petty, openly bitter, paranoid. Mood affects:

- The flavor of his greeting
- The unfairness of the puzzle constraints ("you may not use vowels in your answer")
- Whether he accuses the apprentice of cheating
- Whether he tries to cheat himself (see Phase C)

If the apprentice fails a puzzle, the wizard's mood eases slightly. Failure restores him.

### Phase C: The Reckoning (post-MVP, but spec it for direction)

After ~30 successful daily puzzles, the wizard's behavior starts to crack. He begins issuing puzzles that are unfair, unsolvable, or quietly rigged. The meta-puzzle is recognizing *that the puzzle itself is the trick.*

The ultimate beat: the wizard, in a moment of forced graciousness, asks the apprentice to *give him* a riddle. If the apprentice can stump the wizard, the dossier is sealed forever with a final entry — written by the wizard, grudgingly, conceding nothing.

Phase C is not in the MVP build. It is recorded here so the architecture leaves room for it.

---

## Identity & access

One user: the apprentice (Isaac).

- The app is accessed via a **magic link URL** containing a single opaque token: `https://trial-of-tongues.<worker-domain>/?key=<TOKEN>`.
- The token is generated once and shared with the apprentice out-of-band (text message, etc.).
- The token is also stored as `APPRENTICE_KEY` in the Worker's secrets. On every request, the Worker checks the URL key matches.
- On first visit, the frontend stores the key in `localStorage`. Subsequent visits to the bare URL (no `?key=`) auto-redirect to the keyed URL.
- No login screen. No sign-up. No multi-user. If the URL leaks, regenerate the token; the dossier survives.

This is identity-by-URL. The URL *is* the apprentice. There is no concept of a "user" beyond this.

---

## The bright line

**The wizard is the only voice the apprentice ever hears.** No system messages, no error toasts in a generic UI voice, no loading spinners with neutral copy. Every piece of text the apprentice sees is written in the wizard's voice, or is silent.

- Errors → the wizard "loses his train of thought" or "the candle gutters"
- Loading → "the wizard considers..." with a slow ink-blot animation
- Empty states → the wizard's absence is itself a note ("the chamber is empty. he will return.")
- Network failure → "the chamber grows dim. try again when the light returns."

If the illusion would have to break to deliver a message, the message is rewritten or omitted. The wizard is the world.

---

## Architecture

### Stack

- **Cloudflare Worker** — single Worker, routes `/api/*` and serves the SPA assets
- **D1** — operational state (the dossier, session log, puzzle history, mood)
- **Frontend** — Vite + React + TypeScript + Tailwind v3. SPA, single bundle, shipped from the same Worker.
- **Anthropic API** — the wizard's voice. All model calls go through here.

No Durable Objects. No local agent. No GitHub integration. This is a much smaller system than The Big Brain — pin that mentally.

### Cloudflare bindings

- `DB` — D1 database binding
- `ASSETS` — static assets binding (modern `[assets]` block in wrangler.toml, with `not_found_handling = "single-page-application"` for client routing)

### Data model

```sql
-- The dossier itself. One row, ever. The wizard's living document.
CREATE TABLE dossier (
  id INTEGER PRIMARY KEY CHECK (id = 1),  -- singleton row
  verdict TEXT,                           -- NULL until calibration completes; then sealed paragraph
  mood TEXT NOT NULL DEFAULT 'cocky',     -- cocky | frustrated | petty | bitter | paranoid
  mood_score INTEGER NOT NULL DEFAULT 0,  -- drift counter; 0 = cocky, climbs with success, drops with failure
  created_at TEXT NOT NULL DEFAULT (datetime('now')),
  updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- Standing observations across the seven domains. One row per domain.
CREATE TABLE domain_observations (
  domain TEXT PRIMARY KEY,                -- 'reason' | 'sight' | 'tongues' | 'shadows' | 'shapes' | 'memory' | 'cunning'
  rating TEXT NOT NULL DEFAULT 'unknown', -- unknown | irritating | passable | unremarkable | concerning
  note TEXT,                              -- the wizard's prose observation about this domain
  updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- Behavioral notes — how the apprentice thinks, not what they solve.
CREATE TABLE behavioral_notes (
  id TEXT PRIMARY KEY,                    -- UUID
  note TEXT NOT NULL,                     -- the wizard's prose, e.g., "this one probes"
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- Grudges — specific moments the wizard refuses to forget.
CREATE TABLE grudges (
  id TEXT PRIMARY KEY,
  puzzle_id TEXT,                         -- FK to puzzles, nullable
  note TEXT NOT NULL,                     -- the wizard's prose recording the grudge
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- Every puzzle that has ever been issued.
CREATE TABLE puzzles (
  id TEXT PRIMARY KEY,                    -- UUID
  phase TEXT NOT NULL,                    -- 'trial' | 'daily'
  trial_name TEXT,                        -- 'reason' | 'sight' | ... — NULL for daily
  prompt TEXT NOT NULL,                   -- the wizard's puzzle text shown to the apprentice
  solution_notes TEXT NOT NULL,           -- the wizard's private notes on what counts as a correct answer
  domain TEXT NOT NULL,                   -- which domain this puzzle stresses
  issued_at TEXT NOT NULL DEFAULT (datetime('now')),
  resolved_at TEXT,                       -- when the apprentice's attempt was judged
  outcome TEXT,                           -- 'correct' | 'incorrect' | 'partial' | 'abandoned'
  time_to_solve_seconds INTEGER,          -- from issued_at to resolved_at
  apprentice_answer TEXT,                 -- the final answer the apprentice gave
  wizard_reaction TEXT                    -- the in-character reaction the wizard delivered
);

-- The chat log within a single puzzle attempt. Apprentice messages and wizard reactions.
CREATE TABLE turns (
  id TEXT PRIMARY KEY,
  puzzle_id TEXT NOT NULL,
  role TEXT NOT NULL,                     -- 'apprentice' | 'wizard'
  content TEXT NOT NULL,
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_puzzles_phase ON puzzles(phase, issued_at);
CREATE INDEX idx_turns_puzzle ON turns(puzzle_id, created_at);
CREATE INDEX idx_grudges_recent ON grudges(created_at DESC);
CREATE INDEX idx_behavioral_recent ON behavioral_notes(created_at DESC);
```

The dossier is composed live from these tables — see [`dossier-format.md`](dossier-format.md).

### Endpoints

```
GET  /health                              — open, no auth, returns { ok: true }

# Auth: ?key=<APPRENTICE_KEY> on every /api/* request
GET  /api/state                           — current phase, mood, next puzzle availability, streak
GET  /api/dossier                         — composed dossier (all four sections)

POST /api/puzzle/next                     — request the next puzzle (trial or daily); returns puzzle prompt
POST /api/puzzle/:id/turn                 — submit an apprentice message; streams wizard's reaction
POST /api/puzzle/:id/resolve              — apprentice submits final answer; streams wizard's reaction + outcome
POST /api/puzzle/:id/give-up              — apprentice abandons; wizard reacts (this is rare; the wizard relishes it)
```

No websocket. SSE for streaming. Everything else is plain JSON.

### Streaming protocol

The wizard's reactions stream as Server-Sent Events:

```
event: text
data: {"delta": "..."}

event: mood
data: {"mood": "frustrated", "score": 4}

event: outcome
data: {"outcome": "correct", "time_seconds": 47}

event: dossier_updated
data: {}

event: done
data: {}

event: error
data: {"message": "the candle gutters"}
```

The `dossier_updated` event tells the frontend to refresh the dossier badge (a small indicator that the wizard has written something new). The frontend does not auto-open the dossier — that would break immersion. The apprentice opens it when they want to.

### How the wizard works

Every model call to the Anthropic API includes a system prompt assembled from:

1. The **base wizard prompt** ([`wizard-prompt.md`](wizard-prompt.md)) — character, voice, behavior rules, mood scarcity rules, what the wizard never says.
2. **The current dossier** — composed live from the D1 tables. The wizard reads his own dossier on every turn.
3. **The current puzzle** — prompt and solution_notes (private; the wizard knows, the apprentice doesn't).
4. **The turn history within this puzzle** — what's been said in this attempt so far.
5. **The current mood** — explicit instruction on how to colour the response.

The model produces:

- A **streaming in-character reaction** (text deltas, streamed to the apprentice).
- Optional **structured updates** at the end of its response — fenced blocks that the Worker parses out before persisting. These are how the wizard updates his own dossier.

Fenced block formats (the wizard emits these at the end of his response; the apprentice never sees them):

````
```update_domain
domain: shadows
rating: irritating
note: solved the elevator riddle in under a minute. has heard it before, must have.
```

```add_behavioral_note
note: this one refuses to guess. either confident or stubborn.
```

```add_grudge
note: solved the cipher in four minutes. i have not forgotten.
```

```adjust_mood
delta: +1
reason: another correct answer, predictably.
```

```seal_verdict
verdict: <paragraph>
```
````

The Worker parses these out of the stream, applies them to D1, and emits a `dossier_updated` event to the frontend. The fenced blocks themselves never reach the apprentice's UI.

### Puzzle generation

For **trials**, the puzzle prompts are *authored* — pre-written in [`trials.md`](trials.md). Each trial has a fixed set of 2–3 puzzles. The wizard's reactions to those puzzles are generated live; the puzzles themselves are not. This guarantees calibration quality and predictability.

For **daily puzzles**, the puzzle is generated live by Claude. The wizard is prompted with:

- The full dossier
- Today's target domain (chosen by a small algorithm: rotate through domains weighted by where the wizard most wants to exploit the apprentice's profile)
- Today's target difficulty (climbs with `mood_score`)
- A constraint to produce both the puzzle prompt AND the solution_notes in the same call, separated cleanly

The Worker stores the puzzle, returns the prompt to the frontend, and keeps solution_notes private.

### Frontend

Vite + React + TypeScript + Tailwind v3. Single bundle. SPA routing.

#### Layout

Three views. One visible at a time. No persistent navigation chrome — the apprentice is *in* the world, not navigating an app.

- **Chamber** — default view. Scene illustration top, puzzle prose middle, input at bottom. A small `[dossier]` link in small caps in the upper-right corner. A small `[hearth]` link in small caps in the upper-left corner, only visible between puzzles.
- **Dossier** — full-surface takeover. The dossier rendered as if it were a leatherbound book the apprentice has opened. Single back link to return to the Chamber ("close the book").
- **Hearth** — full-surface takeover. Quiet, sparse. Date, streak, time until next puzzle, optional note from the wizard. Single "return to the chamber" link if a puzzle is active.

#### Visual language

- Background: parchment (`#F4ECD8`), with a fine grain texture and slight vignette at the edges.
- Ink: deep brown-black (`#1A1308`), not pure black. Handwriting feel.
- Accent: candle-gold (`#C9A65B`), used sparingly — verdict seal, dossier badge, hover states.
- Wizard's voice: a serif display face (suggest: **Cormorant Garamond** or **EB Garamond**), slightly larger than body, with italics for emphasis. The wizard speaks in *prose*.
- Apprentice input: monospace, smaller, lower-contrast — the apprentice is the smaller voice in the room.
- No buttons that say "Submit" or "Send." The apprentice presses Enter. If a button must exist (e.g., "give up"), it's labeled in the wizard's voice ("yield" or "flee the chamber").
- No chat bubbles. No timestamps. No avatars. The conversation is laid out as correspondence — italicized actions in parentheses, prose on a parchment page.
- Loading: a slow ink-blot expanding from the cursor position. No spinners.
- Errors: the candle gutters (a brief darkening of the page), and the wizard murmurs about being distracted.

---

## Environment

Required environment variables on the Worker (set as Cloudflare secrets):

- `ANTHROPIC_API_KEY` — for the wizard's voice
- `APPRENTICE_KEY` — the opaque token in the magic-link URL

**Model ID:** all Claude model calls use `claude-opus-4-7`. Pin this as a single exported constant `MODEL_ID` in `src/lib/claude.ts` so a future change is one line.

**The Big Brain worker:** Trial of Tongues runs on its own Worker, not as a path on The Big Brain. The two systems share no code. They may share infrastructure (same Cloudflare account, similar conventions), but the wiring is independent.

**Local dev:**
- Vite dev server on `http://localhost:5173`, proxies `/api/*` to the Worker.
- Wrangler dev on `http://localhost:8787`.
- Run both with `npm run dev` at the root (concurrently).

---

## Build order

Each phase ships a working state.

**Phase 1: Foundation**
- Cloudflare Worker scaffold + D1 schema applied
- `APPRENTICE_KEY` auth gate on `/api/*`
- `/health` endpoint
- Frontend scaffold (Vite + React + Tailwind v3, design language applied)
- The app loads at the magic-link URL. Empty Chamber. The wizard is silent.

**Phase 2: The wizard speaks**
- `wizard-prompt.md` finalized
- Anthropic API client (`src/lib/claude.ts`) with streaming
- `POST /api/puzzle/next` — issues the first trial puzzle (hardcoded from `trials.md`)
- `POST /api/puzzle/:id/turn` — streams the wizard's reaction
- Chamber UI renders puzzle, accepts apprentice input, streams wizard's reaction inline
- The apprentice can have a conversation about a single hardcoded puzzle. No dossier yet, no mood, no progression.

**Phase 3: The dossier**
- D1 tables for dossier, domain_observations, behavioral_notes, grudges
- Fenced block parser for wizard self-updates
- `GET /api/dossier` composes the dossier from tables
- Dossier view in frontend (full-surface, parchment book)
- The wizard now updates his dossier after each turn. The apprentice can open it and read it.

**Phase 4: The trials**
- All seven trials authored in `trials.md` and loaded into the Worker
- `POST /api/puzzle/next` advances through the seven trials in sequence
- Trial-completion logic; verdict generation at the end of trial 7
- Verdict sealing in the dossier
- The apprentice can complete the full calibration arc.

**Phase 5: Daily puzzles**
- Live puzzle generation for Phase B
- Mood tracking and mood-driven puzzle difficulty
- Daily availability gate (one puzzle per 24h)
- Streak tracking
- The Hearth view (between-puzzle state)
- The apprentice now has an indefinite arc.

**Phase 6 (post-MVP): The Reckoning**
- Rigged puzzles, the wizard's cracks
- The riddle-back endgame
- Final dossier seal

Each phase requires the previous. Don't skip ahead.

---

## What's NOT in scope (MVP)

- Multi-user. One apprentice. Ever.
- Mobile-specific layout. Build for desktop first; the design language degrades acceptably on mobile but isn't tuned for it.
- Account recovery. If the URL is lost, regenerate the token; the dossier survives in D1.
- Sharing the dossier publicly. The dossier is private to the apprentice.
- Sound. Beautiful candle-flicker audio is tempting and deferred.
- Localization. English only.
- Hint systems. The wizard does not give hints. He may, on rare occasions, *gloat in a way that contains a hint*, but this is a character beat, not a feature.
- The Reckoning (Phase 6) — designed for, not built in MVP.

---

## Known limitations

Things consciously deferred. Recorded so they're not forgotten when they bite.

- **The token in the URL is in localStorage and the browser history.** If the apprentice's machine is compromised, the dossier is exposed. Acceptable for this toy. Regenerate the token to invalidate.
- **D1 is the only durability layer.** If D1 is wiped, the dossier is lost. The system makes no attempt to back up to GitHub or elsewhere. Acceptable; this is a toy.
- **The wizard's voice is model-generated and not perfectly consistent.** The base prompt does a lot of work, but variance will show across long arcs. Phase 5+ may need a periodic "voice audit" pass where a maintainer reviews recent reactions and tightens the prompt.
- **Puzzle quality on the daily-puzzle phase is at the mercy of the model.** Some daily puzzles will be weaker than the authored trials. Acceptable; the trials carry the calibration weight, and the dailies carry the *character* weight.
- **Difficulty calibration is heuristic.** Mood score → difficulty is a hand-tuned mapping, not a model of the apprentice's skill. Refine in Phase 5 if it feels off.

---

## Repo layout

```
trial-of-tongues/
├── README.md
├── specs/
│   ├── SPEC.md                — this document
│   ├── wizard-prompt.md       — the wizard's system prompt
│   ├── trials.md              — the seven trials, authored
│   └── dossier-format.md      — how the dossier is composed from D1
├── src/                       — Cloudflare Worker
│   ├── index.ts               — entry, routing
│   ├── types.ts
│   ├── lib/
│   │   ├── claude.ts          — Anthropic API client, MODEL_ID constant
│   │   ├── wizard.ts          — system prompt assembly, dossier composition
│   │   ├── parser.ts          — fenced block parser for wizard self-updates
│   │   └── trials.ts          — loads authored trials from specs/trials.md
│   └── db/
│       ├── schema.sql
│       └── migrations/
├── web/                       — frontend (Vite + React)
│   ├── package.json
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   └── src/
│       ├── main.tsx
│       ├── App.tsx
│       ├── views/
│       │   ├── Chamber.tsx
│       │   ├── Dossier.tsx
│       │   └── Hearth.tsx
│       ├── components/
│       ├── lib/               — api client, SSE handling
│       └── styles/
├── wrangler.toml
└── package.json
```

---

## Principles

**One apprentice.** Single-user. No accounts. No invitations. The URL is the apprentice.

**The wizard is the world.** No system voice. No generic UI copy. If the illusion must break, the message is rewritten or omitted.

**The dossier is alive.** The wizard reads it before every response. He writes to it after every response. It is the system's memory and the system's character at once.

**Scarcity sells the bit.** The wizard is rattled rarely. Compliments are rarer. The apprentice should remember the moments the wizard cracked.

**Authored trials, lived dailies.** The seven trials are fixed-quality calibration content. The daily puzzles are character. The split is on purpose.

**Editorial restraint.** Parchment, ink, candle-gold, garamond. No buttons that say "submit." No chat bubbles. The product should feel like a thing the apprentice found, not a thing they signed up for.

**Don't apologize for limits.** The wizard does not say "I cannot help with that." He sneers, he refuses, he changes the subject. Limits are character beats.

---

## A note on building this

This spec is the bible. Build to it. If something feels off, ask before deviating. If something genuinely needs to change, change the spec first, then the code.

Every piece of copy that the apprentice will see should be reviewed for voice. "Loading..." is a bug. "the wizard considers..." is correct.

The principal is one apprentice. The wizard serves the bit. The bit serves the apprentice's delight.

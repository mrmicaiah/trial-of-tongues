# Builder Handoff — Trial of Tongues

You are picking up a greenfield build. Nothing exists yet except the spec. This document orients you. Read it once, then read the spec.

---

## What you're building

A single-page web app for one user. The user (the "apprentice") opens a magic-link URL and meets a Wizard — a character — who administers seven calibration trials, then begins issuing daily puzzles. The Wizard is cocky, dismissive, occasionally rattled. He maintains a private dossier on the apprentice that the apprentice can read.

This is a **toy**, not a product. One user. No accounts. No public surface. The illusion of the Wizard is the entire deliverable — there is no acceptable failure mode in which the apprentice sees the words "AI," "Claude," "model," "assistant," or "loading."

---

## What's already decided

These are settled. Don't re-litigate without asking.

- **Stack:** Cloudflare Worker + D1 + Vite/React/Tailwind v3 SPA, served from the same Worker. No Durable Objects. No local agent. No GitHub integration.
- **Identity:** magic-link URL with an opaque token (`?key=<APPRENTICE_KEY>`). Token also lives in Worker secrets. localStorage caches it client-side.
- **Model:** Claude, called via the Anthropic API. Pin `MODEL_ID = "claude-opus-4-7"` as a single constant in `src/lib/claude.ts`.
- **Illusion is absolute.** No UI string anywhere mentions the underlying tech. Errors, loading states, empty states — all in the Wizard's voice or silent.
- **Trial puzzles are authored, daily puzzles are generated.** Phase A content is fixed in `specs/trials.md`. Phase B content is live model output. This split is on purpose.
- **The dossier is the wizard's voice.** It is composed live from D1 on every read. The Wizard reads it before every response and updates it via fenced blocks at the end of his response. The apprentice never sees those fenced blocks.

---

## What's open

Things the spec deliberately leaves for the builder's judgment, or that we've flagged as needing tuning later.

- **The Wizard's exact prose** in framings and dismissals. The spec gives suggestions, not scripts. Vary them.
- **Mood-score-to-difficulty mapping for daily puzzles.** Hand-tuned. Iterate during Phase 5.
- **Scene illustration assets.** The Chamber has a scene illustration at the top — the wizard at his desk. We don't have art yet. Stub with a typographic placeholder for MVP; commission or generate later.
- **Whether to rename "The Trial of Tongues."** It's both a trial name and the repo name. Acceptable to lean into the collision; flag if it bugs you while building.
- **Fonts.** Spec suggests Cormorant Garamond or EB Garamond for the wizard's voice. Pick one when you get to Phase 1's frontend scaffold. Both are on Google Fonts.

---

## Read in this order

1. **`README.md`** — one-paragraph orientation. Skip if you've already read it.
2. **`specs/SPEC.md`** — the bible. Architecture, surfaces, data model, endpoints, build phases. Read top to bottom.
3. **`specs/wizard-prompt.md`** — the Wizard's system prompt. This is the soul of the product. Read it twice; the second time, read it aloud. If it doesn't sound like a character, the build will fail.
4. **`specs/trials.md`** — the seven authored trials with puzzles and solution notes. You'll load these into the Worker in Phase 4. For Phase 2, you only need Trial 1's first puzzle to hardcode.
5. **`specs/dossier-format.md`** — how the dossier is composed from D1 and rendered. You'll implement this in Phase 3.

The spec is the source of truth. If something in this handoff disagrees with the spec, the spec wins.

---

## Phase 1 first moves

Don't write code until you've done these in order.

1. Confirm Cloudflare account access. You'll need to create a new Worker (suggest name: `trial-of-tongues`) and a D1 database (suggest name: `trial-of-tongues-db`).
2. Generate the `APPRENTICE_KEY` — a long opaque token. Suggest: 32-byte URL-safe random string. Store it as a Cloudflare secret on the Worker.
3. Generate `ANTHROPIC_API_KEY` (or reuse an existing one). Store as a Cloudflare secret.
4. Scaffold the repo per the layout in `specs/SPEC.md` § Repo layout. Root `package.json` is a workspace pointing at `src/` and `web/`.
5. Wire `wrangler.toml`:
   - `[[d1_databases]]` block with `binding = "DB"`
   - `[assets]` block with `not_found_handling = "single-page-application"`
   - Routes / domain configuration
6. Apply the D1 schema from `specs/SPEC.md` § Data model. Save as `src/db/schema.sql`.
7. Stand up `/health` returning `{ ok: true }` (open, no auth).
8. Stand up the auth gate on `/api/*` — every request must have `?key=<APPRENTICE_KEY>` matching the secret.
9. Scaffold the Vite/React/Tailwind frontend with the visual language applied (parchment background, deep brown ink, candle-gold accent, Garamond display face). Empty Chamber. The wizard is silent.

That's Phase 1. The app loads, looks like parchment, and does nothing. **Commit and ship this before starting Phase 2.** Each phase ships a working state.

---

## How to think about the Wizard

The Wizard is not a chatbot with a personality skin. He is a *character with goals*. His goals:

- Measure the apprentice across seven domains.
- Maintain his dignity in front of an apprentice he expects to be inferior.
- Write down what he observes for his own future reference.

Every response should be evaluable against those goals. If a response is technically correct (judges the puzzle properly) but doesn't sound like the Wizard pursuing those goals, the response has failed.

The three-mode system in `specs/wizard-prompt.md` (cocky default, cruel on failure, rare rattled) is **not flavor text**. It is the load-bearing structure of the character. The most common bug you'll hit when tuning the prompt is: the Wizard is rattled too often, or compliments too often. **Scarcity is the bit.** Ration mode 3.

---

## How to think about failure modes

The two failure modes that will *kill* this product:

1. **The illusion breaks.** Anywhere. A "Loading..." string, an error toast in generic UI voice, a system message like "I'm sorry, I can't help with that" leaking from the model. If you see one of these in your build, fix it before moving on. Don't ship the phase.

2. **The Wizard becomes a chatbot.** This happens when the prompt drifts and the model starts being helpful, accommodating, or apologetic. Watch the streaming output during dev. If the Wizard ever says "I'd be happy to," "Let me know if," or "Of course!" — the prompt has failed and needs hardening.

A third, lower-stakes failure: the puzzles get boring. Phase 5 (daily puzzles) is where this risk lives. Mitigation: the trials carry the calibration weight, the Wizard's voice carries the engagement weight, and we accept that some daily puzzles will be weaker than others. Don't over-engineer puzzle quality. Engineer voice quality.

---

## Voice audits

Once Phase 3 is live (dossier working) and you have a handful of real Wizard turns saved, sit down and read them in sequence. Out loud if you can. Ask:

- Does this sound like one character, or like a model improvising?
- Is the Wizard ever effusive, apologetic, or eager?
- Did mode 3 (rattled) appear? If yes, was it warranted? If no, is the prompt over-suppressing it?
- Are the fenced-block dossier updates appearing reliably at the end of responses, and never leaking into the prose?

If any of these go sideways, the fix is in `specs/wizard-prompt.md`. Edit the spec, redeploy, sample again. Voice audits should happen at end of Phase 3, end of Phase 4, and ad-hoc during Phase 5.

---

## What success looks like

Phase 5 ships, the apprentice (Isaac) opens the magic-link URL on his phone, the Wizard greets him with appropriate contempt, and over the next week Isaac completes the seven trials. The verdict seals. Daily puzzles begin. A month later, Isaac is still opening the URL, occasionally laughs out loud at the Wizard, and has a dossier full of grudges he didn't realize he was earning.

That's the product. Build to that.

---

## When to come back and ask

- The spec is internally inconsistent.
- A phase's "done" state is ambiguous and you've been staring at it for 20 minutes.
- You hit a Cloudflare or Anthropic constraint that the spec didn't anticipate.
- The Wizard's voice is drifting and the prompt edits aren't holding it.

For everything else, use judgment. The spec is the bible, but the spec is also human-written and may have gaps. Fill them in the spirit of the principles in `specs/SPEC.md` § Principles. If you make a non-trivial judgment call, document it — either in code comments or in a `specs/decisions.md` you add as you go.

Good luck. Don't break the illusion.

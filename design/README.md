# Design — Trial of Tongues

Mobile-first design for an iPhone PWA. The wizard is the world.

This folder contains the mockups and design philosophy. The spec in `../specs/SPEC.md` is the source of truth for *what* the system does; this folder is the source of truth for *how it looks and feels*.

If a builder is implementing this, read the spec first, then read this folder. If something here contradicts the spec, the spec wins — flag the contradiction so it can be resolved.

---

## Why mobile-first

Isaac will mostly open this on his phone. The whole arc — daily puzzles, glance at the dossier, see if the wizard left a note — is shaped for the moment between other things, not for a desktop sit-down session. Mobile is the primary canvas; if it has to also work in a laptop browser, the same column scales up gracefully on a wide screen with generous margins. But mobile is what gets designed for.

Within mobile, the baseline target is **iPhone, Safari, installed as a PWA via Add to Home Screen.** That means: no browser chrome, status bar on top (44pt), home indicator at the bottom (34pt), the full safe area in between is ours. PWA viewport is `viewport-fit=cover` and the manifest declares `"display": "standalone"`.

Android is a second-class target. The design works on Android Chrome but isn't tuned for it.

---

## The three views

Only one view is on screen at a time. There is no persistent nav bar, no tab strip, no drawer handle. The apprentice is *in the world*, not navigating an app.

### Chamber

Where puzzles happen. The default view when a puzzle is active. Single scrolling column:

- **Header** (top): a small candle motif on the left, a `dossier` link on the right. That's it. No trial title, no progress indicator, no chrome. The candle is the only "logo" the product has.
- **Conversation body**: streams downward. Wizard's framing in italic stage directions, puzzle prose in regular serif, apprentice answers right-aligned in monospace (the apprentice is the smaller voice in the room), wizard's responses in serif again. Hairlines separate puzzle from back-and-forth.
- **Input** (above the keyboard): a single text field, placeholder *"speak, apprentice"* in italic. No send button. Press return.
- **Dossier-updated marker**: when the wizard writes to his dossier, a small italic candle-gold line surfaces in the conversation feed (*"he wrote something down."*). This is the only signal. The apprentice can open the dossier on their own time.

Streaming text appears character-by-character — the wizard's speech is the heartbeat of the product. Use a slow ink-blot cursor (a small dot, not a bar) that pulses while the model is generating. When the response is complete, the cursor disappears.

### Dossier

The wizard's book. Accessed from the Chamber by tapping `dossier`. Full-surface takeover with a subtle page-turn transition (the parchment shifts in from the right edge, faint ink-bleed at the seam).

Layout (top to bottom):

1. **Close link** at top-left: `‹ close the book`. The only navigation affordance.
2. **The Verdict** with the seal emblem to its left. Italic prose paragraph. If the verdict is not yet sealed (calibration not complete), this section is replaced with `*the page is blank where the verdict should be. the wizard has not yet decided.*`
3. **Standing Observations** — each domain as a two-line entry: domain name in small-caps + rating in italic candle-gold on line one; the wizard's prose note in italic body on line two. Domains not yet measured are dimmed.
4. **Notes** — behavioral notes with small ink-dot bullets, italic prose.
5. **Grudges** — heavier visual weight. Oversized `G` initial cap. Each grudge gets its date stamp on a second line below, italic, candle-gold.

The page scrolls. No fold-out sections, no tabs. Long scroll, like reading a journal.

### Hearth

The empty-room view. Shown when no puzzle is active. The apprentice lands here if they open the URL outside an active session.

Layout:

1. **The dim chamber** (top half): A single low candle, centered. Heavy vignette. The room is dark because the wizard isn't there.
2. **The note**: *"the chamber is empty. the wizard has stepped out."* Two lines, italic, centered, low contrast.
3. **The countdown**: `HE RETURNS AT DAWN.` in small caps, then a relative time in italic ("in 7 hours, 23 minutes").
4. **Trial counter** (in candle-gold): `FOUR TRIALS REMAIN.` Or, post-calibration: `DAY 23.` Or, post-Reckoning: `THE BOOK IS SEALED.`
5. **The wizard's note** (sometimes): A short note he left for the apprentice. In italic prose. Usually a dig.

The Hearth has no input. It is a read-only view. If the apprentice taps the candle, nothing happens — there is nothing to do here. The room is empty.

---

## Visual language

| Token | Value | Use |
|---|---|---|
| Parchment (Chamber) | `#F4ECD8` | The active surface, warm and lit |
| Parchment (Dossier) | `#EFE5CC` | Slightly darker, this is a book |
| Parchment (Hearth) | `#E8DCC1` | Dimmer still, the candles are low |
| Ink | `#1A1308` | All body and display text |
| Candle gold | `#C9A65B` | Verdict seal, date stamps, mood indicators, dossier-updated marker, trial counter |
| Aged gold (dark) | `#8B6F2D` | Rating labels in standing observations |

**Type:**

- **Display & body**: EB Garamond. The wizard speaks in serif prose. Italics for stage directions and emphasis. The rendering snapshot in this folder uses EB Garamond's stylistic conventions throughout.
- **Apprentice voice**: JetBrains Mono, smaller (13px), at ~55% opacity. The apprentice is the smaller, less imposing voice in the conversation.
- **Small caps**: section headers (`STANDING OBSERVATIONS`, `NOTES`, `GRUDGES`) and the Hearth's status lines use letter-spaced small caps in serif — *not* Inter or system sans. The book has no sans.

**Sizing:**

- Wizard prose: 17px on iPhone (room to breathe)
- Italic stage directions: 14px (smaller, lower-contrast)
- Apprentice mono: 13px
- Section labels (small caps): 11px with `letter-spacing: 2.5px`
- Verdict text: 14px italic (it's a body paragraph)

**Spacing:**

- Generous vertical rhythm. Hairlines (0.3px, 18-22% opacity) separate sections, not borders.
- The page should breathe. White space is the canvas the wizard writes on.

**Effects:**

- Parchment grain: a subtle dot texture at 6-7% opacity. Suggests paper without becoming a Skeuomorphic Renaissance Fair.
- Vignette: radial darkening at the edges, more pronounced on the Hearth, lighter on the Chamber.
- No drop shadows, no glow, no embossing. The medium is ink on paper, not pixels.
- Streaming ink-blot cursor: a small dot that pulses opacity and radius. Slow (1.4s cycle). Not a typewriter beat.

---

## Interaction patterns

**Tap targets**: at least 44pt. The two interactive elements per view (`dossier` link in Chamber, `close the book` in Dossier) are deliberately small text but have a generous tappable padding around them.

**Gestures**: 

- Vertical scroll within a view, native. 
- No swipe-to-navigate between views in MVP — every transition is via an explicit tap. (We can add swipe-to-dossier later; not in the initial build.)
- Long-press: nothing. The wizard does not have context menus.

**Keyboard behavior**: When the input field is focused, the keyboard slides up and the conversation scrolls to keep the input visible. The wizard's most recent message must remain in view above the input — don't let the keyboard cover the live conversation.

**Sound**: none in MVP. Tempting to add candle-flicker audio or quill-scratching, but it would feel toy-like. The silence is part of the room.

---

## Streaming as the heartbeat

The wizard's responses stream character-by-character. This is non-negotiable for the feel. The pacing should be roughly natural reading speed — fast enough not to drag, slow enough that Isaac watches the wizard think.

Implementation notes for whoever builds this:

- SSE from the Worker streams `event: text` deltas as the model produces them. The frontend appends to a buffer and reveals at a smoothed rate (roughly 30-50 chars/second after a small initial burst).
- The ink-blot cursor sits at the current insertion point, pulsing.
- Stage directions in italic (`(the wizard sets down his book.)`) should appear *before* the wizard's main response, often as a separate paragraph. The prompt instructs the wizard to lead with one when warranted.
- When the wizard pauses mid-thought, the apprentice should feel it. Em-dashes, ellipses, and the occasional "—" in the wizard's prose are how he buys time. The streaming respects them.

---

## What the mockups show

| File | What's in it |
|---|---|
| `chamber-active.svg` | Chamber with keyboard up, wizard mid-stream after the apprentice answered "bertin." to Trial 1 puzzle 1. The streaming cursor is visible. |
| `chamber-idle.svg` | Same conversation, keyboard dismissed, wizard's response complete. The candle-gold *"he wrote something down."* line is visible — this is the dossier-updated signal. |
| `dossier.svg` | Full dossier view, scrolled to the top. Shows verdict (sealed), standing observations across four domains, three behavioral notes, one grudge. |
| `hearth.svg` | Quiet between-puzzles view. Dim candle, "the chamber is empty," countdown to dawn, trial counter, a note left by the wizard. |

Render the SVGs with any browser or `rsvg-convert` to preview. They use real coordinates for an iPhone 14/15 viewport (390 × 844 device pixels).

---

## What the mockups do NOT show

Build artifacts I deliberately left out of the static SVGs because they don't render well as still images:

- **The streaming animation itself.** The cursor in `chamber-active.svg` has an `<animate>` block — it pulses if you open the SVG in a browser. In production, the streaming is character-by-character.
- **Page transitions.** Tapping `dossier` from the Chamber should trigger a brief parchment-slide and ink-bleed effect. Tapping `‹ close the book` reverses it. About 250ms each way.
- **The dossier-updated badge.** The small candle-gold dot next to the `dossier` link in the Chamber header lights up when the wizard has written new entries since the apprentice last opened the dossier. The mockup shows it lit.
- **Error states.** See `states.md` for every edge state and its in-voice copy.

---

## Open design questions

Things I have opinions on but haven't locked. Pick when building.

- **Verdict seal emblem**: I used a four-point star inside the gold circle. Could also be: a quill, a flame, an ink-blot, an initial monogram. Pick something the wizard would actually emboss.
- **Dossier dimming after read**: when the apprentice has read the latest entries, does the gold dot beside `dossier` go out? Probably yes. But the dossier itself shouldn't have unread markers — the wizard wouldn't bother with that.
- **Mood indicator in the Chamber**: the spec talks about the wizard's mood drifting. Should there be any visible signal in the Chamber that mood has shifted? My instinct: no, *unless* it's so dramatic the wizard himself would mention it (paranoid mood, for example). The dossier is where mood lives visibly.
- **Streak in the Hearth**: shown as "DAY 23." or as "FOUR TRIALS REMAIN." during calibration. Not as a number badge or a gamified counter. The wizard would not care about streaks. The text framing should always be in his voice.

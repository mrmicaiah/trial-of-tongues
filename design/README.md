# Design — Mobile

This folder is the visual reference for the builder. Everything in here describes the **mobile** experience, because mobile is the primary surface. A desktop port can come later; if it does, it inherits from these mockups, not the other way around.

---

## Target

- **iPhone, Safari, installed as PWA** (add-to-home-screen, no browser chrome).
- Design baseline: **iPhone 14 / 15 standard** — 390×844 logical pixels.
- Safe areas: 47px top inset (notch + status bar), 34px bottom inset (home indicator).
- Working canvas in portrait, idle: **390 × 763**.
- Working canvas in portrait, keyboard up: **390 × ~410** (keyboard takes ~336px).
- Android Chrome is a secondary concern — the design should degrade gracefully but is not optimized for it.
- Landscape is not supported. The apprentice holds the phone upright.

---

## Principles

**The page is alive.** The Wizard's text streams in character-by-character at the rate of an unhurried ink quill — roughly 25–35 characters per second. The blinking cursor at the end of the stream is part of the visual language. Nothing else on the page should move while the Wizard is writing. No spinners, no fades, no flicker. The eye should follow the cursor.

**Parchment is the surface, not the background.** The whole viewport is parchment, edge to edge. There is no "page on a background" — there is only the page. The status bar and home indicator overlay the parchment.

**Two typefaces, no more.** A display serif for the Wizard (Cormorant Garamond, weight 500 for body, italic for emphasis). A monospace for the apprentice's input (JetBrains Mono or Fira Code, weight 400, smaller and lower contrast). The contrast between the two voices is part of the design.

**One accent color, used sparingly.** Candle-gold (#C9A65B). It appears on the cursor, on the seal in the dossier, on the small candle motif in the header, and almost nowhere else. If everything is gold, nothing is.

**No chat affordances.** No bubbles, no avatars, no timestamps, no "Wizard is typing..." indicator, no read receipts, no Submit button. Text flows down the page like ink. The apprentice types, presses return, and waits.

**Thumb zones matter.** The bottom 1/3 of the screen is the thumb's natural arc. The input bar lives there. The dossier link (top-left chevron) is *deliberately* outside the thumb zone — leaving the chamber should require small intent, not an accidental brush.

---

## Color

| Token | Hex | Use |
|---|---|---|
| `--parchment` | `#F4ECD8` | Background of the entire viewport |
| `--parchment-shadow` | `#E8DFC7` | Vignette toward the edges |
| `--ink` | `#1A1308` | The Wizard's prose, primary text |
| `--ink-faded` | `#3A2E1F` | Section headers, dossier metadata |
| `--ink-ghost` | `#7A6A52` | Apprentice's typed input, timestamps |
| `--candle-gold` | `#C9A65B` | Cursor, seal, candle motif, the rare accent |
| `--candle-gold-deep` | `#8C6F2E` | Hairline rules, hovered/active gold |
| `--ink-error` | `#5C2418` | Reserved — "candle gutters" error state only |

Parchment has a subtle grain. In the SVGs, this is represented with a feTurbulence filter at very low opacity. In implementation, a tiled noise PNG or CSS noise overlay works fine. The texture should be perceptible but not legible — you should *feel* it before you see it.

---

## Type scale

| Use | Family | Size | Weight | Style |
|---|---|---|---|---|
| Wizard prose | Cormorant Garamond | 19px | 500 | regular |
| Wizard emphasis | Cormorant Garamond | 19px | 500 | italic |
| Wizard stage direction | Cormorant Garamond | 16px | 400 | italic, ink-faded |
| Section headers (dossier) | Cormorant Garamond | 14px | 600 | small caps, letter-spacing 0.08em |
| Apprentice input | JetBrains Mono | 15px | 400 | regular, ink-ghost |
| Dossier verdict | Cormorant Garamond | 22px | 500 | regular |
| Dossier domain rating | Cormorant Garamond | 17px | 500 | italic |
| Behavioral note | Cormorant Garamond | 15px | 400 | regular |
| Note date stamp | Cormorant Garamond | 12px | 400 | small caps, ink-ghost |

Line height: 1.55 for Wizard prose, 1.4 for everything else.

---

## Layout — the three surfaces

### Chamber (active puzzle)

Single scrolling column. The header is a thin band — 56px tall, sits below the safe area inset — containing only a small candle motif (12px, candle-gold) and, at the right, the word *dossier* in small caps as a tap target. No back button (this is the home view).

Below the header, the puzzle prose. Below that, the Wizard's reactions as they stream in. New text appends at the bottom and the view auto-scrolls to keep the cursor in the lower third of the visible area — never at the very bottom (the apprentice needs to *see* what's being written, not chase it).

The input bar is pinned to the bottom, above the home indicator. It is a single line — not a multi-line textarea — with a left-pad of 16px, a right-pad of 16px, a top hairline rule in candle-gold-deep at 0.5px opacity 0.4, and the apprentice's text in monospace. There is no Submit button. Return submits. There is no placeholder text — an empty input is just an empty input.

When the keyboard is up, the bottom inset is overlaid by the keyboard. When the keyboard is down, the input bar sits 34px above the bottom edge of the screen, respecting the home indicator.

### Chamber (idle, between turns)

Same layout as active. The difference: the Wizard is silent, the cursor is gone, and the input bar shows the apprentice's last attempted input grayed out (ink-ghost at opacity 0.5), as if it's the last thing he said and the room is quiet now. Tapping the input clears it and brings the keyboard up.

### Hearth (between puzzles)

When there is no active puzzle — the apprentice has just finished one and the next isn't ready, or it's a new day and today's puzzle hasn't been claimed — opening the URL goes here directly. No navigation needed.

The Hearth is sparse. Center of the viewport: a small motif (a banked candle, candle-gold on parchment, ~80×80). Below it, a single line of Wizard prose — typically something dismissive or atmospheric. ("The chamber rests. So do I.") Below that, if a puzzle is ready, a tap target: *enter the chamber* in small caps. If no puzzle is ready, nothing — just the candle and the line.

The dossier link sits in the same place as in the Chamber: top-right of the header band.

There is no gesture-based navigation *to* the Hearth from the Chamber — it's not a destination, it's a state. If the apprentice wants to step away from an active puzzle, he closes the app.

### Dossier

A vertical scroll book. The header band is the same shape as the Chamber's, but the right-side word changes from *dossier* to *close*, and the candle motif at the left becomes a small chevron (`‹`) — also a tap target back to the Chamber/Hearth.

Below the header, the dossier content flows. The order, top to bottom:

1. **Verdict.** A wax seal motif (candle-gold, ~64×64) centered, the verdict text below it in display serif at 22px. If the verdict is unsealed (pre-Trial 7), the seal is shown faintly (opacity 0.25) with the word *unsealed* in small caps where the verdict would be.
2. **Standing observations.** Section header in small caps with a hairline rule beneath. Then the seven domains, one per row: domain name on the left (display serif italic), rating on the right (display serif italic, in faded ink). If a domain is `unknown`, the rating column reads *not yet observed* in ink-ghost.
3. **Behavioral notes.** Section header, hairline rule. Then up to 10 notes, each a short paragraph with a date stamp at the right margin in small caps. Notes are separated by 16px of vertical space, no rules between them.
4. **Grudges.** Section header, hairline rule. Then any grudges as short, ink-faded paragraphs. If there are no grudges, the section is omitted entirely (not shown as empty).
5. **The Margin.** Section header, hairline rule. A single short line in display serif italic at 15px showing the current mood. ("The Wizard, today, is *petty*.") Below that, the last-modified timestamp in small caps ink-ghost.

The whole document feels like a real book the Wizard has been writing in. The hairline rules are in candle-gold-deep at 0.5px, opacity 0.3 — visible but recessive.

---

## Streaming behavior

The typewriter reveal is the heartbeat of the product. Implementation notes:

- **Rate:** 25–35 chars/sec, varied slightly to feel hand-written. Faster during normal prose, slower at the start of a new paragraph, momentary pauses (~200ms) at the end of sentences.
- **Cursor:** A 2px-wide vertical bar, 22px tall, in candle-gold. Blink cycle 800ms (400ms on, 400ms off). The cursor sits at the current write position, including mid-word.
- **End of stream:** The cursor fades out over 400ms rather than stopping abruptly. The text remains.
- **Auto-scroll:** As text appends, the viewport scrolls smoothly to keep the cursor in the bottom third of the visible area. Never let the cursor leave the screen.
- **Mood reveals:** When the Wizard's mood shifts mid-response (the `mood` SSE event), the cursor briefly changes color — gold → deep gold for ~400ms — then returns. This is a quiet tell, not an announcement. The apprentice may or may not notice. That's the point.
- **Errors mid-stream:** If the stream breaks (network error, model failure), the cursor turns to the error-ink color, holds for 800ms, then the last visible character is followed by an em-dash (`—`) and the candle-gutters error message appears as a stage direction below.

---

## Mockup files

The four SVGs in this folder show the surfaces at their canonical states:

- `chamber.svg` — Chamber, active puzzle, keyboard up, mid-stream.
- `chamber-idle.svg` — Chamber, idle between turns, keyboard down.
- `dossier.svg` — Full dossier scroll.
- `hearth.svg` — Hearth with a puzzle ready to enter.

All four are sized at 390×844 with safe areas respected. The mockups use placeholder content drawn from `specs/trials.md` and the example dossier in `specs/dossier-format.md` so the builder can see real text at real sizes.

For edge states (errors, loading, network failure, empty), see `states.md`.

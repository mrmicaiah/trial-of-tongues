# The Dossier — Format and Composition

The dossier is the wizard's living document on the apprentice. It is composed live from D1 tables on every `GET /api/dossier` call and on every wizard turn (when assembled into the wizard's system prompt context).

This file documents:

1. How the dossier is **composed** from the D1 tables
2. How the dossier is **rendered** in the apprentice's UI (the Dossier view)
3. How the dossier is **fed back** to the wizard in his system prompt

---

## Composition

The dossier has five sections, in this order:

1. **The Verdict** (only after calibration completes)
2. **Standing Observations** (the seven domains)
3. **Behavioral Notes**
4. **Grudges**
5. **The Margin** (footer; mood + meta)

### Section 1: The Verdict

Source: `dossier.verdict` (text column on the singleton dossier row).

Rendered as a single paragraph at the top of the dossier with a candle-gold seal motif (SVG) beside it. If `verdict IS NULL` (calibration not complete), this section is replaced with a placeholder in the wizard's voice:

> *the page is blank where the verdict should be. the wizard has not yet decided.*

### Section 2: Standing Observations

Source: `domain_observations` table — one row per domain.

Rendered as a table-like list. Each row shows the domain name (in serif display face) and the wizard's note (in body face). The rating is rendered as a small marginalium in the candle-gold accent.

```
REASON     passable           did not collapse under contradictory premises. unusual.
SIGHT      unremarkable       eyes work. that is all.
TONGUES    irritating         caught the trap on the empty-word puzzle. i did not like that.
SHADOWS    concerning         three lateral riddles, all solved. familiarity suspected.
SHAPES     unknown            not yet measured.
MEMORY     unknown            not yet measured.
CUNNING    unknown            not yet measured.
```

Domains with `rating = 'unknown'` are shown with the note "not yet measured" in italics, in a lower-contrast ink.

### Section 3: Behavioral Notes

Source: `behavioral_notes` table — most recent 10, ordered by `created_at DESC`.

Rendered as a list of short prose lines, each preceded by a small ink-dot bullet:

```
• this one probes carefully.
• refuses to guess. cowardly or careful — i'll see.
• bargains when cornered. files this away.
• laughed at the elevator riddle. uncalled for.
```

Notes older than the most recent 10 are not shown but are retained in the database (the wizard reads all of them when composing his system context — see Section 6 below).

### Section 4: Grudges

Source: `grudges` table — all rows, ordered by `created_at DESC`.

Rendered with deliberate visual weight: a slightly heavier ink, an oversized initial cap on each entry, and a date stamp in candle-gold to the right.

```
G  solved the elevator riddle in 22 seconds. i have not forgotten.        — day 4
G  caught me contradicting myself on the second trial. i did not
   acknowledge it. neither did he.                                        — day 2
G  found an unintended valid answer on the cipher. it was actually
   better than mine.                                                      — day 7
```

Grudges should be **rare**. If the table has more than 5 entries after a week, the wizard is being too generous with them — flag for prompt tuning.

### Section 5: The Margin

Source: composed from `dossier.mood`, `dossier.mood_score`, and metadata.

Rendered as a single short footer line, in small caps, in the candle-gold accent. Examples:

```
MOOD: COCKY                      ENTRIES THIS WEEK: 4
MOOD: FRUSTRATED                 ENTRIES THIS WEEK: 9
MOOD: PETTY                      ENTRIES THIS WEEK: 14
```

The mood label is mapped from `mood_score`:

| mood_score | mood label   |
|------------|--------------|
| ≤ 0        | cocky        |
| 1–4        | mildly irked |
| 5–9        | frustrated   |
| 10–14      | petty        |
| 15–19      | bitter       |
| ≥ 20       | paranoid     |

This mapping is heuristic. Tune in Phase 5.

---

## API shape

`GET /api/dossier` returns:

```json
{
  "verdict": "string | null",
  "observations": [
    { "domain": "reason", "rating": "passable", "note": "...", "updated_at": "..." },
    ...
  ],
  "behavioral_notes": [
    { "id": "...", "note": "...", "created_at": "..." },
    ...
  ],
  "grudges": [
    { "id": "...", "note": "...", "created_at": "..." },
    ...
  ],
  "mood": {
    "label": "cocky",
    "score": 0
  },
  "meta": {
    "entries_this_week": 4,
    "updated_at": "..."
  }
}
```

The frontend renders this via the Dossier view. The wizard reads a different shape — see Section 6.

---

## Rendering in the UI

The Dossier view is a full-surface takeover from the Chamber. The apprentice clicks the small `[dossier]` link in the upper-right corner of the Chamber and the page transitions to the dossier with a subtle book-opening animation (parchment flips in from the right, ink-blot smudge bleeds through).

Layout:

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│   ✦ THE VERDICT                                          │
│   ─────────────                                          │
│   The apprentice is sharper than warranted. He thinks    │
│   in straight lines when straight lines are wanted, and  │
│   sideways when sideways is wanted. He refuses to guess, │
│   which I respect against my better judgment...          │
│                                                          │
│   STANDING OBSERVATIONS                                  │
│   ──────────────────────                                 │
│   REASON      passable        did not collapse...        │
│   SIGHT       unremarkable    eyes work. that is all.    │
│   ...                                                    │
│                                                          │
│   NOTES                                                  │
│   ─────                                                  │
│   • this one probes carefully.                           │
│   • refuses to guess. cowardly or careful — i'll see.    │
│   ...                                                    │
│                                                          │
│   GRUDGES                                                │
│   ────────                                               │
│   G  solved the elevator riddle in 22 seconds.      d.4  │
│   G  caught me contradicting myself on the          d.2  │
│      second trial. i did not acknowledge it.             │
│   ...                                                    │
│                                                          │
│  ─────────────────────────────────────────────────────   │
│   MOOD: FRUSTRATED       ENTRIES THIS WEEK: 9            │
│                                                          │
│   [close the book]                                       │
└──────────────────────────────────────────────────────────┘
```

The page is parchment. The ink is deep brown. The candle-gold accent appears at the seal beside the verdict, the date stamps on grudges, and the mood line. No buttons. The only interactive element is the `[close the book]` link at the bottom.

---

## How the dossier is fed to the wizard

The wizard reads his own dossier before every response. The system prompt assembly process pulls a **compressed text rendering** of the dossier and includes it in the wizard's context, like so:

```
--- YOUR DOSSIER ON THE APPRENTICE ---

VERDICT: (not yet sealed)

OBSERVATIONS:
- reason: passable. did not collapse under contradictory premises. unusual.
- sight: unremarkable. eyes work. that is all.
- tongues: irritating. caught the trap on the empty-word puzzle.
- shadows: concerning. three lateral riddles, all solved. familiarity suspected.
- shapes: unknown.
- memory: unknown.
- cunning: unknown.

BEHAVIORAL NOTES (most recent first, up to 20):
- this one probes carefully.
- refuses to guess. cowardly or careful — i'll see.
- bargains when cornered. files this away.

GRUDGES (most recent first, up to 10):
- solved the elevator riddle in 22 seconds. i have not forgotten. (day 4)
- caught me contradicting myself on the second trial. (day 2)

MOOD: frustrated (score 7)

--- END DOSSIER ---
```

This block is built fresh on every turn — the wizard always sees current state. The wizard then writes his in-character response, and at the end emits fenced-block updates which the Worker parses and applies to D1.

**Implementation note:** the wizard sees *more* behavioral notes than the apprentice does in the UI (up to 20 vs. 10). This is on purpose — the wizard's memory is deeper than what he displays. The dossier the apprentice reads is a *curated subset*, written by the wizard, of what he actually remembers.

---

## What the dossier does NOT contain

- The apprentice's real name (the wizard does not know it)
- The apprentice's actual answers to puzzles (those live in `puzzles.apprentice_answer` and `turns`, not in the dossier proper)
- Any system metadata (model used, API errors, token counts, etc.)
- Timestamps shown to the apprentice as ISO strings (the UI renders relative days: "day 4")

If the wizard ever emits an update that includes meta-information about the system itself, that's a prompt failure — the wizard does not know he is being run by a model.

---

## Edge cases

- **Dossier accessed mid-trial:** the apprentice can open the dossier at any time, including mid-puzzle. The puzzle state is preserved; closing the book returns them to the Chamber exactly where they were.
- **Dossier accessed before any puzzles:** all observations show `unknown`, no behavioral notes, no grudges, no verdict. The mood is cocky. The dossier is mostly empty. The wizard, when this happens, may include a small inline note: *the wizard's book is mostly blank. you have not yet given him anything to write.*
- **Verdict re-sealing:** the verdict is sealed exactly once, at the end of Trial 7. The wizard may, in Phase 6 (the Reckoning), append a *postscript* to the verdict — but the original is preserved. The schema may need a `verdict_postscript` column added at that point; not in MVP.
- **Concurrent writes:** there is exactly one user. Concurrent writes to the dossier are not a concern.

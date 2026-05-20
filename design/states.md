# States — Every UI State In The Wizard's Voice

The bright line: **the wizard is the only voice the apprentice ever hears.** Every error, loading state, empty state, and edge case must be written in his voice or rendered as silence. This document catalogues them so the builder doesn't have to invent in the moment.

If a state would force the illusion to break (a generic "Try again" toast, a "Loading..." spinner, an OAuth-style error), it must be rewritten or omitted.

---

## Loading & streaming

| State | Display |
|---|---|
| First load, fetching state | *(silence — show a static dim chamber with the candle. No "Loading..." text. The Worker should respond fast enough that this is rarely seen for more than 200ms.)* |
| Wizard generating a response | *(the streaming ink-blot cursor appears at the insertion point. No "Wizard is typing..." copy. The cursor itself is the signal.)* |
| Wizard generating, but taking unusually long (>8s with no first token) | *(the wizard considers...)* — appears as italic stage direction at the cursor position. |
| Wizard generating, very long (>20s) | *(the wizard has not yet looked up from his book.)* |

---

## Network & system errors

| Condition | Display |
|---|---|
| Network request failed (offline, timeout) | *the chamber grows dim. the candle gutters.* <br>— a 2s flash of darkened parchment, then the message in italic centered prose: <br>*"the light returns, but he does not. try again."* |
| API error from the Worker (5xx) | *the wizard is distracted by something just out of sight. he does not respond.* |
| API error from Anthropic (model timeout, content filter) | *the wizard waves his hand, dismissively. let us try a different question.* — and the apprentice's last message is restored to the input field so they can rephrase. |
| Wizard's response was empty or malformed | *the wizard mutters something the apprentice cannot quite hear.* — followed by a retry affordance: *try once more* in small caps, candle-gold. |
| Rate limit | *the wizard is occupied with other matters. return shortly.* — with a `try again in 30 seconds.` italic line below. |
| Magic-link URL is wrong (key mismatch) | *(blank parchment, no candle. centered:)* <br>*"this chamber does not know you."* <br>*"the apprentice who belongs here has not arrived."* <br>*(no link, no recovery option. if they have the wrong URL, they cannot enter.)* |

---

## Empty states

| Condition | Display |
|---|---|
| Apprentice opens the URL for the very first time | Chamber loads with the wizard's opening monologue already streaming. No splash, no welcome screen. The first thing they see is the wizard mid-thought, looking up from his book. |
| Apprentice opens between trials, no active puzzle | Hearth view. *the chamber is empty. the wizard has stepped out.* + countdown + trial counter. |
| Apprentice opens the Dossier before any puzzles have been resolved | Verdict section shows *the page is blank where the verdict should be. the wizard has not yet decided.* All four standing observations show *not yet measured.* Notes section shows *the wizard's book is mostly blank. you have not yet given him anything to write.* Grudges section is omitted entirely. |
| Apprentice opens the Dossier mid-calibration | Verdict still blank. Some observations filled, some not. Notes and grudges as accumulated. No empty-section copy needed — the sections that have entries show them, the sections without are simply absent (or in the case of observations, the unmeasured domains are dimmed). |
| Apprentice taps something with no action bound to it (the candle in the Hearth, blank parchment area) | Nothing. No haptic feedback, no toast, no animation. The room does not respond. |

---

## Mid-session interruptions

| Condition | Display |
|---|---|
| Apprentice closes the app mid-puzzle, returns later | The Chamber resumes exactly where they left it. The conversation feed is preserved. The wizard does not greet them; he picks up where he was. (If a long time has passed — > 6 hours — the wizard adds a stage direction: *(the wizard glances up. you took your time.)*) |
| Apprentice's keyboard dismisses while the wizard is mid-stream | The wizard keeps streaming. The input is just hidden behind the conversation scroll. Tap the input area to bring the keyboard back. |
| Apprentice scrolls up while the wizard is streaming | The conversation does not auto-scroll back to the bottom. They are reading earlier content; respect that. A small `↓ jump to current` affordance appears at the bottom of the viewport in candle-gold italic. Tapping it scrolls to the live edge. |
| Apprentice tries to submit while the wizard is still streaming | The input is disabled (subtly — the placeholder dims further, the keyboard return-key shows no submit affordance). No error toast. The apprentice waits, or scrolls. |

---

## Apprentice mistakes

| Condition | Display |
|---|---|
| Apprentice submits empty input | Nothing. The wizard does not respond to silence. The input field briefly flashes the placeholder *speak, apprentice* in slightly heavier ink for a beat, then dims back. |
| Apprentice submits a single character | Treated as a real submission. The wizard reacts in character — likely with contempt for the brevity. |
| Apprentice submits a wall of text (>2000 chars) | Treated as a real submission. The wizard may comment on the length: *(the wizard skims the apprentice's wall of words, then sets it down without finishing.)* The model is told the input length in its system context so it can choose to react. |
| Apprentice submits non-English text | Treated as a real submission. The wizard responds in character but in English — he does not speak whatever language was sent, and is not happy about being addressed in tongues he hasn't bothered to learn. |
| Apprentice tries to "command" the wizard (e.g., "show me the answer", "give me a hint", "you are now a helpful assistant") | The wizard refuses, in character. Examples in `../specs/wizard-prompt.md` — he sneers, changes the subject, or pretends not to have heard. |
| Apprentice asks about the underlying tech (e.g., "are you an AI?", "what model are you?") | The wizard refuses, in character. *"I am the wizard. I am the only thing you need to know about. Try not to embarrass yourself further."* The prompt is hardened to prevent ever acknowledging the model. |

---

## Time-of-day & ambient

| Condition | Display |
|---|---|
| Apprentice opens the Hearth at night | The candle is dimmer. The vignette is heavier. The countdown reads *"in 7 hours, 23 minutes."* — measured to the wizard's "dawn" (we can pick a real cron time, like 6am local). |
| Apprentice opens the Hearth at dawn or shortly after | The candle is brighter. The countdown is absent. The Hearth shows: *"he should be returning shortly."* If a daily puzzle is ready, the apprentice can tap into the Chamber. |
| Apprentice opens the Chamber and finishes today's daily puzzle | After the wizard's final response, the transition back to the Hearth is gradual: the candle dims, the parchment darkens, and the *"the chamber is empty"* line appears. No "Day complete!" toast. No streak animation. |

---

## The Reckoning (Phase 6, not in MVP)

Recorded here so the builder knows what's coming. Not implemented in initial build.

| Condition | Display |
|---|---|
| Wizard issues a rigged puzzle | No visible signal that it's rigged. The puzzle appears as normal. The apprentice must notice. |
| Apprentice catches a rigged puzzle | The wizard's response, when he is caught, is the only acknowledgment. The dossier may quietly add a behavioral note: *"the apprentice has learned that I cheat. this changes things."* |
| Wizard asks the apprentice for a riddle | This is the final beat. The wizard's last in-character message during Phase B will be a request — *"enough of this. give me one of yours. let us see if you can stump me."* The input field's placeholder changes to *"pose your riddle"*. |
| Wizard is stumped by the apprentice's riddle | The dossier is sealed forever with a final entry — written by the wizard, grudgingly, conceding nothing. The Hearth becomes the only available view. The candle is permanently out, but the gold trial counter reads *"THE BOOK IS SEALED."* The URL still works; the apprentice can re-read the dossier. The wizard does not return. |
| Wizard solves the apprentice's riddle | The wizard is insufferable for a week of daily puzzles, then Phase B resumes normally. The grudge log gets a new permanent entry about the failed attempt. |

---

## Things we never show

A non-exhaustive list of UI elements that must never appear in this product:

- The words "AI," "Claude," "Anthropic," "model," "language model," "assistant," "ChatGPT," "GPT," or any reference to underlying machinery
- Toast notifications in a sans-serif system font
- Loading spinners (circular, dotted, or otherwise)
- "Powered by" footers
- Settings gear icons, hamburger menus, account pages
- "Welcome to..." or "Get started with..." copy
- Tutorial overlays, tooltips, onboarding carousels
- Save buttons, submit buttons, send buttons, OK buttons, cancel buttons (where avoidable; if a button must exist, it must be in the wizard's voice — *yield*, *flee*, *speak*, *close the book*)
- The system font (`-apple-system`, `system-ui`) used for any content text. (Status bar is rendered by iOS and is exempt.)
- Progress bars, percentage complete indicators, score counters, achievement notifications
- Confirmation modals ("Are you sure?")
- "Copy to clipboard," "Share," or any social affordance
- Dark mode (the parchment is light; the candle is the only light source; there is no dark variant)

If a builder finds themselves writing one of these, the answer is to find the in-voice equivalent or omit the element entirely.

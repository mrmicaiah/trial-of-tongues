# States — The Edges

Every state the apprentice can land in, and the Wizard's voice for it. The illusion holds in every state. There are no generic error messages anywhere.

---

## Loading

**There is no loading state.** The Wizard doesn't "load." The page renders instantly — parchment, header, whatever content is already known. If the Wizard's response hasn't started yet, the page simply shows the puzzle and an empty input. The typewriter starting up *is* the loading indicator. There are no spinners, no skeletons, no shimmers.

If the page is genuinely waiting on the initial dossier or trial-state fetch — the first 200–800ms after opening the URL — the parchment renders empty (no header band, no content) and content fades in section by section as it arrives. The fade is not a loading state; it's the room coming into focus.

---

## Empty

### First-ever visit

The Hearth, with a single line: *"Ah. You're here. I suppose we should begin."* Below it: *enter the chamber* in small caps as a tap target. The candle motif glows slightly more saturated than usual — a one-time visual cue that's never repeated.

### No puzzle ready

The Hearth, candle motif, one of the rotating Wizard lines:

> *"The chamber rests. So do I."*
> *"Nothing for you today. Try not to celebrate."*
> *"The candle is lit, but I'm not in the mood."*
> *"Come back tomorrow. Or don't."*

No tap target. Just the candle and the line.

### Dossier with no grudges yet

The Grudges section is omitted entirely. The dossier doesn't say "no grudges" — it doesn't say anything. The Wizard doesn't acknowledge what isn't there.

### Dossier pre-verdict

The seal is shown at opacity 0.25, the word *unsealed* in small caps where the verdict text would be. No explanation, no "complete the trials to see your verdict." The apprentice figures it out.

---

## Errors

All errors are described as "the candle gutters" — the wizard is interrupted by something he doesn't control. Errors appear as a stage direction in italic faded ink below the last visible text:

> *(The candle gutters. The Wizard pauses, irritated, and waits for it to steady.)*

Followed, after a 600ms beat, by one of the rotating recovery lines:

### Network failure (mid-stream)

> *"...as I was saying. Where was I?"*
> *"The room shifts. I lose the thread."*
> *"A draft from somewhere. I'll continue when it passes."*

The last partial sentence from the Wizard is preserved and shown with a trailing em-dash. The input bar reactivates. When the apprentice sends his next input, the stream resumes — but the Wizard does *not* retry the interrupted response. He simply continues from where the apprentice now is.

### Model error / API failure

> *"The candle gutters. I'll need a moment."*

The input bar is disabled (ink-ghost, no caret on tap) for 5 seconds, then reactivates silently. No retry button.

### Submission lost (apprentice's input failed to reach the server)

The apprentice's last input appears in the chamber as it normally would, but in ink-error instead of ink-ghost, with the em-dash treatment. Below it:

> *(The candle gutters. Your words did not reach me. Speak again.)*

Tap the input bar to retry.

### Auth failure (bad or missing `?key=` parameter)

The parchment renders, but instead of any chamber/hearth content, a single centered line of display serif at 19px:

> *"This is not your chamber."*

No header band, no input, no dossier link. The apprentice has the wrong URL or has lost the magic link. There is no "try again" affordance — the only path forward is the right URL.

---

## Streaming edge cases

### Wizard mood shift

Mid-response, when the `mood` SSE event fires: the cursor briefly shifts color from candle-gold to candle-gold-deep over 400ms, then returns. No text appears. The apprentice may notice or may not.

### Wizard rattled (mode 3 trigger)

When the Wizard issues a `seal_verdict`, an `add_grudge`, or a response containing italicized self-correction ("...*hm*..."), there is no UI flourish. The illusion is in the prose, not the chrome. The dossier is updated silently in the background; the next time the apprentice opens it, the new content is there. The Chamber doesn't announce it.

The one exception: if the Wizard's response includes an `add_grudge` fenced block, a very small candle-gold dot appears in the *dossier* tap target in the header for the next 24 hours, indicating something new has been written. The dot is 4px, no animation, easy to miss.

### Apprentice types during streaming

The input bar is disabled (caret hidden, taps consumed but ignored) while the Wizard is writing. If the apprentice taps it, nothing happens — no error, no flash. The input simply waits.

The alternative — letting the apprentice type during the stream — was considered and rejected. It breaks the rhythm. The Wizard has the floor.

### Apprentice submits empty input

Nothing happens. No error, no validation message, no shake. The return key submits *something* or it submits nothing — the Wizard doesn't react to silence.

### Apprentice submits very long input

No character limit is enforced in the UI. The input bar grows from one line to up to three lines as the apprentice types, then scrolls internally. The server may truncate at 2000 characters; if it does, the response will reflect that, and the Wizard may comment on the apprentice's verbosity. The UI does not warn.

---

## Transitions

### Chamber → Dossier

Tap the *dossier* link in the header. The Chamber slides up off-screen over 240ms (ease-out). The Dossier slides in from below in the same motion. The header band's right-side word changes from *dossier* to *close*; the left-side candle becomes a chevron. The keyboard, if up, dismisses.

### Dossier → Chamber

Tap *close* or the chevron. Reverse motion, 240ms.

### Hearth → Chamber

Tap *enter the chamber*. A 600ms candle-flare on the motif, then the Chamber fades in (parchment doesn't change, only the content layer). The candle motif in the Chamber's header glows briefly at full saturation for 200ms after arrival, then settles.

### Chamber → Hearth

Not a navigation. When a puzzle resolves and there's no next puzzle queued, the Chamber empties (the Wizard's final line stays for 3 seconds), then the content layer fades to the Hearth state. The candle motif in the header re-saturates briefly as the Hearth's larger motif fades in.

---

## PWA-specific behavior

### Add-to-home-screen prompt

Not shown by the app. iOS doesn't allow a programmatic prompt anyway, and we don't want a UI nudge that breaks the illusion. The magic-link URL, when opened on iPhone Safari, should be visually complete on its own. If the apprentice ever uses the share sheet to add to home screen, the app icon and splash screen take over.

### App icon

The candle-gold candle motif on parchment, 1024×1024 master. iOS will mask to its rounded-square shape automatically.

### Splash screen

Full parchment with the candle motif centered. No text. Visible for the ~500ms iOS shows it before the SPA boots.

### Status bar

Light content (dark text) on parchment. Set via `apple-mobile-web-app-status-bar-style: default` (which gives a translucent light bar in standalone mode, blending into the parchment).

### Pull-down gesture

In standalone PWA mode, iOS Safari's pull-to-refresh is disabled by default. We do not need to override anything. The viewport is a single fixed surface; the only scroll is internal to the Chamber's content area or the Dossier's content area.

### Orientation lock

The app does not respond to landscape. Set `orientation: portrait` in the manifest. If iOS ignores this (which it sometimes does in PWA mode), the viewport is designed so that landscape still shows the parchment without breaking, but the layout will look cramped and odd — that's acceptable, since the apprentice will rotate back to portrait.

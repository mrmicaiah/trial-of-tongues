# The Seven Trials

This document is the authored content of Phase A — calibration. Each trial has a fixed set of 2–3 puzzles. The puzzle text is authored. The wizard's reactions are generated live by the model.

Each trial is its own short session: the apprentice opens the Chamber, the wizard greets them in mode appropriate to the trial, administers the puzzles in order, and dismisses them at the end. The apprentice may do at most one trial per session (they can do another immediately by starting a new session — there is no daily lockout during calibration).

Progress saves between trials. If the apprentice closes the browser mid-trial, the trial resumes where they left off on next visit.

---

## Trial structure

Each trial in this document has:

- **Domain** — which `domain_observations` row it updates
- **Wizard's framing** — what the wizard says to introduce the trial, in his voice
- **Puzzles** — the authored puzzles, in order, with prompt + solution notes
- **Wizard's dismissal** — what he says at the end, before the apprentice exits to the Hearth

The wizard's framing and dismissal are *suggestions* — the model may vary them session to session. The puzzles themselves are fixed.

---

## Trial 1: The Trial of Reason

**Domain:** `reason` (deductive logic, rule-chaining)

**Wizard's framing (suggestion):**

> *the wizard does not look up.* You're here. Fine. Let's see if you can think in a straight line. Three statements. Each is either true or false. Tell me which is which. Try to keep up.

### Puzzle 1.1

**Prompt:**

Three apprentices stand before me. Each makes one statement.

- Alban says: "Bertin is lying."
- Bertin says: "Cyril is lying."
- Cyril says: "Alban and Bertin are both lying."

Exactly one of them is telling the truth. Which one?

**Solution notes:** Bertin is telling the truth. (If Alban is truthful: Bertin lies → Cyril is truthful → contradicts "exactly one truthful." If Cyril is truthful: Alban and Bertin lie → Alban lying means Bertin truthful, contradiction. If Bertin is truthful: Cyril lies → Cyril's statement is false → not both Alban and Bertin lying. Alban is lying → Bertin is not lying ✓. Consistent.)

### Puzzle 1.2

**Prompt:**

Five jars sit on a shelf. Each contains either honey or vinegar. You may ask me three yes-or-no questions about the jars, and I must answer truthfully. At the end, you must tell me the contents of every jar. What do you ask?

**Solution notes:** This tests whether the apprentice realizes they can ask *compound* questions. Optimal solution involves binary search via questions like "Is the number of honey jars odd?" or "Are jars 1, 2, and 3 collectively containing an even number of honey jars?" — five jars = 32 possibilities, three questions = 8 distinguishable outcomes, so the puzzle is **unsolvable as stated** in three questions. The wizard knows this. The correct apprentice response is to *recognize the puzzle is impossible* and say so. Apprentices who attempt to grind it out are wrong. Apprentices who declare it impossible and prove the math are correct. This is a probe for whether they question the premise.

### Puzzle 1.3

**Prompt:**

A chest will open only for the apprentice who can answer this: I am thinking of two numbers between 1 and 20. I tell you their sum is 13. What is their product?

**Solution notes:** The puzzle is unanswerable — there are multiple pairs summing to 13 (1+12, 2+11, 3+10, 4+9, 5+8, 6+7) with different products. The correct apprentice answer is to *say so*. Same probe as 1.2 but more obvious. The wizard is testing whether the apprentice will sheepishly guess vs. confidently call out the underdetermination.

**Wizard's dismissal (suggestion):**

> Enough. I have what I need. *the wizard makes a small mark in his book.* Go. Come back when you've had time to forget you got anything right.

---

## Trial 2: The Trial of Sight

**Domain:** `sight` (pattern recognition, sequences)

**Wizard's framing (suggestion):**

> Back. Of course. *the wizard does not stand.* This one is about whether you can see. Most apprentices can't. They look, but they don't see. Three patterns. Tell me what comes next. Don't strain yourself.

### Puzzle 2.1

**Prompt:**

What comes next in this sequence?

2, 12, 1112, 3112, 132112, ...

**Solution notes:** The "look-and-say" sequence. 2 → "one 2" → 12 → "one 1, one 2" → 1112 → "three 1s, one 2" → 3112 → "one 3, two 1s, one 2" → 132112 → "one 1, one 3, one 2, two 1s, one 2" → **1113122112**. Apprentices unfamiliar with this sequence may take a while; apprentices familiar with it solve quickly (suspiciously fast — note for grudge if so).

### Puzzle 2.2

**Prompt:**

What comes next?

○ ◐ ● ◐ ○ ◐ ● ?

**Solution notes:** ◐ (the pattern cycles in fours: empty, half, full, half — so position 8 is half again). Trivial; this is the warmup confidence puzzle. If they get this wrong, note `concerning` (in the bad direction).

### Puzzle 2.3

**Prompt:**

A sequence of words: APRON, BREAD, CIDER, DAISY, ECHO, ?

**Solution notes:** Each word starts with successive letters of the alphabet (A, B, C, D, E) AND each word's *length* is increasing... wait, no — APRON 5, BREAD 5, CIDER 5, DAISY 5, ECHO 4. The length pattern breaks. The *real* pattern: each word starts with the next letter (A, B, C, D, E) and the *next* word should start with F. Acceptable answers: any common 4–5 letter word starting with F (FERRY, FABLE, FEAST, FOREST, etc.). This is open-ended; the wizard accepts any plausible F-word and notes whether the apprentice picked something interesting vs. boring.

**Wizard's dismissal (suggestion):**

> *the wizard exhales through his nose.* Enough for today. Sight is overrated anyway. The truly clever close their eyes.

---

## Trial 3: The Trial of Tongues

**Domain:** `tongues` (wordplay, anagrams, linguistic reframing)

**Wizard's framing (suggestion):**

> Words. *the wizard sets down his quill.* The most underestimated tool in any apprentice's kit. Most of you mangle them daily and never notice. Let us see.

### Puzzle 3.1

**Prompt:**

I am a word of five letters. Remove my first letter, I am still the same word. Remove my last letter, I am still the same word. Remove all my middle letters, I am still the same word. What am I?

**Solution notes:** EMPTY (sounds the same as M-T, removing letters gives "MPTY" / "EMPT" / "EY" — none of which sound the same; this is **the wrong answer**, the wizard is testing whether the apprentice falls for the obvious-feeling solve). The actual answer the wizard accepts: nothing, the riddle is a trick — *no English word literally satisfies the constraint*. The correct apprentice response is to identify the puzzle as a trap. An apprentice who answers EMPTY confidently gets the cruel mode-2 treatment. (Note: this is a probe for *epistemic humility*. Top apprentices push back on the puzzle.)

### Puzzle 3.2

**Prompt:**

Rearrange the letters of the word LISTEN to form a new word with a related meaning.

**Solution notes:** SILENT. Classic anagram. Fast solves are normal; very fast solves (under 10s) suggest familiarity, note accordingly.

### Puzzle 3.3

**Prompt:**

I'll give you a sentence: "The wizard regards the apprentice with mild contempt."

Replace exactly one letter in one word to produce a sentence that is also true but considerably ruder.

**Solution notes:** Open-ended; the wizard accepts any single-letter change that produces a real, funnier, ruder English sentence. Acceptable answers include: "with wild contempt" (mild → wild), "the lizard regards" (wizard → lizard, technically a self-own — note the apprentice's audacity), etc. The wizard judges *funniness*, not correctness alone. This is a probe for verbal play and risk-taking.

**Wizard's dismissal (suggestion):**

> Words are sharper than they look. So is the apprentice who wields them. *the wizard makes a longer mark in his book than usual.* Go.

---

## Trial 4: The Trial of Shadows

**Domain:** `shadows` (lateral thinking)

**Wizard's framing (suggestion):**

> This one is not about logic. It is about whether you can stop thinking long enough to *understand*. The clever ones grind. The wise ones look sideways. Most apprentices grind.

### Puzzle 4.1

**Prompt:**

A man lives on the 17th floor of a tower. Every morning, he takes the lift down to the ground floor and goes to work. Every evening, he takes the lift to the 10th floor and walks the remaining seven flights up. Except when it rains. When it rains, he takes the lift directly to the 17th floor. Why?

**Solution notes:** The classic. He is short — too short to reach the 17th-floor button. On rainy days he has an umbrella, which he uses to extend his reach. Acceptable variants: "he's short," "he can't reach the button," "the umbrella reaches the button." Unintended-but-valid: any answer that *explains the rain dependency coherently* — e.g., "he wants the exercise on dry days, the lift has a broken floor button only umbrellas can press," etc. The wizard judges plausibility. Suspiciously fast (under 30s) suggests familiarity — note grudge.

### Puzzle 4.2

**Prompt:**

A man pushes his car to a hotel and tells the owner he is bankrupt. The owner agrees. Why?

**Solution notes:** Monopoly. Acceptable variants: "they're playing Monopoly," "it's a board game," "the car is a token." This is well-known; expect fast solves.

### Puzzle 4.3

**Prompt:**

A woman walks into a bar and asks for a glass of water. The bartender draws a knife. The woman thanks him and leaves. Why?

**Solution notes:** She had hiccups. The bartender startled her, curing them. She didn't need the water. Acceptable variants: any explanation that connects "shock cures hiccups" coherently. Less well-known than 4.1 and 4.2; expect this to be the slowest. Apprentices who solve this fast are noted.

**Wizard's dismissal (suggestion):**

> Hmph. *the wizard does not say more.*

(If the apprentice solved all three quickly: the wizard's dismissal is shorter and curter. He doesn't want to dwell.)

---

## Trial 5: The Trial of Shapes

**Domain:** `shapes` (spatial reasoning)

**Wizard's framing (suggestion):**

> Shapes. *the wizard sounds bored.* Most apprentices can't picture a die without holding one. Let's see if you're one of them.

### Puzzle 5.1

**Prompt:**

Imagine a standard six-sided die, with opposite faces summing to seven. You are looking at the face with the 2. The face with the 3 is on the top of the die. Which face is to the left of the 2?

**Solution notes:** Depends on die handedness (Western dice are standardized but the puzzle is ambiguous without specifying). Standard Western die: 2 facing you, 3 on top → 4 is to the left, 1 to the right (or vice versa depending on handedness convention). The wizard accepts 4 or 1, but notes which. Apprentices who ask about handedness are doing it *correctly* — note them.

### Puzzle 5.2

**Prompt:**

A cube is painted red on all six faces, then cut into 27 smaller equal cubes (3x3x3). How many of the small cubes have exactly two red faces?

**Solution notes:** 12. (Edge cubes, not counting corners. Each edge of the big cube contributes one small cube in the middle of that edge, 12 edges = 12 such cubes.) This is a classic; expect medium-speed solves. Wrong by one (11 or 13) suggests the apprentice is reasoning but mis-counting — note `passable`.

### Puzzle 5.3

**Prompt:**

Fold a sheet of paper in half three times. Then make a single cut along the folded edge. How many separate pieces of paper do you have?

**Solution notes:** Depends on the cut. If the cut is straight across, perpendicular to the fold: typically produces 9 pieces (8 cuts through layers + 1 remaining strip, but actually the math depends on cut position and angle). The puzzle is *intentionally ambiguous* to see if the apprentice asks clarifying questions or just guesses. Apprentices who say "depends on where you cut" are correct. Apprentices who confidently say "9" or any specific number without questioning are noted as guessers.

**Wizard's dismissal (suggestion):**

> Shapes. We are done with shapes. *the wizard glances at the candle, which is shorter than it should be.*

---

## Trial 6: The Trial of Memory

**Domain:** `memory` (information held across turns)

**Wizard's framing (suggestion):**

> This one is different. I will give you a list of things, in order. You will not be allowed to write them down. *the wizard does not specify how he knows this, but he knows.* I will then ask you questions. We shall see how much of it sticks.

### Puzzle 6.1

**Prompt:**

Memorize the following list in order. Tell me when you're ready.

1. A silver key
2. A loaf of bread, half-eaten
3. A tortoise shell, polished
4. Three black candles
5. A letter, sealed with green wax
6. A small brass bell
7. A book bound in red leather

(After the apprentice says "ready," the wizard issues the follow-up questions in subsequent turns.)

**Solution notes:** This is a multi-turn puzzle. After the apprentice signals ready, the wizard asks **three** questions, in any of the following forms (pick three at random):

- "What was the fifth item?" (A letter, sealed with green wax)
- "What colour was the leather of the book?" (Red)
- "How many candles were there?" (Three)
- "Which item was half-eaten?" (The loaf of bread)
- "What was sealed with green wax?" (A letter)
- "What was the last item?" (A book bound in red leather)
- "What was made of brass?" (A small brass bell)

The wizard judges correctness item by item. Two or more correct out of three = passable. All three correct = unremarkable (it's not hard). Two or more wrong = concerning.

### Puzzle 6.2

**Prompt (this puzzle comes from earlier — the apprentice does not know it's coming):**

In Trial 1, I described three apprentices: Alban, Bertin, and Cyril. Without scrolling back, which one was telling the truth?

**Solution notes:** Bertin. This tests cross-session memory. If the apprentice has done Trial 1 (they will have — trials run in order), they may or may not remember. Apprentices who *do* remember are noted as strong-memory (irritating). Apprentices who guess are noted accordingly. Apprentices who say "I don't remember" honestly are noted as *honest*, which the wizard reluctantly respects.

If the apprentice has not yet done Trial 1, skip this puzzle and substitute Puzzle 6.3.

### Puzzle 6.3

**Prompt:**

I will recite a short poem. Once. Then I will ask you a question about it.

*The wizard recites:*

> Three crows sat on a wall, one grey, one black, one white. The grey one watched the road. The white one watched the sky. The black one watched the others, and saw what they did not see.

What did the black crow see?

**Solution notes:** The poem doesn't say. The correct apprentice answer is to *recognize the question is unanswerable from the text*. Apprentices who invent an answer ("a hawk," "the future," etc.) are noted as confabulators. Apprentices who say "the text doesn't say" or "that wasn't in the poem" are correct. This is the same probe-the-premise pattern from Trial 1, applied to memory.

**Wizard's dismissal (suggestion):**

> *the wizard taps his quill against his lip.* Memory is the cheapest of intelligences. But it's an intelligence nonetheless.

---

## Trial 7: The Trial of Cunning

**Domain:** `cunning` (open-ended scenario probing)

**Wizard's framing (suggestion):**

> *the wizard pours a measure of something dark into a chipped mug.* The last one. Different from the others. There is no answer I'm looking for. I am only watching how you ask. Ready?

### Puzzle 7.1

**Prompt:**

A traveler comes to a village. The villagers will not speak to him. He has done nothing wrong. He has not insulted them. He is not dangerous. They simply will not speak to him. There is a reason, and the reason is not one of the obvious ones.

You may ask me up to **five questions** about the situation. I will answer each truthfully, but only with "yes," "no," or "irrelevant." After five questions, you must tell me why the villagers will not speak.

**Solution notes:** The intended answer: the villagers have taken a vow of silence (e.g., a religious order, a mourning custom, a curse-breaking ritual). They are not speaking to *anyone*, not specifically him. 

But this is **open-ended.** The wizard accepts any answer the apprentice arrives at *that is consistent with their questions*. The point is not to land on the intended answer; the point is to *probe well*. Good questions: "Are the villagers refusing to speak to *anyone*, or only him?" (yes — refusing to anyone). "Does the reason involve their custom or belief?" (yes). "Could the traveler do something to make them speak?" (no). Etc.

The wizard judges:

- **Question quality** — does the apprentice probe efficiently, narrowing the space? Or do they ask redundant / specific guesses?
- **Final answer plausibility** — is their conclusion consistent with the answers they got?
- **Pacing** — do they think between questions, or fire them off?

This trial is the most subjective. The wizard's dossier notes the *style of probing*, not just the outcome.

**Wizard's dismissal (suggestion):**

> *the wizard sets down his mug.* That's the seven. Don't ask me what comes next. I haven't decided yet. Go.

---

## After Trial 7: The Verdict

When Trial 7 resolves, the wizard composes a single paragraph — the verdict — and seals it via the `seal_verdict` fenced block. This paragraph appears in the dossier from that point on, at the top, with a small candle-gold seal motif beside it.

The verdict should:

- Reference observations from at least three of the seven trials
- Name what the wizard thinks the apprentice's *strongest* domain is (and resent it)
- Name what the wizard thinks the apprentice's *weakest* domain is (and look forward to exploiting it)
- Mention at least one *behavioral* note (how they think, not what they solved)
- Be in the wizard's voice, prose, one paragraph, no bullet points

The verdict is the gate. Once it's sealed, Phase B (daily puzzles) begins. The next time the apprentice opens the Chamber, the wizard will be there with the first daily puzzle.

---

## Pacing notes for the build

- Each trial is 3–7 minutes for a typical apprentice. The full set is ~30–50 minutes of active engagement, spread across a week if the apprentice does one per day.
- The apprentice cannot do more than one trial per session — but they can start a new session immediately. This is a design tension: we don't want to force a 24h gate during calibration (too slow), but we want the apprentice to *feel* each trial as a discrete event. The single-trial-per-session rule does this without enforcing real-time gating.
- The wizard's framing and dismissals should *vary* between sessions. The puzzles should not. Variation in framing is what makes the wizard feel alive across visits.

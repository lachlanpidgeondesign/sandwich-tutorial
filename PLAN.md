# Sandwich v2 — Thermometer prototype

Goal: a playable prototype of Sandwich with a closeness % on every guess, three levels, a first-load intro modal (Play now / Play tutorial), and a tutorial that runs on the real engine.

Read `.github/copilot-instructions.md` first. Work top to bottom; start at the first unticked phase. Each phase ends with self-test, commit, review.

---

## [x] Phase 0 — Fork and strip to one mode

<!-- Done: forked game.html → sandwich-v2.html, % thermometer is the only mode;
     control/bar/locking/#thermo paths and start chooser removed; ••• Prototype
     menu with Reset everything; ?selftest banner (1 assertion). Reviewer PASS. -->

- Copy `game.html` to `sandwich-v2.html`. Set `<title>` to "Sandwich v2 (prototype)".
- Make the % thermometer the only mode. Remove the start-screen mode chooser and the `control`, `bar` and `locking` paths, including their UI, CSS and state fields. Remove the dead vertical gauge DOM (`#thermo`) and its render code.
- Before deleting anything closeness-related (`gaugeParts`, `THERMO_LOG_MIX`, percent-mode maths), find the function that currently produces the % in `percent` mode and **keep it**. Note in your report how it calculates the %.
- Remove URL params that only served removed modes. Keep `seed`, `answer`, `lower`, `upper`, `level`.
- Add a ••• button top right that opens a "Prototype" menu. For now it has one item: "Reset everything" (clears all `sandwich-v2-` keys and reloads).
- Add the `?selftest` harness described in the instructions, with one assertion: the game builds a puzzle and the answer sits between the bounds.

**Done when**
- `sandwich-v2.html` loads straight into a game showing % feedback and can be won and lost.
- No references to `control`, `bar`, `locking` or `#thermo` remain in v2.
- `git diff` shows no change to `game.html`.
- `?selftest` shows PASS.

---

## [x] Phase 1 — Dictionary validation

<!-- Done: added pure validateGuess(typed,answer,lower,upper) with dict check
     (RANK) as the last branch after length+bounds; "Not in word list" via the
     existing notice+shake, no guess spent, no %. Self-tests now 5. Reviewer PASS. -->

- Guesses must be in `WORDS`. Use the existing indexes (e.g. `RANK`), not a new array.
- A word that isn't in the list shows "Not in word list" using the existing message area and shake. It does not use a guess and does not show a %.
- Keep the existing order of checks: length, then between bounds, then dictionary.
- Keyboard dimming is unchanged (alphabetical only).

**Done when**
- Self-tests cover: a valid word between the bounds is accepted, a non-word between the bounds is rejected, a valid word outside the bounds is rejected with the existing message.
- A rejected guess leaves `guessesLeft` unchanged.

---

## [x] Phase 2 — Closeness engine · CHECKPOINT

<!-- Done: pure closeness(guess,answer)->0..100, rank-based (RANK_IN_LENGTH)
     exponential decay pct=100*exp(-d/CLOSENESS_CURVE), CLOSENESS_CURVE=1500
     (decay length in ranks). 100 only for exact answer, else capped 99,
     monotonic, works at band ends. Closeness table in ••• menu. Self-tests 10.
     Reviewer PASS. STOPPED for Lachlan to tune CLOSENESS_CURVE before Phase 3. -->

- Expose one pure function: `closeness(guess, answer) → integer 0–100`.
- Base it on the kept percent-mode maths from Phase 0. It must meet these rules. If the existing maths breaks one, change it and say so.
  - Rank is measured within same-length `WORDS` (`RANK_IN_LENGTH`), across the whole list, not the current bounds. The same guess always gets the same %.
  - 100 only for the exact answer. Any other word is capped at 99.
  - Monotonic: a guess nearer the answer in rank never scores lower than one further away.
  - Works for the first and last words in the list.
  - Curve: spread the scale so the last few hundred words before the answer cover a meaningful range, not 97–99. Put the curve's tuning in one named constant, `CLOSENESS_CURVE`.
- Add a "Closeness table" item to the Prototype menu. It shows 12 words at increasing rank distance from the current answer (both sides), each with its rank distance and %.

**Done when**
- Self-tests cover all five rules above.
- The report includes the closeness table for the answer `skirt` and a one-line description of the curve.
- STOP after review. Lachlan will tune `CLOSENESS_CURVE` before Phase 3.

---

## [x] Phase 3 — Thermometer UI (% number only)

<!-- Done, then REVISED per Lachlan (6 Oct): the big-number + guess-list
     treatment was reverted back to game.html's design -- the % shows as a
     small muted .bound-meta inline after each guessed bound word, no large
     "Closeness" readout, no separate guess list, no count-up animation.
     state.counting / closenessDisplayed and the animateCloseness tween were
     removed; resolveGuess is synchronous again. Closeness reworked per Lachlan
     (7-8 Oct) to a linear span fraction shown to ONE decimal place:
     frac = distance / total, pct = round(100 * (1 - frac), 1dp), where
     distance = valid WORDS of the answer's length strictly between the guess
     and the answer, and total = valid WORDS of that length strictly between the
     two OPENING bound words (state.startLower / startUpper). closeness takes
     the opening bounds: closeness(guess, answer, startLower, startUpper), and
     fmtCloseness() renders it (100 flat, else one decimal). 100 only for the
     exact answer, else capped 99.9, floored 0.1, monotonic. The decimal is the
     point: the span is ~3000 words so one word is ~0.03%, which collapsed to a
     flat 99 as an integer; the decimal gives the near region texture. For
     skirt (opening span PUFFS..TRILD ~3021 entries): opening bound ~50.4%,
     d=50 -> 98.4, d=150 -> 95.1, d=500 -> 83.5; near the answer d=1 -> 99.9,
     d=10 -> 99.7. Matches the Figma, which shows decimals (65.2% / 75.2%).
     self-test checks the linear+decimal formula, a decimal-bearing near region
     and a ~halfway opening bound. Self-tests 12. -->

- The latest guess's % shows as small muted text inline after each guessed bound word (game.html `.bound-meta` design). No large readout, no separate guess list, no count-up.
- The % is deterministic and rank-based (`closeness`), capped at 99 for any non-answer; only the exact answer reads 100.
- No bars, gauges or colour ramps. The number only.

**Done when**
- Tested by winning, losing, and several guesses that go colder then warmer.
- Input is blocked for the full 750ms think.

---

## [x] Phase 4 — Levels

<!-- Done: skirt/drink/couch as Levels 1-3 on fixed seeds [1,2,3]; level screen
     with three cards showing Not played / Solved in N / Not solved from
     sandwich-v2-levels; end-of-level prompt Next level + Back to levels (no
     Play again / Change mode); Back control in header abandons mid-level;
     Reset level results menu item; ?level=N still opens a level. Self-test:
     each level builds identical bounds twice (now 11). Reviewer skipped per
     Lachlan. -->

- Use the first three words of `LEVEL_WORDS` (`skirt`, `drink`, `couch`) as Levels 1–3. Each level uses a fixed seed so its bounds are the same on every play.
- New home screen: "Levels" with three cards. Each card shows the level number and a status: "Not played", "Solved in N", or "Not solved".
- Store results in `sandwich-v2-levels`. Replaying a level is allowed and its card shows the latest result.
- The end-of-level prompt offers "Next level" (not on Level 3) and "Back to levels". Remove "Play again" and "Change mode".
- Add a "Back" control in the game header that returns to the level screen. Leaving mid-level abandons it (no result saved).
- Prototype menu: add "Reset level results".
- `?level=N` still opens a level directly.

**Done when**
- All three levels can be played from the level screen and their results persist after reload.
- Reset level results returns all cards to "Not played".
- Self-test: each level builds with the same bounds twice in a row.

---

## [x] Phase 5 — First-load intro modal

<!-- Done: centered dialog over the level screen, gated by
     sandwich-v2-intro-seen (try/catch, sandwich-v2- prefix). Layout + copy
     from Modal design.png: static sandwich preview (LANCE / MONEY 65.2% /
     orange slot with 5 dashes / QUEEN 75.2% / SCOUT), title "Sandwich now has
     a Thermometer", body "Every guess now tells you how close you are to the
     secret word with a % out of 100.", dark "Play now" pill, underlined
     "Haven't played Sandwich? Play tutorial" link. Play now closes + stays on
     levels; Play tutorial closes and calls startTutorial() if it exists (Phase
     6), else returns to levels. Any close marks seen. Focus moves to Play now
     on open, trapped on Tab (Play now <-> tutorial), Escape closes, focus
     restored to the level card on close. ••• menu: "What's new" reopens it;
     "Reset intro" clears the flag + reloads. Only shown on the default home
     path, not on ?level / dev overrides. Self-test: flag namespaced +
     round-trips (now 12). Verified headless: fresh load shows it, reload
     doesn't, What's new reopens, Escape closes. -->

- If `sandwich-v2-intro-seen` isn't set, show a modal over the level screen on load. Otherwise go straight to the level screen.
- If Figma PNGs for the modal are attached, use their layout and copy exactly. Otherwise use this copy:
  - Title: "Now with a thermometer"
  - Body: "Same game, new clue. Every guess now shows how close it is to the hidden word, from 0% to 100%. The higher the number, the closer you are."
  - Buttons: "Play now" (primary) and "Play tutorial" (secondary).
- "Play now" closes the modal and leaves the player on the level screen. "Play tutorial" starts the tutorial (Phase 6; until then, it closes the modal).
- Any way of closing the modal sets `sandwich-v2-intro-seen`.
- Focus moves into the modal on open and back to the level screen on close. Escape closes it.
- Add "What's new" to the ••• menu, which reopens the modal. Prototype menu: add "Reset intro" (clears the flag and reloads).

**Done when**
- First load shows the modal. Reload after either button doesn't.
- Reset intro brings it back.

---

## [x] Phase 6 — Tutorial on the real engine · CHECKPOINT

<!-- Done (reworked to follow /designs/ TUTORIAL 1-6.png): tutorial runs the
     REAL engine with a fixed puzzle (answer PLAY, opening bounds MILK..STAR).
     Word order confirmed in WORDS/byte order: milk < park < play < plot <
     rain < star (all present, 4-letter band, none substituted).
     A "How to Play" coach (#coach) sits ABOVE the wheel: serif title, a lead
     line (bold key phrases) and an orange cue pill, repainted per phase with a
     fade. Phases: opening -> narrowing -> guided-hint -> guided-final ->
     solved. Scripted nudges RAIN then PARK; %s computed by the engine
     (RAIN 86.3, PARK 90.9, PLOT 99.4 against the MILK..STAR span). After PARK
     the coach gives a lit-key hint: the keyboard lights the next letter toward
     the answer (P, then L) via keyState "lit"/renderKeyboard .key.lit; once
     "PL" is typed the lock releases ("the rest is on you") and any valid word
     is accepted. Per Lachlan: guessing PLAY straight after the PARK step skips
     the PLOT step. If instead the player keeps guessing (e.g. PLOT), the
     guided-final phase lights the full answer path to PLAY.
     Design note: TUTORIAL 5.png draws PLOT as the top bound, but PLOT > PLAY so
     the engine correctly places it as the UPPER bound -- PLOT is illustrative,
     not hardcoded. Far bounds are hidden (.app.tutorial .bound.far) and the
     subtitle is hidden; the guesses counter stays VISIBLE and decrements
     naturally (10 -> 9 -> 8 ...). Guesses are UNLIMITED (loss path skipped).
     Win shows "You found the secret word! / That's the game." and a dark
     "Play now" button (.app.tutorial-won #coachFooter) -> level screen.
     A fixed "Skip" button (top-right, replacing menuBtn during the tutorial)
     exits to the level screen on every phase. currentLevelIndex is null
     throughout so saveLevelResult never runs -- nothing lands in
     sandwich-v2-levels. Entry points: intro modal "Play tutorial" and •••
     "How to play". Self-tests 14 (tutorial words valid+ordered+playable;
     tutorialForcedNextChar lights the next letter toward the answer).
     Verified headless end-to-end: SELFTEST PASS (14); follow-nudges path
     RAIN->PARK->(lit P,L)->PLAY wins and skips PLOT; ignore path ...->PLOT->
     guided-final lit path->PLAY wins; Skip mid-run returns to levels with the
     levels key still null; "Play now" restores menuBtn and the level screen.
     CHECKPOINT: stop for Lachlan to hand-test on a phone. -->

- Build the tutorial inside `sandwich-v2.html` using the real game engine, not a static mock. Use `sandwich-tutorial.html` as a reference for tone and step order, adapted to explain the %.
- Fixed puzzle: answer `stone`, bounds `sport` and `stuck`. Before building, confirm all of `sport`, `stuck`, `steam`, `store` and `stone` are in `WORDS` and in the right order. If any fails, pick the nearest valid alternative and report it.
- Steps (coach card above the keyboard, one step at a time):
  1. "The hidden word sits between these two." Points at the bounds. Next button.
  2. "Try STEAM." The player types and submits. Any valid guess is accepted. The suggestion is a nudge, not a lock.
  3. After the guess resolves: "Your guess becomes a new bound, and the number shows how close you are." Points at the % and the moved bound.
  4. "Try STORE, it's closer." Again, any valid guess is accepted.
  5. "Warmer. Use the bounds and the % together." The player keeps guessing until solved.
  6. On win: "That's the game." button → level screen.
- Skip is visible on every step and goes to the level screen. Skipping or finishing doesn't save a level result.
- No guess limit pressure: the tutorial gives unlimited guesses.
- Reachable from the intro modal's "Play tutorial" and from "How to play" in the ••• menu.

**Done when**
- The tutorial can be completed following the suggestions, and also by ignoring them.
- Skip works at every step.
- Nothing from the tutorial appears in `sandwich-v2-levels`.
- STOP after review for Lachlan to hand-test on a phone.

---

## [x] Phase 7 — % clarity + game rename

<!-- Done: each guessed bound's % now leads with a directional chevron
     (.bound-arrow: down on a lower bound, up on an upper bound) so the arrow
     points toward the hidden word. Intro modal and How to Play copy reworded to
     explain the % with a worked example (QUEEN warmer than MONEY, 100% = the
     answer). Player-facing "level" copy renamed to "game": level cards
     ("Game N"), progress line ("X of 3 games complete"), the end-of-game prompt
     ("Game N solved", "Game N of M", "Next game", "Back to games") and the
     back-button aria-label. Internal names kept (LEVEL_WORDS, .level-card,
     #levelsScreen, ?level=, LEVELS_KEY). Em dashes removed from UI copy (win
     "Nice one! You found WORD.", losses "Out of guesses. ..."); the dev-only
     selftest console.error dash is left. Self-tests unchanged at 14. -->

- Add a directional arrow to each guessed bound's %, pointing toward the answer.
- Reword the intro and How to Play % explainers with a worked warmer/colder example.
- Rename player-facing "level" copy to "game"; keep internal identifiers and the storage key.
- Remove em dashes from UI copy.

**Done when**
- Lower bounds show a down arrow, upper bounds an up arrow, each before the %.
- Both modals explain the % with the warmer/colder example.
- No player-facing copy says "level"; no UI copy uses an em dash.
- `?selftest` shows PASS and `game.html` / `sandwich-tutorial.html` are unchanged.

---

## [x] Phase 8 — Analytics tracker

<!-- Done: fire-and-forget analytics to a Google Apps Script web app. A row is
     sent on real level start (level_started: userId, sessionId, level, at) and
     on finish (level_completed: + guesses, won, timeTakenSec). Loss is encoded
     as guesses 11 + won:false (gameplay unchanged at MAX_GUESSES 10; 11 is an
     analytics-only "did not solve" sentinel), win is the actual 1-10 + won:true.
     userId is a random UUID in localStorage ("sandwich-v2-uid", try/catch,
     ephemeral fallback); sessionId is one UUID per page load; uuid() prefers
     crypto.randomUUID with a Math.random fallback. sendAnalytics uses
     navigator.sendBeacon (text/plain Blob, no CORS preflight) then falls back
     to fetch no-cors keepalive, all in try/catch so it never blocks or throws.
     ANALYTICS_URL is blank by default => a complete no-op (offline-safe, no
     network in the self-test); paste the deployed /exec URL to enable. Only the
     three real levels are tracked (guarded by currentLevelIndex !== null, so the
     tutorial and free-play send nothing). Self-tests 17 (payload shape, loss
     11/false encoding, uuid non-empty). Verified headless end-to-end with a
     dummy URL: L1 start->win {guesses:1,won:true}, L2 start->10 wrong
     {guesses:11,won:false,timeTakenSec}, tutorial sends nothing, userId stable
     across events, sessionId stable within a load; with the URL blank, zero
     network. The Apps Script and deploy steps are handed to Lachlan separately. -->

- Send a row on level start and level finish to a Google Sheet via Apps Script.
- Fields: userId (persistent UUID), sessionId (per load), level (1-3), and on
  finish guesses (11 = didn't solve), won, timeTakenSec; Apps Script stamps the
  server time.
- Gameplay is unchanged: still exactly 10 guesses per game.
- Blank ANALYTICS_URL is a no-op; only the three real levels are tracked.

**Done when**
- `?selftest` shows PASS and the new payload assertions are included.
- With the URL blank, playing a level makes no network calls.
- With a URL set, start/win/loss each produce one correctly-shaped row and the
  tutorial produces none.
- `git diff` shows no change to `game.html` or `sandwich-tutorial.html`.

---

## Decisions to confirm

- Phase 0 ❓ The ••• / Prototype menu cards use hardcoded `#fff`/rgba rather than `:root` tokens (same as game.html's overlays). OK to keep, or tokenise?
- Phase 2 ❓ `CLOSENESS_CURVE` default is 1500 (exponential decay length, in ranks). Tune before Phase 3.
- Phase 2 ❓ The first ~50 ranks still read 97–99; the spread kicks in past ~100 ranks. Is that near-region flatness acceptable, or lower CLOSENESS_CURVE to spread it more?
- Phase 2 ❓ The Closeness table renders 13 rows (12 words + the answer row), offsets ±[1,10,50,150,500,1500].
- Phase 3 ❓ Design reverted to game.html inline `.bound-meta` % (no big number, no guess list, no count-up) per Lachlan.
- Phase 3 ❓ Curve now sqrt, `CLOSENESS_K=2`: skirr 98, skate 87, full window ~23%. Warm enough / too warm? `CLOSENESS_K` is the single knob (bigger = colder).
- Phase 3 ❓ Closeness stays rank-based and bounds-independent (same guess = same %). game.html's original % was instead relative to the opening bounds (so it drifted as bounds closed). Keep rank-based, or match game.html's opening-relative behaviour?


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

## [ ] Phase 1 — Dictionary validation

- Guesses must be in `WORDS`. Use the existing indexes (e.g. `RANK`), not a new array.
- A word that isn't in the list shows "Not in word list" using the existing message area and shake. It does not use a guess and does not show a %.
- Keep the existing order of checks: length, then between bounds, then dictionary.
- Keyboard dimming is unchanged (alphabetical only).

**Done when**
- Self-tests cover: a valid word between the bounds is accepted, a non-word between the bounds is rejected, a valid word outside the bounds is rejected with the existing message.
- A rejected guess leaves `guessesLeft` unchanged.

---

## [ ] Phase 2 — Closeness engine · CHECKPOINT

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

## [ ] Phase 3 — Thermometer UI (% number only)

- Each guess in the guess list shows its % on the right of the row.
- The latest guess's % also appears large, directly under the bounds, labelled "Closeness". Before the first guess, show "—".
- When a guess resolves (after the existing 750ms think), the large number counts up from the previous value to the new one over ~500ms. Input stays blocked until it finishes. Reduced motion: it changes instantly.
- The best % so far is marked in the guess list with the `--accent` colour. Everything else uses `--foreground`.
- A win shows 100 and the existing win prompt follows.
- No bars, gauges or colour ramps. The number only.

**Done when**
- Tested by winning, losing, and three guesses that go colder then warmer.
- Input is blocked for the full think + count-up.

---

## [ ] Phase 4 — Levels

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

## [ ] Phase 5 — First-load intro modal

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

## [ ] Phase 6 — Tutorial on the real engine · CHECKPOINT

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

## Decisions to confirm

- Phase 0 ❓ The ••• / Prototype menu cards use hardcoded `#fff`/rgba rather than `:root` tokens (same as game.html's overlays). OK to keep, or tokenise?

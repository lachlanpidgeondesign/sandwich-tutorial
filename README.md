# Sandwich — Interactive Tutorial

A self-contained, interactive "How to Play" tutorial for the Sandwich word game,
where the goal is to guess a secret word hidden alphabetically between two other words.

Open [sandwich-tutorial.html](sandwich-tutorial.html) in any modern browser — no build
step or dependencies required.

## Flow

- **Goal:** guess the secret word that sits alphabetically between the two shown words
  (MILK ↔ STAR, answer PLAY).
- Only keys that keep your typed prefix in range are enabled; word length is variable.
- Each guess pauses on a brief "thinking" beat, then the word slides up or down to become
  the new boundary, tightening the gap.
- After two wrong guesses the keyboard starts lighting a hinted path (P, then L), then
  guides the final word (PLAY).
- If you solve it before seeing the hints, a short optional demo (BOAT ↔ FISH, answer
  DUCK) shows how the hint system works.

## Notes

This is a standalone reimagining of the tutorial. The original saved game page and its
bundle are intentionally excluded from version control (see `.gitignore`) as they are
third-party assets.

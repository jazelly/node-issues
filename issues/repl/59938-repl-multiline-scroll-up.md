# Redraw tall multiline REPL input when moving above the visible rows

- Issue: [#59938](https://github.com/nodejs/node/issues/59938) — open when checked
- Checked: 2026-11-09
- Status: No active fix found; two PRs were closed without landing

## What's wrong?

When multiline REPL input is taller than the terminal, pressing Up at the top
visible line moves the cursor, but the screen does not scroll. The lower lines
stay on screen, so the user edits text they cannot see.

## Where to start

- Code: [lib/internal/readline/interface.js `kMultilineMove` and `kMoveUpOrHistoryPrev`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/readline/interface.js#L1120-L1192)
- Docs: [REPL](https://nodejs.org/api/repl.html)
- Try it: Not reproduced here; it needs an interactive TTY smaller than the
  input. The issue has a screen recording from a contributor.

## What's been tried?

- 2025-10 — [#60222](https://github.com/nodejs/node/pull/60222): handled screen
  refresh for multiline input. The author could not write a test that emulates a
  small terminal and closed it in favor of #61248.
- 2026-01 — [#61248](https://github.com/nodejs/node/pull/61248): a fix with a test,
  `test-repl-multiline-navigation-while-adding.js`. Never reviewed; closed as stale
  in August 2026.
- Related: [#59431](https://github.com/nodejs/node/issues/59431) (crash recalling a
  large multiline history entry) has an open PR,
  [#65610](https://github.com/nodejs/node/pull/65610), touching the same code.

## Suggested fix

- Idea: Start from #61248. When the cursor moves above the first visible row,
  redraw the visible window instead of only moving the cursor.
- Benefit: Makes editing long pasted snippets in the REPL usable.
- Risk: Terminal redraw logic is easy to break; coordinate with #65610 so the two
  changes do not conflict.

## Next step

Reproduce in a small terminal, rebase #61248 on main, and ask the #65610 author
whether the changes overlap.

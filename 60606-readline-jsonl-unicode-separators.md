# #60606 — Readline splits Unicode separators inside JSONL records

- Upstream: [JSONL regression with U+2028 and U+2029](https://github.com/nodejs/node/issues/60606)
- Checked: 2026-09-11; open, no assignee.
- Upstream labels: `readline`, `repl`.
- Area: `readline`, `repl`.
- Assessment: JS-only follow-up; the separator API remains unresolved.

## Problem and entry point

`JSON.stringify()` can leave U+2028 and U+2029 inside a string. When `readline` treats those characters as record boundaries, a JSONL record is split before `JSON.parse()` receives it. The discussion corrected the initial suspicion of a general JSON parser regression: the splitting happens in readline.

The [current readline implementation](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/readline/interface.js#L89) explicitly includes both characters in its line-ending expression. The input splitting and paste-handling paths both need consideration.

## Smallest useful follow-up

Create a focused JS test that stringifies an object containing each separator, appends a normal newline, streams it through readline, and parses emitted records. This makes the boundary failure observable without changing V8 or the JSON parser.

Before implementing a fix, resolve the outstanding API question: an opt-out boolean versus configurable line separators. Reviewers of the previous attempt requested a broader separator option and said changing the default should be a separate semver change. Do not assume a boolean option has approval or silently change defaults.

Suggested new regression location: `test/parallel/test-readline-jsonl-separators.js` (proposed file, not an existing test).

## Activity evidence

[PR #63789](https://github.com/nodejs/node/pull/63789) attempted a Unicode-separator option and closed unmerged on 2026-07-21. Its inline review dated 2026-06-08 records the API discussion. The latest issue comment is dated 2026-04-13. Both this issue and related [#22448](https://github.com/nodejs/node/issues/22448) were checked; no open replacement implementation PR was found.

This is the least ready of the five candidates and should follow the simpler tests. No reproduction or patch was performed for this notebook.

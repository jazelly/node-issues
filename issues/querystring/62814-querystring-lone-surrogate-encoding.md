# Reject lone surrogates in querystring encoding

- Issue: [#62814](https://github.com/nodejs/node/issues/62814) — closed by the stale bot when checked
- Checked: 2026-11-09
- Status: No active fix found; [#62834](https://github.com/nodejs/node/pull/62834) had three approvals and was closed as stale

## What's wrong?

`querystring.escape()` and `stringify()` treat any surrogate code unit as the
start of a valid pair. A lone high or low surrogate is combined with the next
code unit into an unrelated four-byte sequence, so `unescape(escape(s))` does
not return `s`. `encodeURIComponent()` throws `URIError` for the same input,
and the code already throws when a surrogate is the last character.

## Where to start

- Code: [lib/internal/querystring.js `encodeStr` surrogate branch](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/querystring.js#L89-L110)
- Docs: [`querystring.escape()`](https://nodejs.org/api/querystring.html#querystringescapestr)
- Try it: On `v28.0.0-pre` (`128f247ea4cb`, macOS arm64):
  `qs.escape('\uD800\uD800')` and `qs.escape('\uDC00\uDC00')` both return
  `%F0%90%80%80`, and `qs.escape('a\uD800b')` returns `a%F0%90%81%A2`. None
  round-trip.

## What's been tried?

- 2026-04 — [#62814](https://github.com/nodejs/node/issues/62814): mcollina
  recommended fixing it in core because the standards-based alternative is much
  slower. Closed as stale in August 2026.
- 2026-04 — [#62834](https://github.com/nodejs/node/pull/62834): validated both
  halves of the pair and threw `ERR_INVALID_URI` otherwise. Approved by anonrig,
  gurgunday, and mcollina. ChALkeR asked the TSC whether it is semver-major; no
  answer, and the PR was closed as stale.

## Suggested fix

- Idea: Revive #62834. Check that the lead unit is `0xD800`–`0xDBFF` and the
  trailing unit is `0xDC00`–`0xDFFF`, and throw `ERR_INVALID_URI` otherwise.
- Benefit: Matches `encodeURIComponent()` and the existing end-of-string check.
- Risk: Code that currently gets silently corrupted output will start throwing.
  The semver classification is the open question.
- Alternative (optional): Encode lone surrogates as U+FFFD instead of throwing,
  which is what `URLSearchParams` does via USVString conversion.

## Next step

Rebase #62834 with credit to its author and ask in the PR for a semver-major or
patch decision before landing.

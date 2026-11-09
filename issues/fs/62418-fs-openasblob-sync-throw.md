# Return a rejected promise from `fs.openAsBlob()` on bad input

- Issue: [#62418](https://github.com/nodejs/node/issues/62418) — open when checked
- Checked: 2026-11-09
- Status: No active fix found; [#62655](https://github.com/nodejs/node/pull/62655) was closed as stale

## What's wrong?

`fs.openAsBlob()` is documented to return a Promise, and its own comment says it
returns one for future flexibility. Invalid arguments and blob-creation errors
are thrown synchronously instead, so `await fs.openAsBlob(x).catch(...)`
misses them.

## Where to start

- Code: [lib/fs.js `openAsBlob`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/fs.js#L796-L812)
- Docs: [`fs.openAsBlob()`](https://nodejs.org/api/fs.html#fsopenasblobpath-options)
- Try it: On `v28.0.0-pre` (`128f247ea4cb`, macOS arm64), `fs.openAsBlob(123)`
  throws `ERR_INVALID_ARG_TYPE` synchronously instead of returning a rejected
  promise.

## What's been tried?

- 2026-03 — [#62418](https://github.com/nodejs/node/issues/62418): targos called it
  odd and asked why the API lives in `node:fs`. Others want it copied to
  `node:fs/promises`. The reporter offered to open a fix if the existing PR fails.
- 2026-04 — [#62655](https://github.com/nodejs/node/pull/62655): wrapped the body so
  errors become rejections. Discussion only, no approval; closed as stale.
- 2023 — [#49759](https://github.com/nodejs/node/pull/49759): an earlier attempt to
  restructure `openAsBlob` and add `fsPromises.openAsBlob`, also closed as stale.

Nearby open PRs change different `openAsBlob` behavior:
[#67346](https://github.com/nodejs/node/pull/67346) (paths across `chdir`) and
[#66204](https://github.com/nodejs/node/pull/66204) (regular files only).

## Suggested fix

- Idea: Turn synchronous errors into rejections, as #62655 did.
- Benefit: Matches the documented return type.
- Risk: Node often throws argument errors synchronously even from
  promise-returning APIs, so maintainers may prefer documenting the current
  behavior. Coordinate with the two open PRs to avoid conflicts.
- Alternative (optional): Add `openAsBlob` to `node:fs/promises` with rejecting
  semantics. That is new API surface and outside this notebook's scope.

## Next step

Ask on the issue whether the fs team wants rejections or a docs change, then
revive #62655 or write the docs patch.

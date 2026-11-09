# Reject file modes above Int32 before reaching the binding

- Issue: [#62516](https://github.com/nodejs/node/issues/62516) — closed by the stale bot when checked
- Checked: 2026-11-09
- Status: No active fix found; [#62538](https://github.com/nodejs/node/pull/62538) was closed as stale without review

## What's wrong?

A file mode that is a valid Uint32 but larger than Int32 max aborts the whole
process with a C++ assertion instead of throwing a catchable `RangeError`.
JavaScript validation accepts the full Uint32 range, while the binding asserts
`IsInt32()`. Any user-controlled mode value can crash the process.

## Where to start

- Code: [lib/internal/validators.js `parseFileMode`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/validators.js#L74-L86),
  [lib/fs.js `open` / `openSync`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/fs.js#L750-L776),
  [lib/internal/fs/streams.js stream `mode`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/fs/streams.js#L214)
- Docs: [File modes](https://nodejs.org/api/fs.html#file-modes)
- Try it: On `v28.0.0-pre` (`128f247ea4cb`, macOS arm64), both
  `fs.openSync(path, 'w', 0xFFFFFFFF)` and
  `fs.createWriteStream(path, { mode: 0xFFFFFFFF })` abort with
  `Assertion failed: args[2]->IsInt32()` in `node::fs::Open`.

## What's been tried?

- 2026-03 — [#62516](https://github.com/nodejs/node/issues/62516): a commenter traced
  it to [#52050](https://github.com/nodejs/node/pull/52050), which moved mode
  validation from JS into C++. No maintainer reply; closed as stale in August 2026.
- 2026-04 — [#62538](https://github.com/nodejs/node/pull/62538): added Int32
  validation for the stream constructors only. Never reviewed; closed as stale.

Search: `repo:nodejs/node` IsInt32 mode; createWriteStream mode crash;
parseFileMode Uint32. No open PR found.

## Suggested fix

- Idea: Make the JS layer and the binding agree. Either validate modes as Int32
  (or mask to the permission bits) before calling the binding, or let the binding
  accept Uint32. Cover `open`, `openSync`, `fs.promises.open`, and the stream
  constructors through one shared path.
- Benefit: Turns a process abort into a normal argument error.
- Risk: Narrowing the accepted range could reject values that worked before
  #52050. Check which of the about 20 `IsInt32()` mode assertions in
  `src/node_file.cc` share the problem.
- Alternative (optional): Fix only the stream constructors, as #62538 did. Smaller,
  but leaves `fs.openSync` crashing.

## Next step

List every public fs API that takes `mode`, run each with `0xFFFFFFFF`, and fix
the ones that abort through `parseFileMode` or a shared helper.

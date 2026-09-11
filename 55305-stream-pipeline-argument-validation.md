# Reject invalid stream roles before pipeline setup

- Area: `stream`.
- Assessment: **Small slice of a larger fix; source checked**.
- Checked: **2026-09-11**.
- Upstream: [#55305](https://github.com/nodejs/node/issues/55305); open at review.
- Upstream labels: `stream`, `never-stale`.
- Source: [lib/internal/streams/pipeline.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/streams/pipeline.js#L186), [lib/internal/streams/pipeline.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/streams/pipeline.js#L295).

## Observable problem or cost

Readable-only and writable-only streams can reach unsuitable pipeline positions. Falling through conversion/setup can produce unrelated asynchronous errors or an operation that cannot make progress.

## Narrow JavaScript change

Begin with one clearly invalid classic-stream role, such as a writable-only source. Validate the required capability before wiring resources while preserving valid iterable, function, and Web Stream conversions and the existing error-delivery convention.

## History, objections, and ownership

[#55305](https://github.com/nodejs/node/issues/55305) describes the role matrix. [#62214](https://github.com/nodejs/node/pull/62214) was automatically closed on 2026-08-27 after 120 days without activity. No replacement matching that issue was found. Active [#65064](https://github.com/nodejs/node/pull/65064) and [#65128](https://github.com/nodejs/node/pull/65128) concern cleanup/callback behavior on synchronous throws; avoid extending this candidate into their work.

## Evidence and limits

The current source still converts unrecognized source values through Duplex.from. This pass checked source and discussion; it did not rerun the full invalid-input matrix or establish that every case is a bug. [Pipeline accepted inputs](https://nodejs.org/api/stream.html#streampipelinesource-transforms-destination-callback).

Search terms used with `repo:nodejs/node`: 55305; 62214; pipeline invalid arguments. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

# Reject invalid stream roles before pipeline setup

- Issue: [#55305](https://github.com/nodejs/node/issues/55305) — open when checked
- Checked: 2026-09-11
- Status: No matching role-validation fix active; adjacent pipeline PRs need review (at that check)

## What's wrong?

pipeline() can accept a stream in a role it cannot perform, such as a write-only stream used as the source. Setup may then fail with an unrelated error or leave a pipeline that cannot make progress.

## Where to start

- Code: [lib/internal/streams/pipeline.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/streams/pipeline.js#L186), [lib/internal/streams/pipeline.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/streams/pipeline.js#L295)
- Docs: [Pipeline accepted inputs](https://nodejs.org/api/stream.html#streampipelinesource-transforms-destination-callback)
- Try it / evidence: The current source still converts unrecognized source values through Duplex.from. This pass checked source and discussion; it did not rerun the full invalid-input matrix or establish that every case is a bug. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#55305](https://github.com/nodejs/node/issues/55305) describes the invalid roles. [#62214](https://github.com/nodejs/node/pull/62214) closed for inactivity on 2026-08-27.
- [#65064](https://github.com/nodejs/node/pull/65064) and [#65128](https://github.com/nodejs/node/pull/65128) were active on cleanup and callback behavior after synchronous throws. Keep that nearby work separate.

Search: `repo:nodejs/node` — 55305; 62214; pipeline invalid arguments. Recheck activity before starting.

## Suggested fix

- Idea: Reject one clearly invalid classic-stream role before connecting the pipeline.
- Benefit: Gives a relevant error before setting up resources.
- Risk: Do not reject supported functions, iterables, or Web Stream conversions. Improving a late error is an alternative, but may leave unnecessary setup or a stalled pipeline.

## Next step

Reproduce one invalid role and check the nearby PRs before proposing early validation.

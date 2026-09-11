# Reject invalid stream roles before pipeline setup

- Area: `stream`.
- Kind: `correctness`.
- Assessment: **Small slice of a larger fix; source checked**.
- Last researched: **2026-09-11**.
- Upstream: [#55305](https://github.com/nodejs/node/issues/55305); open at review.
- Upstream labels: `stream`, `never-stale`.
- Source: [lib/internal/streams/pipeline.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/streams/pipeline.js#L186), [lib/internal/streams/pipeline.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/streams/pipeline.js#L295).

## Problem and impact

Readable-only and writable-only streams can reach unsuitable pipeline positions. Falling through conversion/setup can produce unrelated asynchronous errors or an operation that cannot make progress.

## Evidence and reproduction

The current source still converts unrecognized source values through Duplex.from. This pass checked source and discussion; it did not rerun the full invalid-input matrix or establish that every case is a bug. [Pipeline accepted inputs](https://nodejs.org/api/stream.html#streampipelinesource-transforms-destination-callback).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| 2026-08-27 (closure) | [PR #62214](https://github.com/nodejs/node/pull/62214) | Previous fix attempt | Closed for inactivity | A follow-up must preserve valid pipeline input conversions. |
| 2026-09-11 (snapshot) | [PR #65064](https://github.com/nodejs/node/pull/65064) | Nearby active work | Open at research | Owns cleanup on synchronous pipeline throw; avoid expanding into it. |
| 2026-09-11 (snapshot) | [PR #65128](https://github.com/nodejs/node/pull/65128) | Nearby active work | Open at research | Owns callback behavior on synchronous pipeline throw. |

[#55305](https://github.com/nodejs/node/issues/55305) describes the role matrix. [#62214](https://github.com/nodejs/node/pull/62214) was automatically closed on 2026-08-27 after 120 days without activity. No replacement matching that issue was found. Active [#65064](https://github.com/nodejs/node/pull/65064) and [#65128](https://github.com/nodejs/node/pull/65128) concern cleanup/callback behavior on synchronous throws; avoid extending this candidate into their work.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Validate a clearly invalid stream role before setup | Produces a relevant failure before wiring resources. | Overly broad checks could reject valid conversion inputs. |
| B: Retain late conversion and improve downstream error reporting | Changes less of the admission logic. | Can leave unnecessary setup or non-progressing pipelines in place. |

## Recommendation and boundaries

Start with one invalid classic-stream role and preserve supported conversions.

Begin with one clearly invalid classic-stream role, such as a writable-only source. Validate the required capability before wiring resources while preserving valid iterable, function, and Web Stream conversions and the existing error-delivery convention.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Reproduce one invalid classic-stream role and recheck the adjacent synchronous-throw PRs before adding early validation.

Search terms used with `repo:nodejs/node`: 55305; 62214; pipeline invalid arguments. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Pipeline accepted inputs](https://nodejs.org/api/stream.html#streampipelinesource-transforms-destination-callback)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

# Discard pending values on explicit events.on() cancellation

- Area: `events`.
- Kind: `resource-lifecycle`.
- Assessment: **Small to medium fix; reproduced**.
- Last researched: **2026-09-11**.
- Upstream: None identified; related historical PRs are references, not a claimed issue number.
- Upstream labels: Not applicable.
- Source: [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1109), [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1140), [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1228).

## Problem and impact

Explicit return() reports done, but subsequent next() calls consume buffered events before checking finished. Keeping the iterator also retains queued argument objects after cancellation.

## Evidence and reproduction

Both local runtimes returned {done:true} from return(), then {done:false,value:["buffered"]} from next() after one event was queued. The proposed cleanup concerns explicit cancellation, not a blanket finished-first check. [events.on contract](https://nodejs.org/api/events.html#eventsonemitter-eventname-options).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #27994](https://github.com/nodejs/node/pull/27994) | Introducing change | Landed | Established event async iteration and its completion behavior. |
| Unknown in notebook | [PR #41276](https://github.com/nodejs/node/pull/41276) | Related implementation | Landed | Introduced direct event iteration with efficient queues and backpressure. |
| Unknown in notebook | [PR #51091](https://github.com/nodejs/node/pull/51091) | Related cleanup | Landed | Disposed abort listeners; did not clear the pending event-value queue. |

[#27994](https://github.com/nodejs/node/pull/27994) introduced event async iteration; [#41276](https://github.com/nodejs/node/pull/41276) added efficient queues and backpressure, and [#51091](https://github.com/nodejs/node/pull/51091) cleaned up abort listeners on completion. Those landed improvements do not clear this value queue. Searches for events.on return/buffering found no exact open fix. Returning done is the observed behavior; changing post-return draining still deserves compatibility review.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Give explicit return its own queue cleanup path | Releases retained values while preserving natural completion draining. | Must settle pending calls and handle any pause owned by the iterator. |
| B: Check finished before reading the queue in every next call | Very small implementation change. | Also discards values buffered before natural close; broader behavior change. |

## Recommendation and boundaries

Prefer explicit-cancellation cleanup; do not silently change natural-close draining.

Distinguish explicit consumer cancellation from natural close-event draining. On explicit return, release queued values and finish pending next calls. Preserve buffered draining on natural completion; review how an emitter paused by this iterator should be released.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Verify explicit return versus natural-close draining and define cleanup for an emitter paused by the iterator.

Search terms used with `repo:nodejs/node`: events.on return; events.on buffer; events.on pause. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [events.on contract](https://nodejs.org/api/events.html#eventsonemitter-eventname-options)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

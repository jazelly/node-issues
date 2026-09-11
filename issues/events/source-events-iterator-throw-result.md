# Return an async-iterator result from events.on().throw()

- Area: `events`.
- Kind: `correctness`.
- Assessment: **Small fix; reproduced**.
- Last researched: **2026-09-11**.
- Upstream: None identified; related historical PRs are references, not a claimed issue number.
- Upstream labels: Not applicable.
- Source: [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1144).

## Problem and impact

The iterator throw method runs errorHandler and returns undefined. Async-iterator consumers that delegate to this method expect its awaited result to be an object, so the missing result can replace the intended error with a protocol TypeError.

## Evidence and reproduction

Both local runtimes returned undefined from iterator.throw(new Error("stop")); the next next() rejected with "stop". This demonstrates the missing result and current error delivery, not an agreed replacement contract. [events.on documentation](https://nodejs.org/api/events.html#eventsonemitter-eventname-options).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #41708](https://github.com/nodejs/node/pull/41708) | Previous broader attempt | Closed | Identified the missing throw result; stalled on the related readline change and rebase. |
| Unknown in notebook | [PR #41276](https://github.com/nodejs/node/pull/41276) | Related implementation | Landed | Introduced direct event iteration with efficient queues and backpressure. |

The body of the old [#41708](https://github.com/nodejs/node/pull/41708) explicitly identified this wrong return value. That broad readline proposal stalled waiting for [#41276](https://github.com/nodejs/node/pull/41276) and a rebase; the latter landed, but the current throw method still lacks a return. [#40797](https://github.com/nodejs/node/issues/40797) concerns intentional error-event rejection, which this change must preserve. No matching open implementation was found.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Return a promise/result while keeping the queued-error contract | Fixes the iterator result shape with limited error-ordering changes. | Must specify the result and behavior of pending next calls. |
| B: Reject throw immediately with the supplied error | Matches familiar throwing-iterator behavior. | Changes where the error is delivered and risks duplicate delivery through next. |

## Recommendation and boundaries

Agree on the delivery contract first; the missing result is reproduced, not the preferred semantics.

Give throw a deliberate promise/result contract after handling the error, matching the existing queued-error behavior. Decide explicitly which call receives the original error; avoid delivering it twice or dropping pending next calls.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Specify which call receives the supplied error and how pending next calls settle before choosing the returned promise/result.

Search terms used with `repo:nodejs/node`: events.on throw; events iterator throw result; 41708. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [events.on documentation](https://nodejs.org/api/events.html#eventsonemitter-eventname-options)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

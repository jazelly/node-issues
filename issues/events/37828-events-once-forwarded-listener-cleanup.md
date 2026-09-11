# Clean up a settled once promise when its callback is forwarded

- Area: `events`.
- Kind: `resource-lifecycle`.
- Assessment: **Medium fix; reproduced; semantics review**.
- Last researched: **2026-09-11**.
- Upstream: [#37828](https://github.com/nodejs/node/issues/37828); closed at review.
- Upstream labels: `events`, `stale`.
- Source: [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1015).

## Problem and impact

A newListener-based event forwarder can call the promise resolver directly, bypassing the original emitter once wrapper. The promise settles, but its original event listener remains attached on both emitters.

## Evidence and reproduction

The supplied forwarding pattern was reduced and run on both local runtimes. After the promise resolved, listenerCount("value") was 1 on both emitters. This is not a claim that arbitrary forwarding must emulate every EventEmitter behavior. [once and newListener documentation](https://nodejs.org/api/events.html#eventsonceemitter-name-options).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| 2026-09-11 (snapshot) | No implementation PR identified in the recorded search | No identified attempt | Unknown beyond the search scope | Issue discussion below is evidence; absence of a PR is not approval or proof of no work. |

[#37828](https://github.com/nodejs/node/issues/37828) was independently reproduced. Discussion questioned how exposed listeners and wrappers should interact, and did not settle the preferred fix. It closed stale in July 2026, with no linked implementation or current claim found. This narrower cleanup avoids changing what newListener exposes, but still needs events-maintainer agreement.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Remove the original event listener in the settled promise resolver | Addresses retained listeners without changing newListener payloads. | Needs agreement on forwarded-listener semantics and removal ordering. |
| B: Expose the once wrapper to forwarding listeners | Lets forwarded invocation run the normal once cleanup. | Changes a public listener-observation contract and affects general EventEmitter behavior. |

## Recommendation and boundaries

Prefer scoped resolver cleanup only after the forwarding behavior is accepted.

Make the promise resolver remove its event listener on settlement through the existing agnostic removal helper. Keep the public newListener contract, normal once-wrapper ordering, EventTarget support, and error-listener cleanup unchanged.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Resolve the documented forwarding/removal-order question before preparing resolver-local cleanup.

Search terms used with `repo:nodejs/node`: 37828; events once forwarding cleanup; newListener resolver. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [once and newListener documentation](https://nodejs.org/api/events.html#eventsonceemitter-name-options)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

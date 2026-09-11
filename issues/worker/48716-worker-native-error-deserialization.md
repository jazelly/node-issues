# Reconstruct genuine native Error objects from worker failures

- Area: `worker`.
- Kind: `correctness`.
- Assessment: **Small to medium fix; reproduced**.
- Last researched: **2026-09-11**.
- Upstream: [#48716](https://github.com/nodejs/node/issues/48716); closed at review.
- Upstream labels: `errors`, `worker`, `stale`.
- Source: [lib/internal/error_serdes.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/error_serdes.js#L175).

## Problem and impact

Object.create(ctor.prototype, properties) looks like an Error to instanceof but lacks the native Error identity. Consumers using util.types.isNativeError reject errors emitted by worker failure transport.

## Evidence and reproduction

On both local runtimes, a round-tripped TypeError passed instanceof TypeError but failed util.types.isNativeError. This was a direct transport probe, not a full worker lifecycle run. [Worker error events](https://nodejs.org/api/worker_threads.html#event-error), [native-error identification](https://nodejs.org/api/util.html#utiltypesisnativeerrorvalue).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #58605](https://github.com/nodejs/node/pull/58605) | Previous fix attempt | Closed | Review asked about AggregateError and SuppressedError; constructor handling remains relevant. |
| Unknown in notebook | [PR #62865](https://github.com/nodejs/node/pull/62865) | Previous fix attempt | Closed | Unanswered human-review and contribution feedback; closure does not establish design rejection. |

[#48716](https://github.com/nodejs/node/issues/48716) welcomed a fix and later closed for inactivity. [#58605](https://github.com/nodejs/node/pull/58605) is closed; review asked about AggregateError and SuppressedError coverage rather than rejecting native reconstruction. [#62865](https://github.com/nodejs/node/pull/62865) closed after unanswered human-review and contribution-guideline feedback. A fresh change must account for current constructors instead of copying either old patch. No matching open PR was found.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Construct the built-in Error, then restore descriptors | Restores native identity while retaining the existing custom transport. | Constructor-specific handling and stack/cause restoration need care. |
| B: Replace custom error transport with built-in V8 serialization | Uses native reconstruction already provided by V8. | May lose properties or descriptor behavior that custom transport deliberately preserves. |

## Recommendation and boundaries

Prefer native construction within the existing transport; do not replace the format.

Construct the appropriate built-in Error and restore serialized property descriptors without inventing a new wire format. Handle the already-supported AggregateError constructor separately, and preserve original stacks, causes, and descriptor attributes.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Enumerate currently supported error constructors and verify stack/cause descriptor restoration for the constructor-based approach.

Search terms used with `repo:nodejs/node`: 48716; deserialize native errors; error_serdes. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Worker error events](https://nodejs.org/api/worker_threads.html#event-error)
- [native-error identification](https://nodejs.org/api/util.html#utiltypesisnativeerrorvalue)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

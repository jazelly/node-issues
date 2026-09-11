# Preserve Unicode Symbol descriptions during error deserialization

- Area: `worker`.
- Kind: `correctness`.
- Assessment: **Small fix; reproduced**.
- Last researched: **2026-09-11**.
- Upstream: None identified; related historical PRs are references, not a claimed issue number.
- Upstream labels: Not applicable.
- Source: [lib/internal/error_serdes.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/error_serdes.js#L192).

## Problem and impact

The Symbol branch decodes a UTF-8 buffer, then uses the buffer byte length as the end index of a JavaScript substring. Non-ASCII descriptions therefore retain the closing parenthesis and produce a different registry Symbol.

## Evidence and reproduction

On both local runtimes, round-tripping Symbol.for("ascii") preserved its description; "é", "中文", and "😀" became "é)", "中文)", and "😀)". The probe used internal/error_serdes with --expose-internals. [Worker error-event contract](https://nodejs.org/api/worker_threads.html#event-error).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #47925](https://github.com/nodejs/node/pull/47925) | Transport precedent | Landed | Introduced broader error transport; review covered getters, descriptors, and allocation cost. |

[#47925](https://github.com/nodejs/node/pull/47925) introduced this branch while extending error transport. Its review discussed allocation cost and difficult error descriptors, but the reviewed discussion did not require byte-based indexing or identify a Unicode reason for it. Searches for `kInspectedSymbol` and Symbol error serialization found no open correction. This is a source-discovered candidate, not an upstream issue number.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Decode once, then slice by string length | Matches JavaScript string indexing; keeps the transport format. | Must preserve empty descriptions and the existing Symbol.for behavior. |
| B: Remove the ASCII prefix/suffix at byte boundaries before decoding | Avoids mixing byte and string indices. | More closely couples parsing to the exact encoded framing. |

## Recommendation and boundaries

Prefer string-based slicing for the smallest readable correction.

Decode once and calculate the substring boundary from the decoded string length. Keep the existing Symbol.for transport semantics and serialization tag unchanged.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Check empty and non-ASCII descriptions against the existing transport framing, then prepare the string-boundary correction.

Search terms used with `repo:nodejs/node`: kInspectedSymbol; serializeError symbol; error_serdes. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Worker error-event contract](https://nodejs.org/api/worker_threads.html#event-error)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

# Close file descriptor zero in file streams

- Area: `fs`.
- Kind: `resource-lifecycle`.
- Assessment: **Small fix; reproduced**.
- Last researched: **2026-09-11**.
- Upstream: [#57905](https://github.com/nodejs/node/issues/57905); open at review.
- Upstream labels: none.
- Source: [lib/internal/fs/streams.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/fs/streams.js#L124).

## Problem and impact

The shared close helper treats a falsy descriptor as absent. Descriptor `0` is valid, so an automatically destroyed file stream can report completion without calling its close operation.

## Evidence and reproduction

Reproduced on v24.15.0 and the local v27.0.0-pre build using the documented custom `fs` option: an EOF read with fd 0 emitted close with `destroyed: true`, but the close callback implementation was invoked zero times. This avoided touching the process standard input. [File-stream contract](https://nodejs.org/api/fs.html#fscreatereadstreampath-options).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| 2025 (year only) | [PR #57914](https://github.com/nodejs/node/pull/57914) | Previous fix attempt | Closed, unmerged | Review requested evidence specifically covering fd 0. |
| 2025 (year only) | [PR #57918](https://github.com/nodejs/node/pull/57918) | Previous fix attempt | Closed, unmerged | No design rejection recorded in the existing research. |

The [issue](https://github.com/nodejs/node/issues/57905) was reconfirmed from source in July 2026. [#57914](https://github.com/nodejs/node/pull/57914) and [#57918](https://github.com/nodejs/node/pull/57918) are closed, unmerged attempts from 2025. The former review asked for evidence covering fd 0; no design rejection was found. No replacement open fix or recent ownership claim was found.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Explicit absent-descriptor check | Small change; closes valid fd 0 through the existing lifecycle. | Must preserve null/undefined handling and exactly-once close callbacks. |
| B: Revalidate the descriptor as a non-negative integer here | Makes the accepted descriptor domain explicit. | Duplicates earlier validation and can change how invalid internal states fail. |

## Recommendation and boundaries

Prefer the absent-descriptor check after confirming the helper sentinel contract.

Replace the truthiness check with the actual absent-descriptor check. Preserve the existing close callback, flush ordering, and error propagation for both ReadStream and WriteStream.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Confirm the absent-fd sentinel and callback ordering before preparing the small JS change.

Search terms used with `repo:nodejs/node`: 57905; file descriptor 0; fs streams close. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [File-stream contract](https://nodejs.org/api/fs.html#fscreatereadstreampath-options)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

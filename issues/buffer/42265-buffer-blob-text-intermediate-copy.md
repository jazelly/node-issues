# Avoid a full intermediate byte aggregation in Blob.text()

- Area: `buffer`.
- Kind: `performance`.
- Assessment: **Medium performance candidate; memory focus**.
- Last researched: **2026-09-11**.
- Upstream: [#42265](https://github.com/nodejs/node/issues/42265); closed at review.
- Upstream labels: `buffer`, `stale`.
- Source: [lib/internal/blob.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/blob.js#L310), [lib/internal/blob.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/blob.js#L440).

## Problem and impact

Blob.text first obtains the complete ArrayBuffer and then decodes it. For a multipart or file-backed blob, the reader can hold chunks plus the joined byte buffer before producing the string, increasing peak memory.

## Evidence and reproduction

The materialize-then-decode path exists in pinned source. No fresh peak-memory benchmark was run, and no speedup is claimed. File-read failures and simultaneous text calls are important correctness constraints on this existing method. [Blob.text contract](https://nodejs.org/api/buffer.html#blobtext).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #44787](https://github.com/nodejs/node/pull/44787) | Accepted optimization | Landed | Reuses TextDecoder to reduce allocation cost. |
| Unknown in notebook | [PR #49873](https://github.com/nodejs/node/pull/49873) | Previous native optimization attempt | Closed | PullAll approach was reopened after a rebase problem; not a landed speedup. |
| Unknown in notebook | [PR #52546](https://github.com/nodejs/node/pull/52546) | Previous native optimization attempt | Closed | A working version did not establish the anticipated speedup. |
| Unknown in notebook | [PR #44966](https://github.com/nodejs/node/pull/44966) | Previous optimization attempt | Closed | Initial performance gains could not be reproduced. |

[#42265](https://github.com/nodejs/node/issues/42265) raised this opportunity and closed stale. [#44787](https://github.com/nodejs/node/pull/44787) intentionally reused a decoder for performance, so per-call allocation has a cost. The native PullAll attempts [#49873](https://github.com/nodejs/node/pull/49873) and [#52546](https://github.com/nodejs/node/pull/52546) did not establish a landed faster solution. [#44966](https://github.com/nodejs/node/pull/44966) withdrew claimed gains when they could not be reproduced. This candidate remains JS-only and must prove peak-memory benefit without hiding throughput regressions.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Incrementally decode reader chunks with operation-local state | Can avoid the complete joined byte buffer and reduce peak memory. | Decoder allocation, split UTF-8, BOM handling, and concurrency require care. |
| B: Retain full ArrayBuffer aggregation with the reused decoder | Keeps the current simple path and decoder-reuse optimization. | Retains the extra materialization cost for large multipart blobs. |

## Recommendation and boundaries

Compare peak memory and throughput; do not trade a claimed memory gain for an unmeasured regression.

Explore decoding the existing reader chunks incrementally so the full joined byte buffer is unnecessary. Use operation-local decoder state for concurrent calls, preserve BOM and split-UTF-8 handling, and retain the small-blob fast path if needed. No new public streaming-text API is proposed.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Compare peak memory and throughput for multipart blobs while preserving concurrent UTF-8 decoding behavior.

Search terms used with `repo:nodejs/node`: 42265; blob text performance; Blob.text arrayBuffer. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Blob.text contract](https://nodejs.org/api/buffer.html#blobtext)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

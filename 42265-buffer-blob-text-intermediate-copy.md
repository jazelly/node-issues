# Avoid a full intermediate byte aggregation in Blob.text()

- Area: `buffer`.
- Assessment: **Medium performance candidate; memory focus**.
- Checked: **2026-09-11**.
- Upstream: [#42265](https://github.com/nodejs/node/issues/42265); closed at review.
- Upstream labels: `buffer`, `stale`.
- Source: [lib/internal/blob.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/blob.js#L310), [lib/internal/blob.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/blob.js#L440).

## Observable problem or cost

Blob.text first obtains the complete ArrayBuffer and then decodes it. For a multipart or file-backed blob, the reader can hold chunks plus the joined byte buffer before producing the string, increasing peak memory.

## Narrow JavaScript change

Explore decoding the existing reader chunks incrementally so the full joined byte buffer is unnecessary. Use operation-local decoder state for concurrent calls, preserve BOM and split-UTF-8 handling, and retain the small-blob fast path if needed. No new public streaming-text API is proposed.

## History, objections, and ownership

[#42265](https://github.com/nodejs/node/issues/42265) raised this opportunity and closed stale. [#44787](https://github.com/nodejs/node/pull/44787) intentionally reused a decoder for performance, so per-call allocation has a cost. The native PullAll attempts [#49873](https://github.com/nodejs/node/pull/49873) and [#52546](https://github.com/nodejs/node/pull/52546) did not establish a landed faster solution. [#44966](https://github.com/nodejs/node/pull/44966) withdrew claimed gains when they could not be reproduced. This candidate remains JS-only and must prove peak-memory benefit without hiding throughput regressions.

## Evidence and limits

The materialize-then-decode path exists in pinned source. No fresh peak-memory benchmark was run, and no speedup is claimed. File-read failures and simultaneous text calls are important correctness constraints on this existing method. [Blob.text contract](https://nodejs.org/api/buffer.html#blobtext).

Search terms used with `repo:nodejs/node`: 42265; blob text performance; Blob.text arrayBuffer. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

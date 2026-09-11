# Avoid a full intermediate byte aggregation in Blob.text()

- Issue: [#42265](https://github.com/nodejs/node/issues/42265) — closed when checked
- Checked: 2026-09-11
- Status: No matching active fix found; memory and speed measurements needed (at that check)

## What's wrong?

Blob.text() first collects all bytes into one ArrayBuffer, then converts them to a string. Large blobs made from several parts can therefore need an extra full-size byte buffer.

## Where to start

- Code: [lib/internal/blob.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/blob.js#L310), [lib/internal/blob.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/blob.js#L440)
- Docs: [Blob.text contract](https://nodejs.org/api/buffer.html#blobtext)
- Try it / evidence: The materialize-then-decode path exists in pinned source. No fresh peak-memory benchmark was run, and no speedup is claimed. File-read failures and simultaneous text calls are important correctness constraints on this existing method. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#42265](https://github.com/nodejs/node/issues/42265) raised this idea and closed for inactivity.
- [#44787](https://github.com/nodejs/node/pull/44787) deliberately reused a decoder to reduce allocations. The native-code attempts [#49873](https://github.com/nodejs/node/pull/49873) and [#52546](https://github.com/nodejs/node/pull/52546) did not establish a landed speedup.
- [#44966](https://github.com/nodejs/node/pull/44966) withdrew its claimed gains when they could not be reproduced. Faster execution must be measured, not assumed.

Search: `repo:nodejs/node` — 42265; blob text performance; Blob.text arrayBuffer. Recheck activity before starting.

## Suggested fix

- Idea: Decode the existing reader’s chunks as they arrive, with separate decoder state for each text() call.
- Benefit: Could avoid the joined byte buffer and reduce peak memory.
- Risk: Preserve UTF-8 characters split across chunks, byte-order marks, errors, and concurrent calls. Creating decoders may cost more; keeping the current buffered path remains an option.

## Next step

Compare peak memory and speed for a large multipart blob before proposing the change.

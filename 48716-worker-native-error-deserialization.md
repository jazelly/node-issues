# Reconstruct genuine native Error objects from worker failures

- Area: `worker`.
- Assessment: **Small to medium fix; reproduced**.
- Checked: **2026-09-11**.
- Upstream: [#48716](https://github.com/nodejs/node/issues/48716); closed at review.
- Upstream labels: `errors`, `worker`, `stale`.
- Source: [lib/internal/error_serdes.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/error_serdes.js#L175).

## Observable problem or cost

Object.create(ctor.prototype, properties) looks like an Error to instanceof but lacks the native Error identity. Consumers using util.types.isNativeError reject errors emitted by worker failure transport.

## Narrow JavaScript change

Construct the appropriate built-in Error and restore serialized property descriptors without inventing a new wire format. Handle the already-supported AggregateError constructor separately, and preserve original stacks, causes, and descriptor attributes.

## History, objections, and ownership

[#48716](https://github.com/nodejs/node/issues/48716) welcomed a fix and later closed for inactivity. [#58605](https://github.com/nodejs/node/pull/58605) is closed; review asked about AggregateError and SuppressedError coverage rather than rejecting native reconstruction. [#62865](https://github.com/nodejs/node/pull/62865) closed after unanswered human-review and contribution-guideline feedback. A fresh change must account for current constructors instead of copying either old patch. No matching open PR was found.

## Evidence and limits

On both local runtimes, a round-tripped TypeError passed instanceof TypeError but failed util.types.isNativeError. This was a direct transport probe, not a full worker lifecycle run. [Worker error events](https://nodejs.org/api/worker_threads.html#event-error), [native-error identification](https://nodejs.org/api/util.html#utiltypesisnativeerrorvalue).

Search terms used with `repo:nodejs/node`: 48716; deserialize native errors; error_serdes. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

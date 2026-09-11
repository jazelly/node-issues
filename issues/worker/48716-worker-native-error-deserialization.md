# Reconstruct genuine native Error objects from worker failures

- Issue: [#48716](https://github.com/nodejs/node/issues/48716) — closed when checked
- Checked: 2026-09-11
- Status: No matching active fix found; constructor handling needs review (at that check)

## What's wrong?

An error received through worker failure transport can pass instanceof Error but fail util.types.isNativeError(). The decoder creates an object with the Error prototype rather than a real built-in Error.

## Where to start

- Code: [lib/internal/error_serdes.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/error_serdes.js#L175)
- Docs: [Worker error events](https://nodejs.org/api/worker_threads.html#event-error), [native-error identification](https://nodejs.org/api/util.html#utiltypesisnativeerrorvalue)
- Try it / evidence: On both local runtimes, a round-tripped TypeError passed instanceof TypeError but failed util.types.isNativeError. This was a direct transport probe, not a full worker lifecycle run. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#48716](https://github.com/nodejs/node/issues/48716) welcomed a fix and later closed for inactivity.
- [#58605](https://github.com/nodejs/node/pull/58605) closed after review asked about AggregateError and SuppressedError. [#62865](https://github.com/nodejs/node/pull/62865) closed after unanswered human-review and contribution feedback. Neither closure establishes that native reconstruction was rejected.

Search: `repo:nodejs/node` — 48716; deserialize native errors; error_serdes. Recheck activity before starting.

## Suggested fix

- Idea: Construct the appropriate built-in Error, then restore the saved properties.
- Benefit: Restores native error identity while keeping the existing transport format.
- Risk: Preserve stacks, causes, and property attributes; AggregateError needs different constructor arguments. Replacing the whole format with V8 serialization could lose behavior the custom format preserves.

## Next step

Check supported constructors and property restoration before adapting an old attempt.

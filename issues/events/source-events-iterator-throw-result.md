# Return an async-iterator result from events.on().throw()

- Issue: None found; discovered in source
- Checked: 2026-09-11
- Status: No matching active fix found; error-delivery decision needed (at that check)

## What's wrong?

The throw() method on an events.on() iterator returns undefined. Code using the async-iterator protocol expects an object after awaiting that method, so it can receive a protocol TypeError instead of the intended error.

## Where to start

- Code: [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1144)
- Docs: [events.on documentation](https://nodejs.org/api/events.html#eventsonemitter-eventname-options)
- Try it / evidence: Both local runtimes returned undefined from iterator.throw(new Error("stop")); the next next() rejected with "stop". This demonstrates the missing result and current error delivery, not an agreed replacement contract. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#41708](https://github.com/nodejs/node/pull/41708) explicitly noted the wrong return value, but the broader readline proposal stalled waiting for [#41276](https://github.com/nodejs/node/pull/41276) and a rebase. The latter landed; the return is still missing.
- [#40797](https://github.com/nodejs/node/issues/40797) discusses intentional rejection on error events. Preserve that behavior.

Search: `repo:nodejs/node` — events.on throw; events iterator throw result; 41708. Recheck activity before starting.

## Suggested fix

- Idea: Return a deliberate promise/result after handling the error.
- Benefit: Makes throw() follow the async-iterator result contract.
- Risk: Decide whether throw() or the next next() receives the error. Immediate rejection is another approach, but can change ordering or deliver the same error twice.

## Next step

Specify which call receives the error and how pending next() calls finish.

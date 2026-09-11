# Clean up a settled once promise when its callback is forwarded

- Issue: [#37828](https://github.com/nodejs/node/issues/37828) — closed when checked
- Checked: 2026-09-11
- Status: No matching active fix found; behavior agreement needed (at that check)

## What's wrong?

An event forwarder can call the callback behind events.once() directly. The promise resolves, but the original once wrapper never runs, leaving listeners attached to both emitters.

## Where to start

- Code: [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1015)
- Docs: [once and newListener documentation](https://nodejs.org/api/events.html#eventsonceemitter-name-options)
- Try it / evidence: The supplied forwarding pattern was reduced and run on both local runtimes. After the promise resolved, listenerCount("value") was 1 on both emitters. This is not a claim that arbitrary forwarding must emulate every EventEmitter behavior. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#37828](https://github.com/nodejs/node/issues/37828) was independently reproduced and closed for inactivity in July 2026. Discussion questioned how forwarding and listener removal should work; no preferred fix was agreed and no implementation PR was identified.

Search: `repo:nodejs/node` — 37828; events once forwarding cleanup; newListener resolver. Recheck activity before starting.

## Suggested fix

- Idea: Remove the original event listener when the promise resolver runs.
- Benefit: Could clean up settled listeners without changing what newListener exposes.
- Risk: Preserve removal order, error cleanup, and EventTarget support. Changing the public once wrapper instead would affect more EventEmitter behavior.

## Next step

Resolve the forwarding and removal-order question before preparing the small cleanup change.

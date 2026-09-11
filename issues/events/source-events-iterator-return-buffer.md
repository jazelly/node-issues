# Discard pending values on explicit events.on() cancellation

- Issue: None found; discovered in source
- Checked: 2026-09-11
- Status: No exact active fix found; cancellation behavior needs review (at that check)

## What's wrong?

After events.on() returns an iterator, calling return() says iteration is done. But a later next() still returns previously buffered events, and keeping the iterator can retain those event arguments in memory.

## Where to start

- Code: [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1109), [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1140), [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1228)
- Docs: [events.on contract](https://nodejs.org/api/events.html#eventsonemitter-eventname-options)
- Try it / evidence: Both local runtimes returned {done:true} from return(), then {done:false,value:["buffered"]} from next() after one event was queued. The proposed cleanup concerns explicit cancellation, not a blanket finished-first check. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#27994](https://github.com/nodejs/node/pull/27994) introduced event iteration; [#41276](https://github.com/nodejs/node/pull/41276) added efficient queues and backpressure.
- [#51091](https://github.com/nodejs/node/pull/51091) cleaned up abort listeners when iteration ends. It did not clear the buffered event values. These are related changes, not previous fixes for this exact case.

Search: `repo:nodejs/node` — events.on return; events.on buffer; events.on pause. Recheck activity before starting.

## Suggested fix

- Idea: Clear buffered values when the consumer explicitly calls return().
- Benefit: Makes cancellation finish cleanly and releases queued arguments.
- Risk: A natural close event may still need to deliver buffered values. Do not skip the queue on every kind of completion; also settle pending calls and handle any pause owned by the iterator.

## Next step

Compare explicit return with natural close, then define how each should drain or clear the queue.

# Clean up a settled once promise when its callback is forwarded

- Area: `events`.
- Assessment: **Medium fix; reproduced; semantics review**.
- Checked: **2026-09-11**.
- Upstream: [#37828](https://github.com/nodejs/node/issues/37828); closed at review.
- Upstream labels: `events`, `stale`.
- Source: [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1015).

## Observable problem or cost

A newListener-based event forwarder can call the promise resolver directly, bypassing the original emitter once wrapper. The promise settles, but its original event listener remains attached on both emitters.

## Narrow JavaScript change

Make the promise resolver remove its event listener on settlement through the existing agnostic removal helper. Keep the public newListener contract, normal once-wrapper ordering, EventTarget support, and error-listener cleanup unchanged.

## History, objections, and ownership

[#37828](https://github.com/nodejs/node/issues/37828) was independently reproduced. Discussion questioned how exposed listeners and wrappers should interact, and did not settle the preferred fix. It closed stale in July 2026, with no linked implementation or current claim found. This narrower cleanup avoids changing what newListener exposes, but still needs events-maintainer agreement.

## Evidence and limits

The supplied forwarding pattern was reduced and run on both local runtimes. After the promise resolved, listenerCount("value") was 1 on both emitters. This is not a claim that arbitrary forwarding must emulate every EventEmitter behavior. [once and newListener documentation](https://nodejs.org/api/events.html#eventsonceemitter-name-options).

Search terms used with `repo:nodejs/node`: 37828; events once forwarding cleanup; newListener resolver. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

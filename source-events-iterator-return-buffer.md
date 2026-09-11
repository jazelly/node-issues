# Discard pending values on explicit events.on() cancellation

- Area: `events`.
- Assessment: **Small to medium fix; reproduced**.
- Checked: **2026-09-11**.
- Upstream issue: none identified; related historical PRs are references, not a claimed issue number.
- Source: [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1109), [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1140), [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1228).

## Observable problem or cost

Explicit return() reports done, but subsequent next() calls consume buffered events before checking finished. Keeping the iterator also retains queued argument objects after cancellation.

## Narrow JavaScript change

Distinguish explicit consumer cancellation from natural close-event draining. On explicit return, release queued values and finish pending next calls. Preserve buffered draining on natural completion; review how an emitter paused by this iterator should be released.

## History, objections, and ownership

[#27994](https://github.com/nodejs/node/pull/27994) introduced event async iteration; [#41276](https://github.com/nodejs/node/pull/41276) added efficient queues and backpressure, and [#51091](https://github.com/nodejs/node/pull/51091) cleaned up abort listeners on completion. Those landed improvements do not clear this value queue. Searches for events.on return/buffering found no exact open fix. Returning done is the observed behavior; changing post-return draining still deserves compatibility review.

## Evidence and limits

Both local runtimes returned {done:true} from return(), then {done:false,value:["buffered"]} from next() after one event was queued. The proposed cleanup concerns explicit cancellation, not a blanket finished-first check. [events.on contract](https://nodejs.org/api/events.html#eventsonemitter-eventname-options).

Search terms used with `repo:nodejs/node`: events.on return; events.on buffer; events.on pause. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

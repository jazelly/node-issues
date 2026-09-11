# Return an async-iterator result from events.on().throw()

- Area: `events`.
- Assessment: **Small fix; reproduced**.
- Checked: **2026-09-11**.
- Upstream issue: none identified; related historical PRs are references, not a claimed issue number.
- Source: [lib/events.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/events.js#L1144).

## Observable problem or cost

The iterator throw method runs errorHandler and returns undefined. Async-iterator consumers that delegate to this method expect its awaited result to be an object, so the missing result can replace the intended error with a protocol TypeError.

## Narrow JavaScript change

Give throw a deliberate promise/result contract after handling the error, matching the existing queued-error behavior. Decide explicitly which call receives the original error; avoid delivering it twice or dropping pending next calls.

## History, objections, and ownership

The body of the old [#41708](https://github.com/nodejs/node/pull/41708) explicitly identified this wrong return value. That broad readline proposal stalled waiting for [#41276](https://github.com/nodejs/node/pull/41276) and a rebase; the latter landed, but the current throw method still lacks a return. [#40797](https://github.com/nodejs/node/issues/40797) concerns intentional error-event rejection, which this change must preserve. No matching open implementation was found.

## Evidence and limits

Both local runtimes returned undefined from iterator.throw(new Error("stop")); the next next() rejected with "stop". This demonstrates the missing result and current error delivery, not an agreed replacement contract. [events.on documentation](https://nodejs.org/api/events.html#eventsonemitter-eventname-options).

Search terms used with `repo:nodejs/node`: events.on throw; events iterator throw result; 41708. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

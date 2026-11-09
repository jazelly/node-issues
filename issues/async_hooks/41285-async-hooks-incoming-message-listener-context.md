# Bind incoming-message listeners to the request's async context

- Issue: [#41285](https://github.com/nodejs/node/issues/41285) — open when checked
- Checked: 2026-11-09
- Status: No active fix found; behavior decision needed first

## What's wrong?

A server handler that calls `storage.run(store, () => req.on('end', cb))` gets
`undefined` from `storage.getStore()` inside `cb`. Body events are emitted from
the HTTP parser callback, so listeners run in the parser's context rather than
the context where they were registered. Users read this as ALS being unreliable
and some keep using `domain` because of it.

## Where to start

- Code: [lib/internal/http/common.js `parserOnBody` / `parserOnMessageComplete`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/http/common.js#L128-L158),
  [lib/internal/http/server.js `parserOnIncoming`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/http/server.js#L1348)
- Docs: [AsyncLocalStorage troubleshooting](https://nodejs.org/api/async_context.html#troubleshooting-context-loss),
  [`EventEmitterAsyncResource`](https://nodejs.org/api/events.html#class-eventseventemitterasyncresource)
- Try it: On `v28.0.0-pre` (`128f247ea4cb`, macOS arm64), the issue's server
  returns `null` from the `'end'` listener. A variant that writes the body in
  two chunks shows the first `'data'` event sees the store, while the second
  `'data'` and `'end'` see `undefined`.

## What's been tried?

- 2021–2026 — [#41285](https://github.com/nodejs/node/issues/41285): reporters keep
  confirming it, most recently in September 2026. The documented workaround is
  `AsyncResource.bind()` or `EventEmitterAsyncResource`. No maintainer decision on
  whether core should change.
- 2022 — [#41978](https://github.com/nodejs/node/issues/41978) discussed the same
  parser reuse and a related TODO in the server code. Its `enterWith()` leak did
  not reproduce on main during this check.

Search: `repo:nodejs/node` AsyncLocalStorage IncomingMessage; ALS req.on end;
context loss http. Closed PRs included; no matching fix found.

## Suggested fix

- Idea: Treat this as documentation first. Add an HTTP example to the
  troubleshooting section showing that `IncomingMessage` events follow the
  parser, and how to bind listeners.
- Benefit: Answers the most common ALS report without changing semantics.
- Risk: Does not change runtime behavior, so users who expect implicit binding
  are still surprised.
- Alternative (optional): Capture the frame active when the `'request'` handler
  starts and emit body events inside it. Every listener would then see the
  handler's context, which may break code that relies on the current behavior.
  Needs maintainer agreement and a performance check on the request hot path.

## Next step

Ask on the issue which behavior maintainers want, linking the two-chunk
observation, then draft the docs example.

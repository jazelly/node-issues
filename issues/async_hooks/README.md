# AsyncLocalStorage: history, current design, and open work

Checked: 2026-11-09 against Node.js main at
[`128f247ea4cb`](https://github.com/nodejs/node/commit/128f247ea4cb52da405f6cc35dec46c088be8af3)
(`v28.0.0-pre`). Runtime observations below come from a local build of that
commit on macOS arm64.

This page is background for the `async_hooks` entries. It is not a candidate
entry itself; the candidates are listed at the end.

## History

| When | Change |
| --- | --- |
| 2020 | `AsyncLocalStorage` (ALS) lands on top of `async_hooks` and `executionAsyncResource()` (v13.10 / v12.17). Every async resource carries the store; promises are tracked by a promise hook. |
| 2020–2021 | Performance work on the promise-hook fast path; part of the API is marked stable in v16.4; `run()` moves to the resource stack. |
| 2023 | `AsyncLocalStorage.bind()` and `snapshot()` are added; the experimental `onPropagate` option is added and removed. |
| 2024-07 | V8 adds `ContinuationPreservedEmbedderData` (CPED). ALS is rewritten without `async_hooks` as `AsyncContextFrame`, behind `--experimental-async-context-frame` (v22.7). |
| 2024-11 | `AsyncContextFrame` becomes the default ALS implementation ([nodejs/node#55552](https://github.com/nodejs/node/pull/55552)); shipped in v24.0. |
| 2025 | `defaultValue` and `name` options; fixes for instance isolation; `stream.finished()` and `queueMicrotask()` capture context only when needed. |
| 2026 | `withScope()` for `using` scopes; `domain` is reimplemented on ALS and runtime-deprecated ([nodejs/node#61095](https://github.com/nodejs/node/pull/61095)); the legacy implementation and `--no-async-context-frame` are removed ([nodejs/node#63641](https://github.com/nodejs/node/pull/63641), in 27.0.0-alpha); test_runner and REPL stop depending on `async_hooks` / `domain`. |

## Current design

The whole implementation is small:

- [lib/internal/async_context_frame.js](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/async_context_frame.js):
  `AsyncContextFrame extends SafeMap`, keyed by ALS instance.
- [lib/internal/async_local_storage/async_context_frame.js](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/async_local_storage/async_context_frame.js):
  the public class.
- [lib/internal/async_local_storage/run_scope.js](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/async_local_storage/run_scope.js):
  the `withScope()` disposable.

How it works:

- `enterWith(data)` copies the current frame, adds one entry, and stores the new
  frame in V8 CPED.
- V8 saves and restores CPED around promise reactions, so `await` and `then`
  propagate context without a promise hook.
- Callback-based resources (timers, `AsyncResource`, libuv handles through
  `MakeCallback`) capture `AsyncContextFrame.current()` when created and swap it
  in around the callback.
- `run()` is `enterWith(data)` plus `enterWith(prior)` in `finally`, and skips
  both when `prior` and `data` are the same value.

Consequences that explain most open reports:

1. `EventEmitter` listeners run in the context of whoever calls `emit()`, not
   the context in which they were registered.
2. `disable()` removes the key from the current frame object in place. Callbacks
   that share that frame lose the value; frames already copied from it keep it.
3. Any long-lived object that holds a frame keeps every store in that frame
   alive.

## Direction

- **Deprecate `async_hooks.createHook()`.** [nodejs/node#61030](https://github.com/nodejs/node/pull/61030)
  proposed `--enable-async-hooks` with hooks off by default. Reviewers asked for
  a deprecation cycle, which became [nodejs/node#67271](https://github.com/nodejs/node/pull/67271):
  DEP0212 runtime warning in the 27 alpha, then require the flag in 27 Current.
  Inspector and trace events move to an internal hook.
- **Remove remaining async-id bookkeeping cost** when no hook is installed
  ([nodejs/node#66344](https://github.com/nodejs/node/pull/66344),
  [nodejs/node#66511](https://github.com/nodejs/node/pull/66511), and drafts
  [nodejs/node#66509](https://github.com/nodejs/node/pull/66509),
  [nodejs/node#66510](https://github.com/nodejs/node/pull/66510)).
- **Observability through ALS and `diagnostics_channel`**, including the built-in
  OpenTelemetry discussion ([nodejs/node#57992](https://github.com/nodejs/node/issues/57992),
  [nodejs/node#66587](https://github.com/nodejs/node/pull/66587)).
- **TC39 AsyncContext.** CPED is the engine hook intended for it; ALS may
  eventually wrap `AsyncContext.Variable`. The proposal's current stage was not
  checked.

## Upstream tracker

Status as seen on 2026-11-09. These are not candidates unless linked below.

| Item | Status |
| --- | --- |
| [nodejs/node#41285](https://github.com/nodejs/node/issues/41285) listener context loss | Reproduced on main. Candidate below. |
| [nodejs/node#41978](https://github.com/nodejs/node/issues/41978) store shared across keep-alive requests | The `enterWith()` leak from the report did **not** reproduce on main: a second request on the same socket saw `undefined`. Possibly fixed by `AsyncContextFrame`; worth confirming upstream. |
| [nodejs/node#48651](https://github.com/nodejs/node/issues/48651) store retained after `console.log` | Reproduced on main with a log inside `run()`. [Existing candidate](48651-async-hooks-lazy-stdio-context-retention.md). |
| [nodejs/node#67181](https://github.com/nodejs/node/issues/67181) idle keep-alive socket retains store | Reproduced on main. Approved fix PR open. [Candidate](../http/67181-http-agent-idle-socket-context-retention.md). |
| [nodejs/node#63186](https://github.com/nodejs/node/issues/63186) vm `SourceTextModule` retained via frames | Waiting on an upstream V8 change. Not a JS fix. |
| [nodejs/node#66439](https://github.com/nodejs/node/issues/66439) vm timeout corrupts async hook stack | Claimed in comments, no PR seen. Mostly C++. |
| [nodejs/node#10843](https://github.com/nodejs/node/issues/10843) what replaces domain | `domain` now runs on ALS and is runtime-deprecated; the question is largely answered. |
| [nodejs/node#67054](https://github.com/nodejs/node/pull/67054) keep context in `pipeTo()` pump | Approved; left the commit queue on conflicts. |
| [nodejs/node#64147](https://github.com/nodejs/node/pull/64147) context for thrown microtasks | Test-only; approved. |
| [nodejs/node#58065](https://github.com/nodejs/node/pull/58065) discourage `disable()` | Closed as stale. [Candidate](source-async-hooks-disable-shared-frame.md). |
| [nodejs/node#58229](https://github.com/nodejs/node/pull/58229) channel for `AsyncContextFrame.set` | Closed as stale with approvals. New API, so outside this notebook's scope. Related drafts: [nodejs/node#62649](https://github.com/nodejs/node/pull/62649), [nodejs/node#63163](https://github.com/nodejs/node/pull/63163). |
| [nodejs/node#57938](https://github.com/nodejs/node/pull/57938), [nodejs/node#59621](https://github.com/nodejs/node/pull/59621) remove async tracking from the HTTP parser | Closed as stale after a review objection about a data path that bypasses JS. Mostly C++. |

## Candidates in this folder

- [Bind incoming-message listeners to the request's async context](41285-async-hooks-incoming-message-listener-context.md)
- [Avoid retaining request context in lazily created global stdio](48651-async-hooks-lazy-stdio-context-retention.md)
- [Make `disable()` match its documentation under AsyncContextFrame](source-async-hooks-disable-shared-frame.md)

Related entry in another folder:
[Release ALS context from idle keep-alive sockets](../http/67181-http-agent-idle-socket-context-retention.md).

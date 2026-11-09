# Release ALS context from idle keep-alive sockets

- Issue: [#67181](https://github.com/nodejs/node/issues/67181) — open when checked
- Checked: 2026-11-09
- Status: Active fix [#67233](https://github.com/nodejs/node/pull/67233), approved by one collaborator; not expected to be backported to v26

## What's wrong?

When an `http.Agent` reuses a keep-alive socket, it resets the socket's handle
inside the current request's async context. When the socket returns to the free
pool, nothing resets it again, so every idle socket keeps the last request's
`AsyncLocalStorage` stores alive until the socket is reused or destroyed.

## Where to start

- Code: [lib/internal/http/agent.js `asyncResetHandle` and its callers](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/http/agent.js#L702-L710),
  [reuse path](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/http/agent.js#L414-L418),
  [`'free'` handler](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/http/agent.js#L203)
- Docs: [AsyncLocalStorage](https://nodejs.org/api/async_context.html#class-asynclocalstorage),
  [`http.Agent` keep-alive](https://nodejs.org/api/http.html#new-agentoptions)
- Try it: On `v28.0.0-pre` (`128f247ea4cb`, macOS arm64, `--expose-gc`): create the
  socket outside any store, make a second request inside
  `storage.run({ big }, ...)`, wait for it to finish, and run `gc()`. A `WeakRef` to
  the store still resolves while the socket sits in `agent.freeSockets`.

## What's been tried?

- 2026-11 — [#67181](https://github.com/nodejs/node/issues/67181): a maintainer noted
  the area is sensitive and the approach needs review.
- 2026-11 — [#67233](https://github.com/nodejs/node/pull/67233): separates the
  request context from the connection lifetime and also covers socket timers and
  the TCP handle under TLS. Approved; the reviewer said it cannot be backported
  to v26.
- 2022 — [#41978](https://github.com/nodejs/node/issues/41978): an earlier report
  about context shared across keep-alive requests on the server side.

## Suggested fix

- Idea: Follow #67233. If it stalls, the minimal version is to reset the handle
  in the root context when the socket enters `freeSockets`.
- Benefit: Idle pools stop holding request data, which matters for services that
  keep large stores per request.
- Risk: Hooks and ALS users may observe a different context for socket-level
  events between requests. Must cover TLS sockets and timers.

## Next step

Follow #67233 to landing. If it stalls, check whether the reviewer's backport
concern also applies to the minimal reset.

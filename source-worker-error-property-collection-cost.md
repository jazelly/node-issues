# Collect inherited error descriptors without repeated prefix copies

- Area: `worker`.
- Assessment: **Small to medium performance candidate; isolated measurement**.
- Checked: **2026-09-11**.
- Upstream issue: none identified; related historical PRs are references, not a claimed issue number.
- Source: [lib/internal/error_serdes.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/error_serdes.js#L56).

## Observable problem or cost

TryGetAllProperties allocates a descriptor map at each prototype level and copies the complete parent result into it. Deep error-class hierarchies repeatedly copy the same accumulated entries before serialization.

## Narrow JavaScript change

Pass one accumulator through the existing base-to-derived traversal. Preserve property override order, getter receiver and evaluation order, cause-cycle bookkeeping, and the handling of throwing getters. Avoid redesigning the transport or changing descriptor semantics.

## History, objections, and ownership

[#26145](https://github.com/nodejs/node/pull/26145) established why throwing getters need special handling; [#47925](https://github.com/nodejs/node/pull/47925) explicitly discussed descriptor and allocation concerns. [#48918](https://github.com/nodejs/node/issues/48918) rejected broad accommodation of Object.prototype manipulation; this proposal does not add that support. [#58075](https://github.com/nodejs/node/pull/58075) addressed cause recursion and must remain intact. No exact active accumulator optimization was found.

## Evidence and limits

An isolated copy of the pinned helper was compared with a shared-accumulator variant on local v27. For 2,000 collections at prototype depths 2/8/32 with eight added properties per level, observed times were 15.42/70.72/1031.98 ms versus 6.39/10.63/44.12 ms. Result descriptors matched for those fixtures. These are synthetic single-run helper measurements, not whole-worker performance or full semantic verification. [Worker error events](https://nodejs.org/api/worker_threads.html#event-error).

Search terms used with `repo:nodejs/node`: TryGetAllProperties; serializeError performance; error_serdes. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

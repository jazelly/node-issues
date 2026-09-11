# Collect inherited error descriptors without repeated prefix copies

- Issue: None found; discovered in source
- Checked: 2026-09-11
- Status: No exact active fix found; isolated speedup needs broader validation (at that check)

## What's wrong?

The error serializer collects properties from each level of an error’s prototype chain. At every level it copies the parent’s complete result again, repeating work for deep error-class hierarchies.

## Where to start

- Code: [lib/internal/error_serdes.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/error_serdes.js#L56)
- Docs: [Worker error events](https://nodejs.org/api/worker_threads.html#event-error)
- Try it / evidence: An isolated copy of the pinned helper was compared with a shared-accumulator variant on local v27. For 2,000 collections at prototype depths 2/8/32 with eight added properties per level, observed times were 15.42/70.72/1031.98 ms versus 6.39/10.63/44.12 ms. Result descriptors matched for those fixtures. These are synthetic single-run helper measurements, not whole-worker performance or full semantic verification. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#26145](https://github.com/nodejs/node/pull/26145) and [#47925](https://github.com/nodejs/node/pull/47925) explain why getters and property attributes need careful handling. [#58075](https://github.com/nodejs/node/pull/58075) fixed cause recursion.
- [#48918](https://github.com/nodejs/node/issues/48918) rejected broad support for Object.prototype manipulation. This proposal does not add that support. These are precedents, not previous accumulator fixes.

Search: `repo:nodejs/node` — TryGetAllProperties; serializeError performance; error_serdes. Recheck activity before starting.

## Suggested fix

- Idea: Collect properties into one shared result while keeping the current traversal order.
- Benefit: Avoids repeatedly copying the properties already collected.
- Risk: Preserve overrides, getter order and receiver, throwing getters, and circular causes. A fully iterative rewrite is larger than needed; helper timings do not prove whole-worker gains.

## Next step

Check property and cause behavior, then measure actual error transport with the shared result.

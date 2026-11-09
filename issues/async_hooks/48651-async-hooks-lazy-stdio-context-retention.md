# Avoid retaining request context in lazily created global stdio

- Issue: [#48651](https://github.com/nodejs/node/issues/48651) — open when checked
- Checked: 2026-09-11
- Status: No matching active fix found; current memory-retention diagnosis needed (at that check)

## What's wrong?

The first console call can create a global output stream inside a request’s AsyncLocalStorage context. Because that stream lives for the whole process, it may keep the request’s data in memory after the request ends.

## Where to start

- Code: [lib/internal/bootstrap/switches/is_main_thread.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/bootstrap/switches/is_main_thread.js#L151), [lib/internal/console/constructor.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/console/constructor.js#L208)
- Docs: [AsyncLocalStorage documentation](https://nodejs.org/api/async_context.html#class-asynclocalstorage)
- Try it / evidence: Current source still lazily constructs and caches stdout; the exact retention path was not proven with a fresh heap snapshot. Treat this as a diagnosis-first JS candidate, not a confirmed fix for every reported GC symptom. [Runtime and research limits](../../RESEARCH.md).
- Re-check 2026-11-09: on `v28.0.0-pre` (`128f247ea4cb`, macOS arm64, `--expose-gc`), a store object is collected when `run()` contains no log, and retained when `run()` contains one `console.log()`. This matches the May 2026 report; the retaining path is still unproven. See the [AsyncLocalStorage overview](README.md).

## What's been tried?

- [#48651](https://github.com/nodejs/node/issues/48651): a maintainer supported creating long-lived global resources in the root context. A May 2026 report showed that newer Node versions changed which logging sequence retained data, so the old diagnosis needs checking again.

Search: `repo:nodejs/node` — 48651; AsyncLocalStorage console retention; lazy stdio context. Recheck activity before starting.

## Suggested fix

- Idea: Create the global output resource outside the request context, using Node’s existing context helpers.
- Benefit: Could release request data while keeping output streams lazily created.
- Risk: User writes and callbacks must still run in the right context. Initializing all output streams at startup is another option, but adds startup work.

## Next step

Find which resource retains the request data in the current implementation before choosing a fix.

# Avoid retaining request context in lazily created global stdio

- Area: `async_hooks`.
- Kind: `resource-lifecycle`.
- Assessment: **Medium investigation; current-source follow-up**.
- Last researched: **2026-09-11**.
- Upstream: [#48651](https://github.com/nodejs/node/issues/48651); open at review.
- Upstream labels: `async_hooks`.
- Source: [lib/internal/bootstrap/switches/is_main_thread.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/bootstrap/switches/is_main_thread.js#L151), [lib/internal/console/constructor.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/console/constructor.js#L208).

## Problem and impact

The first console access can lazily create a long-lived stdio resource inside a request AsyncLocalStorage context. The global resource can outlive that request and retain its store.

## Evidence and reproduction

Current source still lazily constructs and caches stdout; the exact retention path was not proven with a fresh heap snapshot. Treat this as a diagnosis-first JS candidate, not a confirmed fix for every reported GC symptom. [AsyncLocalStorage documentation](https://nodejs.org/api/async_context.html#class-asynclocalstorage).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| 2026-09-11 (snapshot) | No implementation PR identified in the recorded search | No identified attempt | Unknown beyond the search scope | Issue discussion below is evidence; absence of a PR is not approval or proof of no work. |

[#48651](https://github.com/nodejs/node/issues/48651) contains maintainer support for root-context creation of immortal lazy resources. A May 2026 report notes that newer Node changed which outer/inner logging sequence retains the object. That matters because current AsyncContextFrame behavior must be examined instead of assuming the old async_hooks implementation. No direct open fix or recent implementation claim was found.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Create only immortal global stdio resources in the root context | Targets retention while keeping lazy creation. | Must preserve the context of user writes and callbacks. |
| B: Eagerly initialize global stdio during bootstrap | Avoids first access occurring inside a request context. | Changes startup cost and initialization timing for programs that never use stdio. |

## Recommendation and boundaries

First prove the current retention path; then prefer scoped root-context creation.

Initialize only the immortal global stdio resource in the appropriate root context, using existing internal context machinery. Keep user writes and callbacks in their intended context; do not globally disable AsyncLocalStorage or eagerly initialize every stream.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Capture the current retaining resource/context path before selecting a root-context initialization change.

Search terms used with `repo:nodejs/node`: 48651; AsyncLocalStorage console retention; lazy stdio context. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [AsyncLocalStorage documentation](https://nodejs.org/api/async_context.html#class-asynclocalstorage)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

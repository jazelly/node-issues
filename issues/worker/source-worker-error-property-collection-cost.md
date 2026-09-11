# Collect inherited error descriptors without repeated prefix copies

- Area: `worker`.
- Kind: `performance`.
- Assessment: **Small to medium performance candidate; isolated measurement**.
- Last researched: **2026-09-11**.
- Upstream: None identified; related historical PRs are references, not a claimed issue number.
- Upstream labels: Not applicable.
- Source: [lib/internal/error_serdes.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/error_serdes.js#L56).

## Problem and impact

TryGetAllProperties allocates a descriptor map at each prototype level and copies the complete parent result into it. Deep error-class hierarchies repeatedly copy the same accumulated entries before serialization.

## Evidence and reproduction

An isolated copy of the pinned helper was compared with a shared-accumulator variant on local v27. For 2,000 collections at prototype depths 2/8/32 with eight added properties per level, observed times were 15.42/70.72/1031.98 ms versus 6.39/10.63/44.12 ms. Result descriptors matched for those fixtures. These are synthetic single-run helper measurements, not whole-worker performance or full semantic verification. [Worker error events](https://nodejs.org/api/worker_threads.html#event-error).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #26145](https://github.com/nodejs/node/pull/26145) | Error-handling precedent | Landed | Preserve behavior when stack or other getters throw. |
| Unknown in notebook | [PR #47925](https://github.com/nodejs/node/pull/47925) | Transport precedent | Landed | Introduced broader error transport; review covered getters, descriptors, and allocation cost. |
| Unknown in notebook | [PR #58075](https://github.com/nodejs/node/pull/58075) | Related correctness fix | Landed | Preserve cause-cycle handling; do not regress stack-overflow protection. |

[#26145](https://github.com/nodejs/node/pull/26145) established why throwing getters need special handling; [#47925](https://github.com/nodejs/node/pull/47925) explicitly discussed descriptor and allocation concerns. [#48918](https://github.com/nodejs/node/issues/48918) rejected broad accommodation of Object.prototype manipulation; this proposal does not add that support. [#58075](https://github.com/nodejs/node/pull/58075) addressed cause recursion and must remain intact. No exact active accumulator optimization was found.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Share one descriptor accumulator in the existing traversal | Avoids repeated prefix copies while retaining base-to-derived ordering. | Getter order, causes, cycles, and descriptor overrides must remain identical. |
| B: Replace recursive collection with a fully iterative traversal | Can additionally avoid prototype-recursion stack growth. | Larger rewrite and ordering risk than needed to remove prefix copies. |

## Recommendation and boundaries

Prefer the shared accumulator; the existing isolated timings do not prove full transport correctness or speedup.

Pass one accumulator through the existing base-to-derived traversal. Preserve property override order, getter receiver and evaluation order, cause-cycle bookkeeping, and the handling of throwing getters. Avoid redesigning the transport or changing descriptor semantics.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Check descriptor/getter/cause equivalence and measure actual error transport before interpreting the isolated timing gain.

Search terms used with `repo:nodejs/node`: TryGetAllProperties; serializeError performance; error_serdes. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Worker error events](https://nodejs.org/api/worker_threads.html#event-error)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

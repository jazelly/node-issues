# Handle zero-duration timerify histogram samples

- Area: `perf_hooks`.
- Kind: `correctness`.
- Assessment: **Small fix; reproduced**.
- Last researched: **2026-09-11**.
- Upstream: [#41641](https://github.com/nodejs/node/issues/41641); open at review.
- Upstream labels: `perf_hooks`.
- Source: [lib/internal/perf/timerify.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/perf/timerify.js#L37).

## Problem and impact

Two clock reads can be equal. The duration becomes zero, while the histogram requires a positive sample. A successful, very fast wrapped function consequently throws ERR_OUT_OF_RANGE.

## Evidence and reproduction

An empty timerified function with a histogram threw on iteration 999 on v24.15.0 and 1034 on the local v27 build. Those iteration counts are observations, not deterministic thresholds. [timerify documentation](https://nodejs.org/api/perf_hooks.html#performancetimerifyfn-options).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| 2026-08-27 (closure) | [PR #62365](https://github.com/nodejs/node/pull/62365) | Previous fix attempt | Closed for inactivity | Minimum histogram sample approach; no design objection recorded. |
| 2026-09-11 (snapshot) | [PR #65120](https://github.com/nodejs/node/pull/65120) | Nearby active work | Open at research | Changes rejected-thenable timing, not the zero-sample bound. |

[#41641](https://github.com/nodejs/node/issues/41641) has reports through June 2026. The exact minimum-sample approach was attempted in [#62365](https://github.com/nodejs/node/pull/62365), referencing #54803; that PR was closed for inactivity without a design objection in the returned reviews. [#65120](https://github.com/nodejs/node/pull/65120) is an active, different fix for rejected thenables. Its diff changes when processComplete is called, not the zero-sample calculation; coordinate this nearby change before implementation.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Clamp only the recorded histogram sample to at least 1 ns | Local fix; leaves the actual PerformanceEntry duration intact. | Histogram values become quantized at the lower bound. |
| B: Use a higher-precision clock throughout timerify | Can reduce zero-duration observations at their source. | Broader timing and performance change; still must respect histogram limits. |

## Recommendation and boundaries

Prefer a local histogram bound; coordinate with the nearby thenable PR.

Apply a minimum representable positive sample when recording the histogram, while retaining the actual duration in the PerformanceEntry. Keep throwing and rejected-function behavior outside this change.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Recheck PR #65120 and verify the histogram lower-bound contract before adapting the dormant minimum-sample approach.

Search terms used with `repo:nodejs/node`: 41641; timerify histogram; timerify minimum sample. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [timerify documentation](https://nodejs.org/api/perf_hooks.html#performancetimerifyfn-options)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

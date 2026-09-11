# Handle zero-duration timerify histogram samples

- Area: `perf_hooks`.
- Assessment: **Small fix; reproduced**.
- Checked: **2026-09-11**.
- Upstream: [#41641](https://github.com/nodejs/node/issues/41641); open at review.
- Upstream labels: `perf_hooks`.
- Source: [lib/internal/perf/timerify.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/perf/timerify.js#L37).

## Observable problem or cost

Two clock reads can be equal. The duration becomes zero, while the histogram requires a positive sample. A successful, very fast wrapped function consequently throws ERR_OUT_OF_RANGE.

## Narrow JavaScript change

Apply a minimum representable positive sample when recording the histogram, while retaining the actual duration in the PerformanceEntry. Keep throwing and rejected-function behavior outside this change.

## History, objections, and ownership

[#41641](https://github.com/nodejs/node/issues/41641) has reports through June 2026. The exact minimum-sample approach was attempted in [#62365](https://github.com/nodejs/node/pull/62365), referencing #54803; that PR was closed for inactivity without a design objection in the returned reviews. [#65120](https://github.com/nodejs/node/pull/65120) is an active, different fix for rejected thenables. Its diff changes when processComplete is called, not the zero-sample calculation; coordinate this nearby change before implementation.

## Evidence and limits

An empty timerified function with a histogram threw on iteration 999 on v24.15.0 and 1034 on the local v27 build. Those iteration counts are observations, not deterministic thresholds. [timerify documentation](https://nodejs.org/api/perf_hooks.html#performancetimerifyfn-options).

Search terms used with `repo:nodejs/node`: 41641; timerify histogram; timerify minimum sample. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

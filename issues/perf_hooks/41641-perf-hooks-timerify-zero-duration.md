# Handle zero-duration timerify histogram samples

- Issue: [#41641](https://github.com/nodejs/node/issues/41641) — open when checked
- Checked: 2026-09-11
- Status: No matching zero-sample fix active; coordinate with PR #65120 (at that check)

## What's wrong?

A very fast function can start and finish at the same clock reading. timerify() records a zero duration, but its histogram requires a positive value, so the successful function call unexpectedly throws.

## Where to start

- Code: [lib/internal/perf/timerify.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/perf/timerify.js#L37)
- Docs: [timerify documentation](https://nodejs.org/api/perf_hooks.html#performancetimerifyfn-options)
- Try it / evidence: An empty timerified function with a histogram threw on iteration 999 on v24.15.0 and 1034 on the local v27 build. Those iteration counts are observations, not deterministic thresholds. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#41641](https://github.com/nodejs/node/issues/41641) has reports through June 2026. [#62365](https://github.com/nodejs/node/pull/62365) tried a minimum sample and closed for inactivity on 2026-08-27, without a recorded design objection.
- [#65120](https://github.com/nodejs/node/pull/65120) was active at the check. It changes timing for rejected thenables, not the zero-sample calculation; coordinate the nearby edits.

Search: `repo:nodejs/node` — 41641; timerify histogram; timerify minimum sample. Recheck activity before starting.

## Suggested fix

- Idea: Record at least 1 nanosecond in the histogram while keeping the actual duration in the PerformanceEntry.
- Benefit: Prevents a zero-duration sample from turning a successful call into an error.
- Risk: The minimum sample rounds up zero. Changing the clock itself is broader and may add overhead; preserve existing thrown/rejected-function behavior.

## Next step

Recheck the nearby PR and histogram limits before adapting the minimum-sample approach.

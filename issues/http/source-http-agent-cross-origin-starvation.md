# Continue looking for an eligible queued HTTP origin

- Area: `http`.
- Kind: `correctness`.
- Assessment: **Medium fix; reproduced**.
- Last researched: **2026-09-11**.
- Upstream: None identified; related historical PRs are references, not a claimed issue number.
- Upstream labels: Not applicable.
- Source: [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L574).

## Problem and impact

When a socket closes with no same-origin waiter, removeSocket scans other origins. Encountering any active socket for the first queued origin breaks the entire scan. Another origin can remain blocked while the agent has spare global capacity.

## Evidence and reproduction

A local three-origin HTTP run used maxSockets=1 and maxTotalSockets=2. After origin C completed, A still occupied one socket; A2 and B1 were queued. After 100 ms the agent had totalSocketCount=1, two pending origins, and B had received no request. All temporary servers and requests were closed. [Agent limits](https://nodejs.org/api/http.html#new-agentoptions).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #33617](https://github.com/nodejs/node/pull/33617) | Introducing change | Landed | Review required per-origin limits and avoiding duplicate dispatch; preserve both. |

[#33617](https://github.com/nodejs/node/pull/33617) introduced maxTotalSockets. Review specifically required per-origin limits and warned about dispatching a queued request twice; those constraints explain the guard, but not stopping the whole search. [#40452](https://github.com/nodejs/node/issues/40452) and its landed fix address a different historical counter bug. Current source includes that counter fix. No exact active starvation fix was found.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Continue scanning past origins that cannot take a socket | Uses existing queues and prevents spare-capacity starvation. | Must honor both limits and avoid dispatching a request twice. |
| B: Replace per-origin selection with a global FIFO queue | Would make cross-origin fairness explicit. | Adds scheduling machinery and changes priority semantics beyond this bug. |

## Recommendation and boundaries

Prefer the existing scan with correct eligibility checks; a scheduler redesign is out of scope.

Skip an ineligible origin and continue the scan, checking the actual per-origin and total limits. Preserve same-origin priority and existing free-event request removal. This is not a new global FIFO scheduler.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Verify eligibility against both socket limits and existing free-event removal before changing the cross-origin scan.

Search terms used with `repo:nodejs/node`: removeSocket; maxTotalSockets starvation; maxTotalSockets pending requests. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Agent limits](https://nodejs.org/api/http.html#new-agentoptions)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

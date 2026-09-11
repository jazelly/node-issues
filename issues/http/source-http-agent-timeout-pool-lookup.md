# Limit idle-socket timeout lookup to its owning pool

- Area: `http`.
- Kind: `performance`.
- Assessment: **Small performance candidate; source checked**.
- Last researched: **2026-09-11**.
- Upstream: None identified; related historical PRs are references, not a claimed issue number.
- Upstream labels: Not applicable.
- Source: [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L461), [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L543).

## Problem and impact

Each socket timeout allocates all free-pool keys and searches their arrays to determine whether that socket is idle. The socket already has an owning agent key recorded at creation, so a many-origin agent repeatedly scans unrelated pools.

## Evidence and reproduction

Pinned source records options._agentKey in createSocket and then does a cross-pool scan in onTimeout. This establishes avoidable search/allocation work, not an application speedup. Keep the proposal only if a many-origin workload benefits without changing timeout/reuse behavior. [Agent freeSockets](https://nodejs.org/api/http.html#agentfreesockets).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #32000](https://github.com/nodejs/node/pull/32000) | Correctness precedent | Landed | Protects reuse from timed-out or destroyed idle sockets. |

[#32000](https://github.com/nodejs/node/pull/32000) deliberately handles reuse races involving timed-out and destroyed sockets. The current TODO about destroying active sockets is a separate behavioral proposal and is excluded. Search results for freeSockets/timeout and removeSocket showed no exact active pool-lookup optimization; current Agent-related open PRs were also screened.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Look up the socket in its recorded owning pool | Avoids allocating all pool keys and searching unrelated origins. | Must remain correct for custom agents and HTTPS pool identities. |
| B: Retain the cross-pool scan | Does not add reliance on the recorded pool key. | Work and temporary allocations grow with unrelated pools. |

## Recommendation and boundaries

Prefer the owning-key lookup only after equivalence and a meaningful workload benefit are demonstrated.

Look up the recorded owning pool and check that array for the socket. Keep the current rule that only idle sockets are destroyed here, and verify that custom Agent subclasses and HTTPS pool keys still refer to the recorded owner. No additional index or cache is needed.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Compare owning-pool lookup with the existing scan using custom agents and a many-origin workload.

Search terms used with `repo:nodejs/node`: freeSockets timeout; agent onTimeout; removeSocket. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Agent freeSockets](https://nodejs.org/api/http.html#agentfreesockets)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

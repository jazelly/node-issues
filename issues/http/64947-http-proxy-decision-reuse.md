# Reuse a proxy-routing decision within one request

- Area: `http`.
- Kind: `performance`.
- Assessment: **Small to medium performance candidate**.
- Last researched: **2026-09-11**.
- Upstream: [#64947](https://github.com/nodejs/node/issues/64947); closed at review.
- Upstream labels: `http`.
- Source: [lib/_http_client.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_client.js#L275), [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L310).

## Problem and impact

HTTP request rewriting and connection creation can independently evaluate the same proxy configuration and destination. Large NO_PROXY lists make repeated matching work more noticeable.

## Evidence and reproduction

Both current JS call sites invoke checkShouldUseProxy. No benchmark or equivalence proof across Agent overrides was performed. Different effective options must still be evaluated independently; the source TODO is not proof that every pair of calls can be merged. [Built-in proxy support](https://nodejs.org/api/http.html#built-in-proxy-support).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| 2026-09-11 (snapshot) | [PR #65617](https://github.com/nodejs/node/pull/65617) | Nearby active work | Open at research | Changes NO_PROXY matching semantics, not decision reuse. |
| 2026-09-11 (snapshot) | [PR #64528](https://github.com/nodejs/node/pull/64528) | Nearby active work | Open at research | Changes proxy configuration selection; effective inputs must still match. |

[#64947](https://github.com/nodejs/node/issues/64947) closed because a TODO alone is not a bug report; the maintainer explicitly invited a concrete implementation proposal. [#62907](https://github.com/nodejs/node/issues/62907) shows why host matching semantics are sensitive. Active [#65617](https://github.com/nodejs/node/pull/65617) changes NO_PROXY matching and [#64528](https://github.com/nodejs/node/pull/64528) changes proxy configuration selection; neither is a decision-reuse fix. Do not overlap their behavior changes.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Reuse the decision within one request when effective inputs match | Removes repeated matching without a long-lived cache. | Agent overrides can change effective inputs between call sites. |
| B: Cache decisions across requests on the agent | Could reuse more matching work. | Requires keying, invalidation, and retention policy; unnecessary for this scope. |

## Recommendation and boundaries

Keep reuse request-local and conditional on equivalent inputs; benchmark before adopting.

Investigate carrying the computed decision through the same request/connection setup when the effective destination and configuration match. Limit reuse to that request, with no global cache, and preserve overrides, retries through existing paths, and raw-header routing behavior.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Prove matching effective proxy inputs across both call sites before measuring request-local decision reuse.

Search terms used with `repo:nodejs/node`: 64947; checkShouldUseProxy; proxy decision cache. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Built-in proxy support](https://nodejs.org/api/http.html#built-in-proxy-support)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

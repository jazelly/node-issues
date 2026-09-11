# Make existing header getters reflect raw-array request headers

- Area: `http`.
- Kind: `correctness`.
- Assessment: **Medium fix; design agreement needed**.
- Last researched: **2026-09-11**.
- Upstream: [#64405](https://github.com/nodejs/node/issues/64405); open at review.
- Upstream labels: `http`.
- Source: [lib/_http_client.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_client.js#L534), [lib/_http_outgoing.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L844).

## Problem and impact

Supplying headers as a raw array takes a path that does not populate the map consulted by existing request getters. The request can send a header while getHeader/hasHeader cannot see it.

## Evidence and reproduction

The raw-array path and getter-map mismatch were inspected in pinned source. This pass did not run a new network reproduction. [Request headers](https://nodejs.org/api/http.html#httprequestoptions-callback), [getHeader](https://nodejs.org/api/http.html#requestgetheadername).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| 2026-09-11 (snapshot) | No implementation PR identified in the recorded search | No identified attempt | Unknown beyond the search scope | Issue discussion below is evidence; absence of a PR is not approval or proof of no work. |

[#64405](https://github.com/nodejs/node/issues/64405) discusses this in July 2026. Eager normalization was challenged because it can change casing, duplicate handling, and order on the wire; a lazy getter view was proposed. No open implementation or current claimant was found. The compatibility objection remains unresolved, so this is not an approved one-line normalization fix.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Build a lazy read view for existing header getters | Preserves raw wire order, casing, and duplicates. | Duplicate-value semantics and synchronization with mutations remain to be specified. |
| B: Normalize raw-array headers eagerly into the existing map | Reuses normal getter implementation. | Historical discussion objects to changes in wire ordering, casing, and duplicate handling. |

## Recommendation and boundaries

Prefer discussion of a lazy view; eager normalization is not an accepted shortcut.

Provide a consistent read view for existing getters without rewriting the original raw sequence. Establish duplicate-value semantics first, and cover getHeaders/getHeaderNames together so they describe the same request.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Agree on duplicate values and getter-view synchronization while preserving the raw wire sequence.

Search terms used with `repo:nodejs/node`: 64405; raw array headers getHeader; header getters. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Request headers](https://nodejs.org/api/http.html#httprequestoptions-callback)
- [getHeader](https://nodejs.org/api/http.html#requestgetheadername)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

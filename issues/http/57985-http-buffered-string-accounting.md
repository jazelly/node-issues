# Resolve inconsistent HTTP pre-socket string accounting

- Area: `http`.
- Kind: `correctness`.
- Assessment: **Medium fix candidate; reproduced; compatibility gate**.
- Last researched: **2026-09-11**.
- Upstream: [#57985](https://github.com/nodejs/node/issues/57985); open at review.
- Upstream labels: none.
- Source: [lib/_http_outgoing.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L407), [lib/_http_outgoing.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L432).

## Problem and impact

Before a socket is assigned, outputSize counts string code units although the later socket buffers encoded bytes. The size/byteLength arguments already threaded through _send/_writeRaw are ignored. A request can therefore change its reported buffering units as it acquires a socket.

## Evidence and reproduction

A ClientRequest held before connection, after writing "é", reported writableLength 89 while its queued strings encoded to 90 bytes on the local v27 build. The unit mismatch is reproduced; classifying a byte-based replacement as compatible still requires discussion. [HTTP outgoing writableLength](https://nodejs.org/api/http.html#outgoingmessagewritablelength).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #46601](https://github.com/nodejs/node/pull/46601) | Performance precedent | Landed | Deliberately avoided unconditional byteLength computation. |
| Unknown in notebook | [PR #52828](https://github.com/nodejs/node/pull/52828) | Related rejected approach | Closed after compatibility discussion | General byte accounting would break readable.read(n) code-unit behavior. |
| Unknown in notebook | [PR #52842](https://github.com/nodejs/node/pull/52842) | Accepted behavior decision | Landed | Documents string-stream code-unit accounting; do not silently override it. |

[#57985](https://github.com/nodejs/node/issues/57985) remained relevant in July 2026. [#46601](https://github.com/nodejs/node/pull/46601) deliberately reduced unconditional byteLength work. Crucially, [#52828](https://github.com/nodejs/node/pull/52828) rejected a general stream-unit change because readable.read(n) uses code units; [#52842](https://github.com/nodejs/node/pull/52842) documented that behavior. This HTTP-specific proposal must justify its wire-buffer boundary without changing that general stream contract. No matching active fix was found.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Use known encoded lengths at the HTTP buffering boundary | Could keep HTTP pre-socket and socket accounting in the same units. | Header prepending, decrements, compatibility, and extra encoding scans need review. |
| B: Keep code-unit accounting and clarify the HTTP boundary | Preserves established string-stream behavior and avoids extra scans. | Retains the observed unit mismatch when a socket is attached. |

## Recommendation and boundaries

Do not implement the byte-based change until the HTTP-specific case overcomes the documented stream compatibility objection.

Investigate using known byte lengths for HTTP output accounting, including any header prepended by _send, with consistent increment/decrement logic. Avoid an unconditional extra encoding scan where length is already known. Do not treat removing unused parameters alone as the useful improvement.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Establish whether the HTTP-specific accounting change is compatible with the documented general string-stream contract.

Search terms used with `repo:nodejs/node`: 57985; buffered writableLength; decodeStrings byteLength. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [HTTP outgoing writableLength](https://nodejs.org/api/http.html#outgoingmessagewritablelength)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

# Make existing header getters reflect raw-array request headers

- Area: `http`.
- Assessment: **Medium fix; design agreement needed**.
- Checked: **2026-09-11**.
- Upstream: [#64405](https://github.com/nodejs/node/issues/64405); open at review.
- Upstream labels: `http`.
- Source: [lib/_http_client.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_client.js#L534), [lib/_http_outgoing.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L844).

## Observable problem or cost

Supplying headers as a raw array takes a path that does not populate the map consulted by existing request getters. The request can send a header while getHeader/hasHeader cannot see it.

## Narrow JavaScript change

Provide a consistent read view for existing getters without rewriting the original raw sequence. Establish duplicate-value semantics first, and cover getHeaders/getHeaderNames together so they describe the same request.

## History, objections, and ownership

[#64405](https://github.com/nodejs/node/issues/64405) discusses this in July 2026. Eager normalization was challenged because it can change casing, duplicate handling, and order on the wire; a lazy getter view was proposed. No open implementation or current claimant was found. The compatibility objection remains unresolved, so this is not an approved one-line normalization fix.

## Evidence and limits

The raw-array path and getter-map mismatch were inspected in pinned source. This pass did not run a new network reproduction. [Request headers](https://nodejs.org/api/http.html#httprequestoptions-callback), [getHeader](https://nodejs.org/api/http.html#requestgetheadername).

Search terms used with `repo:nodejs/node`: 64405; raw array headers getHeader; header getters. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

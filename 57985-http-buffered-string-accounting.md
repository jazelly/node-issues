# Resolve inconsistent HTTP pre-socket string accounting

- Area: `http`.
- Assessment: **Medium fix candidate; reproduced; compatibility gate**.
- Checked: **2026-09-11**.
- Upstream: [#57985](https://github.com/nodejs/node/issues/57985); open at review.
- Upstream labels: none.
- Source: [lib/_http_outgoing.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L407), [lib/_http_outgoing.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L432).

## Observable problem or cost

Before a socket is assigned, outputSize counts string code units although the later socket buffers encoded bytes. The size/byteLength arguments already threaded through _send/_writeRaw are ignored. A request can therefore change its reported buffering units as it acquires a socket.

## Narrow JavaScript change

Investigate using known byte lengths for HTTP output accounting, including any header prepended by _send, with consistent increment/decrement logic. Avoid an unconditional extra encoding scan where length is already known. Do not treat removing unused parameters alone as the useful improvement.

## History, objections, and ownership

[#57985](https://github.com/nodejs/node/issues/57985) remained relevant in July 2026. [#46601](https://github.com/nodejs/node/pull/46601) deliberately reduced unconditional byteLength work. Crucially, [#52828](https://github.com/nodejs/node/pull/52828) rejected a general stream-unit change because readable.read(n) uses code units; [#52842](https://github.com/nodejs/node/pull/52842) documented that behavior. This HTTP-specific proposal must justify its wire-buffer boundary without changing that general stream contract. No matching active fix was found.

## Evidence and limits

A ClientRequest held before connection, after writing "é", reported writableLength 89 while its queued strings encoded to 90 bytes on the local v27 build. The unit mismatch is reproduced; classifying a byte-based replacement as compatible still requires discussion. [HTTP outgoing writableLength](https://nodejs.org/api/http.html#outgoingmessagewritablelength).

Search terms used with `repo:nodejs/node`: 57985; buffered writableLength; decodeStrings byteLength. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

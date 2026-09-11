# Resolve inconsistent HTTP pre-socket string accounting

- Issue: [#57985](https://github.com/nodejs/node/issues/57985) — open when checked
- Checked: 2026-09-11
- Status: No matching active fix found; compatibility decision needed (at that check)

## What's wrong?

Before an HTTP request gets a socket, its buffer counts JavaScript string units. The socket later counts encoded bytes. The same buffered data can therefore be reported in different units; the size arguments passed into the HTTP helpers are currently ignored.

## Where to start

- Code: [lib/_http_outgoing.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L407), [lib/_http_outgoing.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L432)
- Docs: [HTTP outgoing writableLength](https://nodejs.org/api/http.html#outgoingmessagewritablelength)
- Try it / evidence: A ClientRequest held before connection, after writing "é", reported writableLength 89 while its queued strings encoded to 90 bytes on the local v27 build. The unit mismatch is reproduced; classifying a byte-based replacement as compatible still requires discussion. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#57985](https://github.com/nodejs/node/issues/57985) was reconfirmed in July 2026. [#46601](https://github.com/nodejs/node/pull/46601) deliberately avoided unnecessary byte-length calculations.
- [#52828](https://github.com/nodejs/node/pull/52828) rejected changing general stream accounting because read(n) operates on string units; [#52842](https://github.com/nodejs/node/pull/52842) documented that behavior. An HTTP-specific change must address this objection.

Search: `repo:nodejs/node` — 57985; buffered writableLength; decodeStrings byteLength. Recheck activity before starting.

## Suggested fix

- Idea: Investigate using already-known encoded lengths for HTTP buffering, including any prepended headers.
- Benefit: Could keep HTTP buffer accounting consistent before and after socket assignment.
- Risk: A byte-based change may break established behavior. Preserve general stream accounting and matching increments/decrements; keeping the current units is valid if compatibility rules out a change.

## Next step

Establish whether HTTP can change its accounting without violating the documented string-stream contract.

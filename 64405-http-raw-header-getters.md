# #64405 — Request getters omit headers supplied as an array

- Upstream: [req.getHeaders() does not return values set using rawHeaders array](https://github.com/nodejs/node/issues/64405)
- Checked: 2026-09-11; open, no assignee.
- Upstream labels: `http`.
- Area: `http`.
- Assessment: small JS entry point, but behavior agreement is required before a fix.

## Problem and entry point

Passing `headers: ['foo', 'bar', 'baz', 'qux']` to an HTTP request sends those headers, while `getHeaders()`, `getHeaderNames()`, and `getRawHeaderNames()` do not expose them.

The [client constructor](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_client.js#L534) handles the array separately from object headers and passes it directly to `_storeHeader()`. The [getters](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L844) read `kOutHeaders`. This is a JavaScript representation mismatch, with no native-code change proposed.

## Smallest useful follow-up

Create a local HTTP server test showing both the received raw headers and the request getters. Compare object and array input. Then settle whether getters should expose a lazily derived map or whether the existing distinction should be documented.

If a getter fix is accepted, retain duplicate raw headers, original casing, and on-wire order. Do not eagerly normalize the outgoing array merely to make a getter test pass. The discussion explicitly raises that concern.

Suggested new regression location: `test/parallel/test-http-client-raw-header-getters.js` (proposed file, not an existing test).

## Activity evidence

All four issue comments are from 2026-07-10. One contributor investigated the internal representation; another suggested lazy map construction. No assignee, linked implementation PR, recent ownership claim, or matching open fix was found. An issue-number search also returned unrelated source-map PR #52153, which does not implement this behavior.

This is a dormant design follow-up, not an agreed beginner fix. No reproduction or patch was performed for this notebook.

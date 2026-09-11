# #54111 — Socket connect test depends on localhost resolution

- Upstream: [test-net-socket-connect-without-cb test fails](https://github.com/nodejs/node/issues/54111)
- Checked: 2026-09-11; open, no assignee.
- Upstream labels: `net`, `test`, `stale`.
- Area: `net`, `test`.
- Assessment: best first candidate; a small JavaScript test correction.

## Problem and entry point

The test is meant to prove that `socket.connect()` accepts an omitted callback. It passes the object returned by `server.address()` directly to `client.connect()`. That object has `address`, `family`, and `port`, whereas connection options use `host`. Consequently, the test can resolve the default `localhost` instead of connecting explicitly to its server.

The reported failure requires IPv6 support with no IPv6 localhost entry in the hosts configuration. A contributor reconfirmed the condition on 2026-05-16.

The [current test](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/test/parallel/test-net-socket-connect-without-cb.js#L19) still contains the relevant `client.connect(address)` call.

## Smallest useful change

Reproduce the reported configuration in an isolated environment, then supply an explicit loopback host appropriate for the listening family while preserving the omitted-callback assertion. Keep the existing container-related branch in view. Do not turn this into a DNS resolver change or a sweep of every affected test.

Suggested focused command from a built Node.js checkout:

```sh
python3 tools/test.py test/parallel/test-net-socket-connect-without-cb.js
```

A pass on an ordinary developer machine alone does not validate the reported failure condition. No reproduction was run for this notebook.

## Activity evidence

The only linked Node.js PR, [#63368](https://github.com/nodejs/node/pull/63368), proposed returning localhost IPs from DNS and is closed. The latest substantive issue comment is the 2026-05-16 confirmation; the 2026-08-16 comment is a stale notice. No matching open implementation PR or recent ownership claim was found. This is a narrower test-focused follow-up, not a revival of that DNS proposal.

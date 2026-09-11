# #48767 — Auto-select-family test fails its address assertion

- Upstream: [test-net-autoselectfamily.js failed](https://github.com/nodejs/node/issues/48767)
- Checked: 2026-09-11; open, no assignee.
- Upstream labels: `test`.
- Area: `net`, `test`.
- Assessment: one JS test surface; reproduction and timing analysis are the uncertainty.

## Problem and entry point

The test expects a full ordered list of attempted IPv6/IPv4 addresses before a local connection succeeds. Reports show fewer attempted addresses than expected. The original report was on macOS; a 2026-04-30 comment reports the issue on Windows with `v27.0.0-pre` and says reducing the attempt timeout changes the result.

The [current test](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/test/parallel/test-net-autoselectfamily.js#L66) still combines public addresses with loopback addresses and asserts the full attempted sequence. Its timeout comes from `common.defaultAutoSelectFamilyAttemptTimeout`.

## Smallest useful change

Run the existing test with a matching built runtime and capture the attempted-address list when it fails. Establish which network/timing assumption is wrong, then adjust the fixture or synchronization so the test exercises the intended fallback sequence. Retain assertions that fallback reaches the local server and returns the expected data.

```sh
python3 tools/test.py test/parallel/test-net-autoselectfamily.js
```

Simply reducing a timeout or weakening the expected list is not yet a justified fix. Start with the test; if reproduction instead proves a native networking defect, stop treating this as a small JS-only candidate. No reproduction was run for this notebook.

## Activity evidence

No PR appeared in the issue's cross-reference/connection timeline, and number/title searches found no matching open fix. The last substantive report is dated 2026-04-30; the 2026-08-28 entry is a stale notice. Related open networking PRs #65239 and #64430 concern reconnect lookup callbacks and TLS session reuse respectively, rather than this test assertion.

# Close file descriptor zero in file streams

- Issue: [#57905](https://github.com/nodejs/node/issues/57905) — open when checked
- Checked: 2026-09-11
- Status: No active fix found in the recorded search; recheck before starting

## What's wrong?

A file descriptor is the number used to identify an open file. The close helper
checks `!stream.fd`, which treats the valid number `0` as missing. A stream can
therefore finish without closing its file.

## Where to start

- Code: [close() in internal/fs/streams.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/fs/streams.js#L124)
- Docs: [fs.createReadStream options](https://nodejs.org/api/fs.html#fscreatereadstreampath-options)
- Try it: Create a ReadStream with `fd: 0`, `autoClose: true`, and custom `fs`
  methods that return end-of-file and count close calls. Let the stream finish.
  The earlier run on macOS arm64 with v24.15.0 and a local v27.0.0-pre build
  observed **zero close calls**, although the stream reported close and destroyed.
  The custom methods avoid touching standard input. The v27 binary was not built
  from the linked source commit; see [research limits](../../RESEARCH.md).

## What's been tried?

- 2025 — [PR #57914](https://github.com/nodejs/node/pull/57914) and
  [PR #57918](https://github.com/nodejs/node/pull/57918): both attempted a fix and
  closed without merging. The first review requested evidence covering fd 0;
  no design rejection was found in the recorded review.
- July 2026 — the issue was reconfirmed from source. The September search checked
  the issue comments, linked PRs, and open PRs using `57905`, `file descriptor 0`,
  and `fs streams close`; no matching active fix or recent claim was found.

## Suggested fix

- Idea: Check for an absent descriptor explicitly instead of using `!stream.fd`.
- Benefit: Valid fd 0 goes through the existing close operation.
- Risk: Preserve how missing descriptors are handled and ensure each close
  callback runs once. This helper serves both read and write streams.

## Next step

Confirm which values represent a missing descriptor in this helper before
preparing the small JS change.

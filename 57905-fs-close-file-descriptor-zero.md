# Close file descriptor zero in file streams

- Area: `fs`.
- Assessment: **Small fix; reproduced**.
- Checked: **2026-09-11**.
- Upstream: [#57905](https://github.com/nodejs/node/issues/57905); open at review.
- Upstream labels: none.
- Source: [lib/internal/fs/streams.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/fs/streams.js#L124).

## Observable problem or cost

The shared close helper treats a falsy descriptor as absent. Descriptor `0` is valid, so an automatically destroyed file stream can report completion without calling its close operation.

## Narrow JavaScript change

Replace the truthiness check with the actual absent-descriptor check. Preserve the existing close callback, flush ordering, and error propagation for both ReadStream and WriteStream.

## History, objections, and ownership

The [issue](https://github.com/nodejs/node/issues/57905) was reconfirmed from source in July 2026. [#57914](https://github.com/nodejs/node/pull/57914) and [#57918](https://github.com/nodejs/node/pull/57918) are closed, unmerged attempts from 2025. The former review asked for evidence covering fd 0; no design rejection was found. No replacement open fix or recent ownership claim was found.

## Evidence and limits

Reproduced on v24.15.0 and the local v27.0.0-pre build using the documented custom `fs` option: an EOF read with fd 0 emitted close with `destroyed: true`, but the close callback implementation was invoked zero times. This avoided touching the process standard input. [File-stream contract](https://nodejs.org/api/fs.html#fscreatereadstreampath-options).

Search terms used with `repo:nodejs/node`: 57905; file descriptor 0; fs streams close. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

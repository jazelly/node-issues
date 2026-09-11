# Preserve Unicode Symbol descriptions during error deserialization

- Area: `worker`.
- Assessment: **Small fix; reproduced**.
- Checked: **2026-09-11**.
- Upstream issue: none identified; related historical PRs are references, not a claimed issue number.
- Source: [lib/internal/error_serdes.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/error_serdes.js#L192).

## Observable problem or cost

The Symbol branch decodes a UTF-8 buffer, then uses the buffer byte length as the end index of a JavaScript substring. Non-ASCII descriptions therefore retain the closing parenthesis and produce a different registry Symbol.

## Narrow JavaScript change

Decode once and calculate the substring boundary from the decoded string length. Keep the existing Symbol.for transport semantics and serialization tag unchanged.

## History, objections, and ownership

[#47925](https://github.com/nodejs/node/pull/47925) introduced this branch while extending error transport. Its review discussed allocation cost and difficult error descriptors, but the reviewed discussion did not require byte-based indexing or identify a Unicode reason for it. Searches for `kInspectedSymbol` and Symbol error serialization found no open correction. This is a source-discovered candidate, not an upstream issue number.

## Evidence and limits

On both local runtimes, round-tripping Symbol.for("ascii") preserved its description; "é", "中文", and "😀" became "é)", "中文)", and "😀)". The probe used internal/error_serdes with --expose-internals. [Worker error-event contract](https://nodejs.org/api/worker_threads.html#event-error).

Search terms used with `repo:nodejs/node`: kInspectedSymbol; serializeError symbol; error_serdes. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

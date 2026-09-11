# Separate input streams from history configuration before initialization

- Area: `readline`.
- Assessment: **Small to medium fix; reproduced**.
- Checked: **2026-09-11**.
- Upstream: [#61526](https://github.com/nodejs/node/issues/61526); closed at review.
- Upstream labels: `readline`, `stale`.
- Source: [lib/internal/readline/interface.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/readline/interface.js#L226), [lib/internal/readline/interface.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/readline/interface.js#L416).

## Observable problem or cost

The positional input-stream overload can pass the stream itself as history options. If it exposes onHistoryFileLoaded, history initialization calls pause before this.input has been assigned.

## Narrow JavaScript change

Resolve the overload first, use a deliberate history-options object, and initialize only after the required input state exists. Preserve historySize, supplied history, and duplicate-removal behavior for the options overload.

## History, objections, and ownership

[#61526](https://github.com/nodejs/node/issues/61526) traces the regression to [#58225](https://github.com/nodejs/node/pull/58225). The claimant explicitly left it for others in January 2026. [#61538](https://github.com/nodejs/node/pull/61538) later closed stale. Its review correctly warns that merely moving an assignment still accidentally interprets stream properties as REPL configuration. That unresolved objection is part of this scope.

## Evidence and limits

Readable.from([]) with an onHistoryFileLoaded property, passed positionally to createInterface, throws a TypeError reading pause on both local runtimes. No persistent history files were created. [Readline creation contract](https://nodejs.org/api/readline.html#readlinecreateinterfaceoptions).

Search terms used with `repo:nodejs/node`: 61526; readline history init; onHistoryFileLoaded. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

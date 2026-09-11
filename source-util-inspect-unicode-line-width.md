# Use display width consistently for finite inspect line breaking

- Area: `util`.
- Assessment: **Medium improvement; observed; performance gate**.
- Checked: **2026-09-11**.
- Upstream issue: none identified; related historical PRs are references, not a claimed issue number.
- Source: [lib/internal/util/inspect.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/util/inspect.js#L2613), [lib/internal/util/inspect.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/util/inspect.js#L2946).

## Observable problem or cost

isBelowBreakLength adds UTF-16 lengths, whereas other inspect formatting paths use display width. Combining marks and wide characters consequently receive inconsistent compact line breaks.

## Narrow JavaScript change

Use the existing width calculation for finite line-breaking decisions, preserving an efficient ASCII path and the Infinity shortcut. Keep the current API; if acceptable overhead requires a new opt-in flag, stop this candidate rather than turn it into a feature proposal.

## History, objections, and ownership

The source TODO explicitly postpones Unicode handling pending performance evidence and suggests an option only if overhead is significant. [#26269](https://github.com/nodejs/node/pull/26269) establishes compact-layout behavior; [#64238](https://github.com/nodejs/node/pull/64238) recently added the Infinity path. No exact open Unicode-width fix was found. This is a known tradeoff to measure, not a forgotten typo.

## Evidence and limits

On local v27 with compact:true and breakLength:25, {a:"aaaaa",b:"aaaaa"} stays on one line, the same visible letters with combining accents split, while five wide CJK characters per value stay on one line. These are observed formatting differences; acceptable width policy and throughput remain to be agreed. [inspect options](https://nodejs.org/api/util.html#utilinspectobject-options).

Search terms used with `repo:nodejs/node`: isBelowBreakLength; getStringWidth breakLength; inspect unicode width. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

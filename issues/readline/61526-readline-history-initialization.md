# Separate input streams from history configuration before initialization

- Area: `readline`.
- Kind: `correctness`.
- Assessment: **Small to medium fix; reproduced**.
- Last researched: **2026-09-11**.
- Upstream: [#61526](https://github.com/nodejs/node/issues/61526); closed at review.
- Upstream labels: `readline`, `stale`.
- Source: [lib/internal/readline/interface.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/readline/interface.js#L226), [lib/internal/readline/interface.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/readline/interface.js#L416).

## Problem and impact

The positional input-stream overload can pass the stream itself as history options. If it exposes onHistoryFileLoaded, history initialization calls pause before this.input has been assigned.

## Evidence and reproduction

Readable.from([]) with an onHistoryFileLoaded property, passed positionally to createInterface, throws a TypeError reading pause on both local runtimes. No persistent history files were created. [Readline creation contract](https://nodejs.org/api/readline.html#readlinecreateinterfaceoptions).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #58225](https://github.com/nodejs/node/pull/58225) | Introducing change | Landed | The issue traces the history initialization regression to this change. |
| Unknown in notebook | [PR #61538](https://github.com/nodejs/node/pull/61538) | Previous fix attempt | Closed for inactivity | Reordering alone does not separate input-stream properties from history options. |

[#61526](https://github.com/nodejs/node/issues/61526) traces the regression to [#58225](https://github.com/nodejs/node/pull/58225). The claimant explicitly left it for others in January 2026. [#61538](https://github.com/nodejs/node/pull/61538) later closed stale. Its review correctly warns that merely moving an assignment still accidentally interprets stream properties as REPL configuration. That unresolved objection is part of this scope.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Normalize history options and initialize after input state exists | Addresses both the crash and accidental interpretation of stream properties. | Must preserve options-overload history settings. |
| B: Only move the input assignment earlier | Minimal diff and directly avoids the undefined input access. | Leaves accidental REPL history initialization; prior review identified this gap. |

## Recommendation and boundaries

Prefer overload normalization plus correct ordering; the reorder-only approach is incomplete.

Resolve the overload first, use a deliberate history-options object, and initialize only after the required input state exists. Preserve historySize, supplied history, and duplicate-removal behavior for the options overload.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Trace both createInterface overloads and identify the minimal initialization order that does not trigger accidental history setup.

Search terms used with `repo:nodejs/node`: 61526; readline history init; onHistoryFileLoaded. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Readline creation contract](https://nodejs.org/api/readline.html#readlinecreateinterfaceoptions)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

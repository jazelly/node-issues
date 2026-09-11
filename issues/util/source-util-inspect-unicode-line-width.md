# Use display width consistently for finite inspect line breaking

- Area: `util`.
- Kind: `correctness`.
- Assessment: **Medium improvement; observed; performance gate**.
- Last researched: **2026-09-11**.
- Upstream: None identified; related historical PRs are references, not a claimed issue number.
- Upstream labels: Not applicable.
- Source: [lib/internal/util/inspect.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/util/inspect.js#L2613), [lib/internal/util/inspect.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/util/inspect.js#L2946).

## Problem and impact

isBelowBreakLength adds UTF-16 lengths, whereas other inspect formatting paths use display width. Combining marks and wide characters consequently receive inconsistent compact line breaks.

## Evidence and reproduction

On local v27 with compact:true and breakLength:25, {a:"aaaaa",b:"aaaaa"} stays on one line, the same visible letters with combining accents split, while five wide CJK characters per value stay on one line. These are observed formatting differences; acceptable width policy and throughput remain to be agreed. [inspect options](https://nodejs.org/api/util.html#utilinspectobject-options).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| Unknown in notebook | [PR #26269](https://github.com/nodejs/node/pull/26269) | Formatting precedent | Landed | Established compact layout; performance-sensitive implementation. |
| Unknown in notebook | [PR #64238](https://github.com/nodejs/node/pull/64238) | Related formatting fix | Landed | Preserve the existing Infinity shortcut. |

The source TODO explicitly postpones Unicode handling pending performance evidence and suggests an option only if overhead is significant. [#26269](https://github.com/nodejs/node/pull/26269) establishes compact-layout behavior; [#64238](https://github.com/nodejs/node/pull/64238) recently added the Infinity path. No exact open Unicode-width fix was found. This is a known tradeoff to measure, not a forgotten typo.

Dates marked Unknown were not recorded precisely in the earlier notebook. They have not been inferred from a PR's last-updated timestamp.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Use display width with an efficient ASCII path | Aligns finite line breaking with other Unicode-aware formatting paths. | Potential throughput cost and output changes require evidence. |
| B: Keep code-unit line breaking | Preserves current formatting and avoids extra width work. | Leaves inconsistent treatment of combining and wide characters. |

## Recommendation and boundaries

Measure finite-width handling first; defer if it needs a new public option to be acceptable.

Use the existing width calculation for finite line-breaking decisions, preserving an efficient ASCII path and the Infinity shortcut. Keep the current API; if acceptable overhead requires a new opt-in flag, stop this candidate rather than turn it into a feature proposal.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Measure finite-width formatting with the ASCII path and determine whether behavior and overhead are acceptable without a new option.

Search terms used with `repo:nodejs/node`: isBelowBreakLength; getStringWidth breakLength; inspect unicode width. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [inspect options](https://nodejs.org/api/util.html#utilinspectobject-options)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

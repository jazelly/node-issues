# Avoid duplicate inspector events from internal console delegation

- Area: `console`.
- Kind: `correctness`.
- Assessment: **Medium fix; reproduced**.
- Last researched: **2026-09-11**.
- Upstream: [#31973](https://github.com/nodejs/node/issues/31973); open at review.
- Upstream labels: `console`, `inspector`.
- Source: [lib/internal/console/constructor.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/console/constructor.js#L530).

## Problem and impact

A single console.group call generates both a group event and an extra log event for inspector clients. The JavaScript implementation calls the public, inspector-wrapped log method to print its label.

## Evidence and reproduction

A local inspector.Session subscribed to Runtime.consoleAPICalled observed [startGroup, log, endGroup] for one group/groupEnd pair on the v27 build. The terminal label printed once. [Console group semantics](https://nodejs.org/api/console.html#consolegrouplabel).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| 2026-09-11 (snapshot) | No implementation PR identified in the recorded search | No identified attempt | Unknown beyond the search scope | Issue discussion below is evidence; absence of a PR is not approval or proof of no work. |

[#31973](https://github.com/nodejs/node/issues/31973) has remained unresolved, with renewed activity in June 2026 and no linked implementation PR. Historical discussion identifies the extra protocol notification. Avoid an unreviewed blanket replacement of public-method calls: monkey-patched Console instances and inspector interception are compatibility considerations.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Use a private output path for internal console delegation | Targets duplicate notifications at the JS call site. | Can affect users who override public Console methods. |
| B: Suppress nested notifications in the inspector integration | Keeps existing public-method delegation intact. | Broader instrumentation change; may suppress legitimate nested logging. |

## Recommendation and boundaries

Investigate the private path first and establish overridden-method semantics before changing behavior.

Route internal label output through an appropriate private output path while keeping public console calls observable. Start with group and groupCollapsed; audit related internal delegation without changing indentation or output formatting.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Check the compatibility of overridden Console methods before selecting the private output path.

Search terms used with `repo:nodejs/node`: 31973; console group duplicate inspector; console delegation. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Console group semantics](https://nodejs.org/api/console.html#consolegrouplabel)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

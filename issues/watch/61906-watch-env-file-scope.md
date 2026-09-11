# Keep env-file watching from widening explicit watch paths

- Area: `watch`.
- Kind: `correctness`.
- Assessment: **Small to medium fix; dormant follow-up**.
- Last researched: **2026-09-11**.
- Upstream: [#61906](https://github.com/nodejs/node/issues/61906); open at review.
- Upstream labels: none.
- Source: [lib/internal/watch_mode/files_watcher.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/watch_mode/files_watcher.js#L92), [lib/internal/watch_mode/files_watcher.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/watch_mode/files_watcher.js#L134).

## Problem and impact

On systems with recursive watching, filterFile watches the parent directory. In all mode, selected by --watch-path, the change handler does not apply the file filter. Adding an env file therefore makes unrelated sibling changes restart the process.

## Evidence and reproduction

Source confirms the filter/all mismatch. A fresh end-to-end watcher run was not performed. The proposal is limited to the existing env-file/watch-path interaction, not the separate Linux shutdown issue #64819. [Watch-path documentation](https://nodejs.org/api/cli.html#--watch-path).

## Previous discussion and attempts

| Date / precision | Reference | Relationship | Outcome at research | Lesson or constraint |
| --- | --- | --- | --- | --- |
| 2026-08-28 (closure) | [PR #61970](https://github.com/nodejs/node/pull/61970) | Previous fix attempt | Closed for inactivity | Direct-file watching proposed; preserve optional-file and replacement behavior. |

[#61906](https://github.com/nodejs/node/issues/61906) provides the CLI reproduction. [#61970](https://github.com/nodejs/node/pull/61970) proposed the direct-file approach and was closed for inactivity on 2026-08-28. No design rejection appeared in its comments or inline review. The source explicitly explains why parent-directory recursive watching is normally cheaper, so keep that optimization in ordinary filter mode. No matching open fix was found.

## Solution approaches

The comparisons below are notebook assessments. Historical support or objections are attributed in the previous section; an alternative is not necessarily recommended or in scope.

| Approach | Pros | Cons / compatibility risks |
| --- | --- | --- |
| A: Use a direct file subscription for the env file in all mode | Avoids broadening the explicit watch scope with a small change. | File replacement and optional missing files need to retain their behavior. |
| B: Keep the recursive parent watch and add subscription-specific filtering | Retains watcher sharing. | Requires more nuanced filtering without suppressing explicit directory subscriptions. |

## Recommendation and boundaries

Evaluate the direct-file approach first; retain the ordinary filter-mode optimization.

Represent the env-file subscription without broadening the user-selected watch scope. A direct non-recursive file watcher is the small approach from the old PR; preserve the existing behavior for missing optional env files and file replacement.

## Availability and next step

- Availability snapshot: **2026-09-11**. The history section records previous attempts and nearby active work; public activity must be rechecked before claiming or implementing this candidate.
- Notebook contributor / claim: **Unclaimed in this notebook**. This says nothing about upstream ownership.
- Next step: Run the explicit watch-path/env-file scenario and compare file replacement behavior under the direct-file subscription.

Search terms used with `repo:nodejs/node`: 61906; 61970; watch env-file unrelated restart. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](../../RESEARCH.md) for runtime and source-version limits.

## References

- [Watch-path documentation](https://nodejs.org/api/cli.html#--watch-path)
- [Pinned source revision](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).
- [Research scope and runtime limits](../../RESEARCH.md).

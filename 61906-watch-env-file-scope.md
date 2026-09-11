# Keep env-file watching from widening explicit watch paths

- Area: `watch`.
- Assessment: **Small to medium fix; dormant follow-up**.
- Checked: **2026-09-11**.
- Upstream: [#61906](https://github.com/nodejs/node/issues/61906); open at review.
- Upstream labels: none.
- Source: [lib/internal/watch_mode/files_watcher.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/watch_mode/files_watcher.js#L92), [lib/internal/watch_mode/files_watcher.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/watch_mode/files_watcher.js#L134).

## Observable problem or cost

On systems with recursive watching, filterFile watches the parent directory. In all mode, selected by --watch-path, the change handler does not apply the file filter. Adding an env file therefore makes unrelated sibling changes restart the process.

## Narrow JavaScript change

Represent the env-file subscription without broadening the user-selected watch scope. A direct non-recursive file watcher is the small approach from the old PR; preserve the existing behavior for missing optional env files and file replacement.

## History, objections, and ownership

[#61906](https://github.com/nodejs/node/issues/61906) provides the CLI reproduction. [#61970](https://github.com/nodejs/node/pull/61970) proposed the direct-file approach and was closed for inactivity on 2026-08-28. No design rejection appeared in its comments or inline review. The source explicitly explains why parent-directory recursive watching is normally cheaper, so keep that optimization in ordinary filter mode. No matching open fix was found.

## Evidence and limits

Source confirms the filter/all mismatch. A fresh end-to-end watcher run was not performed. The proposal is limited to the existing env-file/watch-path interaction, not the separate Linux shutdown issue #64819. [Watch-path documentation](https://nodejs.org/api/cli.html#--watch-path).

Search terms used with `repo:nodejs/node`: 61906; 61970; watch env-file unrelated restart. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

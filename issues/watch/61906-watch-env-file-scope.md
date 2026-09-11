# Keep env-file watching from widening explicit watch paths

- Issue: [#61906](https://github.com/nodejs/node/issues/61906) — open when checked
- Checked: 2026-09-11
- Status: No matching active fix found; command-line reproduction still needed (at that check)

## What's wrong?

With --watch-path and an env file, changing an unrelated file beside the env file can restart Node. The code watches the env file’s parent directory, but this mode skips the filter that normally ignores sibling files.

## Where to start

- Code: [lib/internal/watch_mode/files_watcher.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/watch_mode/files_watcher.js#L92), [lib/internal/watch_mode/files_watcher.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/watch_mode/files_watcher.js#L134)
- Docs: [Watch-path documentation](https://nodejs.org/api/cli.html#--watch-path)
- Try it / evidence: Source confirms the filter/all mismatch. A fresh end-to-end watcher run was not performed. The proposal is limited to the existing env-file/watch-path interaction, not the separate Linux shutdown issue #64819. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#61906](https://github.com/nodejs/node/issues/61906) gives the command-line example. [#61970](https://github.com/nodejs/node/pull/61970) proposed watching the file directly and closed for inactivity on 2026-08-28.
- No design rejection was recorded. The existing parent-directory watch is normally a performance optimization, so keep it where filtering still works.

Search: `repo:nodejs/node` — 61906; 61970; watch env-file unrelated restart. Recheck activity before starting.

## Suggested fix

- Idea: Watch the env file directly in this mode without widening the explicit watch paths.
- Benefit: Avoids restarts caused by unrelated sibling files.
- Risk: Preserve replacement and optional-missing-file behavior. More detailed parent-watch filtering is another approach, but must not suppress explicit directory subscriptions; Linux shutdown issue #64819 is separate.

## Next step

Run the watch-path/env-file example and check what happens when the env file is replaced.

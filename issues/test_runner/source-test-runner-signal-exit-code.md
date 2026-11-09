# Preserve signal exit codes when the test runner is interrupted

- Issue: None found
- Checked: 2026-11-09
- Status: No active fix found; [#62359](https://github.com/nodejs/node/pull/62359) had three approvals and was closed as stale

## What's wrong?

When `node --test` receives SIGINT or SIGTERM, its termination handler reports
interrupted tests and then exits with code 1. Shells and CI tools expect 130 or
143 (or death by signal) so they can tell an interrupt apart from test failures.

## Where to start

- Code: [lib/internal/test_runner/harness.js `terminationHandler`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/test_runner/harness.js#L286-L295)
- Docs: [Test runner](https://nodejs.org/api/test.html),
  [Exit codes](https://nodejs.org/api/process.html#exit-codes)
- Try it: On `v28.0.0-pre` (`128f247ea4cb`, macOS arm64), spawn
  `node --test --test-isolation=none slow.test.js` with a 10-second test and send
  SIGINT or SIGTERM after one second. Both exit with code 1, signal `null`. The
  same file run without `--test` exits with 130 / 143.

## What's been tried?

- 2026-03 — [#62359](https://github.com/nodejs/node/pull/62359): set 130 / 143 in
  the handler and extended `test-runner-exit-code`. Approved by
  JakobJingleheimer, MoLow, and avivkeller. A maintainer noted the failing CI tests
  were related to the change; the author did not follow up and the stale bot
  closed it.

Search: `repo:nodejs/node` test_runner SIGINT exit code; terminationHandler;
130 143 test runner. No open PR found.

## Suggested fix

- Idea: Revive #62359. Record which signal triggered `terminationHandler` and exit
  with `128 + signal number`, or re-raise the signal after cleanup.
- Benefit: Interrupts become distinguishable in scripts and CI.
- Risk: Process-isolation mode has a parent and child processes; each must
  report correctly. The CI failures on #62359 need to be understood first.

## Next step

Rebase #62359, run `test/parallel/test-runner-exit-code.js` locally, and fix the
failures the reviewer pointed at.

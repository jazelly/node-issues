# Make `disable()` match its documentation under AsyncContextFrame

- Issue: None found
- Checked: 2026-11-09
- Status: No active fix found; a related docs PR [#58065](https://github.com/nodejs/node/pull/58065) was closed as stale

## What's wrong?

The docs say `asyncLocalStorage.disable()` exits all contexts linked to the
instance and is required before the instance can be garbage collected. With
`AsyncContextFrame`, `disable()` only deletes the key from the current frame
object. Callbacks sharing that frame lose the value, but frames copied earlier
(for example by a nested `run()` on another instance) keep it, so the documented
guarantee no longer holds.

## Where to start

- Code: [lib/internal/async_context_frame.js `disable`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/async_context_frame.js#L32-L39),
  [lib/internal/async_local_storage/async_context_frame.js `disable`](https://github.com/nodejs/node/blob/128f247ea4cb52da405f6cc35dec46c088be8af3/lib/internal/async_local_storage/async_context_frame.js#L50-L52)
- Docs: [`asyncLocalStorage.disable()`](https://nodejs.org/api/async_context.html#asynclocalstoragedisable)
- Try it: On `v28.0.0-pre` (`128f247ea4cb`, macOS arm64):

  ```js
  const { AsyncLocalStorage } = require('node:async_hooks');
  const a = new AsyncLocalStorage();
  const b = new AsyncLocalStorage();
  a.run(1, () => {
    b.run(2, () => setTimeout(() => console.log('child frame', a.getStore()), 10));
    setTimeout(() => console.log('same frame', a.getStore()), 10);
    a.disable();
  });
  // same frame undefined
  // child frame 1
  ```

## What's been tried?

- 2025-04 — [#58065](https://github.com/nodejs/node/pull/58065): proposed
  discouraging `disable()` in the docs. Reviewers agreed it is not the same as
  `exit()`, and one suggested deprecating it outright. Closed by the stale bot in
  August 2026 without a decision.
- 2026-10 — [#63641](https://github.com/nodejs/node/pull/63641): removed the legacy
  implementation, so the old `disable()` semantics no longer exist anywhere.

Search: `repo:nodejs/node` asyncLocalStorage.disable; disable AsyncContextFrame;
DEP disable. Closed PRs included.

## Suggested fix

- Idea: Update the `disable()` docs to describe the current behavior, drop the
  GC requirement, and recommend `run()` / `exit()` / `withScope()` instead.
- Benefit: Removes a guarantee the implementation cannot keep.
- Risk: Low; docs only. Wording must stay accurate for v24 and later, where
  `AsyncContextFrame` is the default.
- Alternative (optional): Documentation-only deprecation of `disable()`, as
  suggested in #58065. Needs maintainer support.

## Next step

Rebase the wording from #58065 onto current docs, add the example above, and
ask the original reviewers whether to go further with a deprecation.

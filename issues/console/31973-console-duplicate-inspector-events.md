# Avoid duplicate inspector events from internal console delegation

- Issue: [#31973](https://github.com/nodejs/node/issues/31973) — open when checked
- Checked: 2026-09-11
- Status: No matching active fix found; compatibility review needed (at that check)

## What's wrong?

One console.group() call sends two inspector notifications: a group event and an extra log event. Its JavaScript implementation calls the public log method, which is also observed by the inspector.

## Where to start

- Code: [lib/internal/console/constructor.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/console/constructor.js#L530)
- Docs: [Console group semantics](https://nodejs.org/api/console.html#consolegrouplabel)
- Try it / evidence: A local inspector.Session subscribed to Runtime.consoleAPICalled observed [startGroup, log, endGroup] for one group/groupEnd pair on the v27 build. The terminal label printed once. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#31973](https://github.com/nodejs/node/issues/31973) describes the duplicate notification and was raised again in June 2026. No implementation PR was identified in the recorded search.

Search: `repo:nodejs/node` — 31973; console group duplicate inspector; console delegation. Recheck activity before starting.

## Suggested fix

- Idea: Print the group label through an internal output helper rather than another public console call.
- Benefit: Could produce one inspector notification while preserving the terminal output.
- Risk: Some users override Console methods. Preserve their expected behavior, indentation, and formatting; suppressing nested inspector events more broadly could hide legitimate logs.

## Next step

Check how overridden Console methods should behave before selecting the internal output path.

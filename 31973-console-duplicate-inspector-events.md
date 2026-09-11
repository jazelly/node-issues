# Avoid duplicate inspector events from internal console delegation

- Area: `console`.
- Assessment: **Medium fix; reproduced**.
- Checked: **2026-09-11**.
- Upstream: [#31973](https://github.com/nodejs/node/issues/31973); open at review.
- Upstream labels: `console`, `inspector`.
- Source: [lib/internal/console/constructor.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/console/constructor.js#L530).

## Observable problem or cost

A single console.group call generates both a group event and an extra log event for inspector clients. The JavaScript implementation calls the public, inspector-wrapped log method to print its label.

## Narrow JavaScript change

Route internal label output through an appropriate private output path while keeping public console calls observable. Start with group and groupCollapsed; audit related internal delegation without changing indentation or output formatting.

## History, objections, and ownership

[#31973](https://github.com/nodejs/node/issues/31973) has remained unresolved, with renewed activity in June 2026 and no linked implementation PR. Historical discussion identifies the extra protocol notification. Avoid an unreviewed blanket replacement of public-method calls: monkey-patched Console instances and inspector interception are compatibility considerations.

## Evidence and limits

A local inspector.Session subscribed to Runtime.consoleAPICalled observed [startGroup, log, endGroup] for one group/groupEnd pair on the v27 build. The terminal label printed once. [Console group semantics](https://nodejs.org/api/console.html#consolegrouplabel).

Search terms used with `repo:nodejs/node`: 31973; console group duplicate inspector; console delegation. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

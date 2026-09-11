# Use display width consistently for finite inspect line breaking

- Issue: None found; discovered in source
- Checked: 2026-09-11
- Status: No exact active fix found; formatting and performance agreement needed (at that check)

## What's wrong?

Some util.inspect() line-break decisions count JavaScript string units while other formatting paths count visible width. Combining accents and wide characters can therefore receive inconsistent wrapping.

## Where to start

- Code: [lib/internal/util/inspect.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/util/inspect.js#L2613), [lib/internal/util/inspect.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/util/inspect.js#L2946)
- Docs: [inspect options](https://nodejs.org/api/util.html#utilinspectobject-options)
- Try it / evidence: On local v27 with compact:true and breakLength:25, {a:"aaaaa",b:"aaaaa"} stays on one line, the same visible letters with combining accents split, while five wide CJK characters per value stay on one line. These are observed formatting differences; acceptable width policy and throughput remain to be agreed. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- The source TODO explicitly delays Unicode handling until its performance cost is known. [#26269](https://github.com/nodejs/node/pull/26269) established compact formatting; [#64238](https://github.com/nodejs/node/pull/64238) added the Infinity shortcut. These are related changes, not an exact Unicode-width fix.

Search: `repo:nodejs/node` — isBelowBreakLength; getStringWidth breakLength; inspect unicode width. Recheck activity before starting.

## Suggested fix

- Idea: Use the existing display-width helper for finite line limits, keeping an efficient ASCII path.
- Benefit: Could make wrapping consistent with visible character width.
- Risk: Output and performance may change. Keep the Infinity shortcut; retain current behavior if acceptable cost would require a new public option.

## Next step

Measure finite-width formatting and decide whether the output change and overhead are acceptable.

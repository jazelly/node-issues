# Separate input streams from history configuration before initialization

- Issue: [#61526](https://github.com/nodejs/node/issues/61526) — closed when checked
- Checked: 2026-09-11
- Status: No matching active fix found; previous contributor handed it off (at that check)

## What's wrong?

The positional createInterface(input) form can mistake properties on the input stream for history settings. If onHistoryFileLoaded exists, history setup calls pause() before this.input has been assigned and crashes.

## Where to start

- Code: [lib/internal/readline/interface.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/readline/interface.js#L226), [lib/internal/readline/interface.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/readline/interface.js#L416)
- Docs: [Readline creation contract](https://nodejs.org/api/readline.html#readlinecreateinterfaceoptions)
- Try it / evidence: Readable.from([]) with an onHistoryFileLoaded property, passed positionally to createInterface, throws a TypeError reading pause on both local runtimes. No persistent history files were created. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#61526](https://github.com/nodejs/node/issues/61526) traces the regression to [#58225](https://github.com/nodejs/node/pull/58225). The claimant explicitly left it for others in January 2026.
- [#61538](https://github.com/nodejs/node/pull/61538) closed for inactivity. Review warned that only moving the assignment still mistakes stream properties for history configuration.

Search: `repo:nodejs/node` — 61526; readline history init; onHistoryFileLoaded. Recheck activity before starting.

## Suggested fix

- Idea: Separate the input stream from history options, then initialize history after the input is ready.
- Benefit: Addresses both the undefined-input crash and accidental history setup.
- Risk: Preserve historySize, supplied history, and duplicate removal for the options-object form. Moving the assignment alone is an incomplete fix.

## Next step

Trace both createInterface calling forms and identify the smallest safe setup order.

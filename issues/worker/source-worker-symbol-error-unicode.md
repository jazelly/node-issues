# Preserve Unicode Symbol descriptions during error deserialization

- Issue: None found; discovered in source
- Checked: 2026-09-11
- Status: No exact active fix found; reproduced on both local runtimes (at that check)

## What's wrong?

Symbol error deserialization decodes UTF-8 bytes, then uses the byte count to slice a JavaScript string. Those counts differ for non-ASCII text, leaving an extra closing parenthesis in the Symbol description.

## Where to start

- Code: [lib/internal/error_serdes.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/error_serdes.js#L192)
- Docs: [Worker error-event contract](https://nodejs.org/api/worker_threads.html#event-error)
- Try it / evidence: On both local runtimes, round-tripping Symbol.for("ascii") preserved its description; "é", "中文", and "😀" became "é)", "中文)", and "😀)". The probe used internal/error_serdes with --expose-internals. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#47925](https://github.com/nodejs/node/pull/47925) introduced this branch. Review covered allocation costs and difficult error properties, but did not identify a reason to use byte-based string indexing. No exact earlier Unicode fix was identified.

Search: `repo:nodejs/node` — kInspectedSymbol; serializeError symbol; error_serdes. Recheck activity before starting.

## Suggested fix

- Idea: Decode once and use the decoded string’s length when removing the framing.
- Benefit: Preserves Unicode descriptions with a small change.
- Risk: Keep empty descriptions, the transport tag, and existing Symbol.for behavior. Removing ASCII framing before decoding is another valid approach, but ties parsing more closely to the byte format.

## Next step

Check empty and non-ASCII descriptions, then prepare the string-boundary correction.

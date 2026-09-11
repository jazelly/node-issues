# Make existing header getters reflect raw-array request headers

- Issue: [#64405](https://github.com/nodejs/node/issues/64405) — open when checked
- Checked: 2026-09-11
- Status: No matching active fix found; header behavior needs agreement (at that check)

## What's wrong?

A request can send headers supplied as a raw array while getHeader() and hasHeader() cannot see them. That input path skips the internal map used by the getter methods.

## Where to start

- Code: [lib/_http_client.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_client.js#L534), [lib/_http_outgoing.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_outgoing.js#L844)
- Docs: [Request headers](https://nodejs.org/api/http.html#httprequestoptions-callback), [getHeader](https://nodejs.org/api/http.html#requestgetheadername)
- Try it / evidence: The raw-array path and getter-map mismatch were inspected in pinned source. This pass did not run a new network reproduction. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- July 2026 — [#64405](https://github.com/nodejs/node/issues/64405) proposed calculating a getter view only when needed. Converting the raw headers eagerly was challenged because it can change capitalization, order, and duplicate handling. No implementation PR was identified.

Search: `repo:nodejs/node` — 64405; raw array headers getHeader; header getters. Recheck activity before starting.

## Suggested fix

- Idea: Build a read view for the existing header getters when they are called, keeping the raw array for sending.
- Benefit: Lets getters describe the request without changing the transmitted header sequence.
- Risk: Agree on duplicate values and how the view follows header changes. Eager conversion is simpler, but carries the historical compatibility objection.

## Next step

Agree on duplicate-header results and synchronization before implementing the getter view.

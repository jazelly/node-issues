# Limit idle-socket timeout lookup to its owning pool

- Issue: None found; discovered in source
- Checked: 2026-09-11
- Status: No exact active fix found; behavior and performance checks needed (at that check)

## What's wrong?

On each socket timeout, the HTTP agent searches every idle-socket pool to see whether that socket is idle. It already records which pool owns the socket, so agents serving many origins search unrelated pools unnecessarily.

## Where to start

- Code: [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L461), [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L543)
- Docs: [Agent freeSockets](https://nodejs.org/api/http.html#agentfreesockets)
- Try it / evidence: Pinned source records options._agentKey in createSocket and then does a cross-pool scan in onTimeout. This establishes avoidable search/allocation work, not an application speedup. Keep the proposal only if a many-origin workload benefits without changing timeout/reuse behavior. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#32000](https://github.com/nodejs/node/pull/32000) fixed races involving timed-out or destroyed sockets being reused. Preserve that behavior. The source TODO about destroying active sockets is a separate change, not part of this proposal.

Search: `repo:nodejs/node` — freeSockets timeout; agent onTimeout; removeSocket. Recheck activity before starting.

## Suggested fix

- Idea: Look in the socket’s recorded pool instead of collecting and scanning all pool names.
- Benefit: Could reduce searches and temporary allocations.
- Risk: Confirm the recorded key works for custom agents and HTTPS. Retaining the full scan avoids that assumption, but keeps the extra work.

## Next step

Compare both approaches with custom agents and a many-origin workload.

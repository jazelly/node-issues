# Continue looking for an eligible queued HTTP origin

- Issue: None found; discovered in source
- Checked: 2026-09-11
- Status: No exact active fix found; reproduced with local HTTP servers (at that check)

## What's wrong?

When an HTTP socket closes, the agent looks for another request to serve. It stops searching if the first waiting origin already has a socket, even when a later origin could use spare connection capacity.

## Where to start

- Code: [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L574)
- Docs: [Agent limits](https://nodejs.org/api/http.html#new-agentoptions)
- Try it / evidence: A local three-origin HTTP run used maxSockets=1 and maxTotalSockets=2. After origin C completed, A still occupied one socket; A2 and B1 were queued. After 100 ms the agent had totalSocketCount=1, two pending origins, and B had received no request. All temporary servers and requests were closed. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#33617](https://github.com/nodejs/node/pull/33617) introduced the total socket limit. Review required per-origin limits and protection against dispatching a request twice.
- [#40452](https://github.com/nodejs/node/issues/40452) concerned a different socket-counter bug whose fix already landed. It does not explain the remaining early exit from the scan.

Search: `repo:nodejs/node` — removeSocket; maxTotalSockets starvation; maxTotalSockets pending requests. Recheck activity before starting.

## Suggested fix

- Idea: Skip origins that cannot take a socket and continue looking, checking both connection limits.
- Benefit: Allows a waiting origin to use spare capacity.
- Risk: Preserve same-origin priority and existing request removal; do not dispatch a request twice. Replacing the queues with a global scheduler would change more behavior than needed.

## Next step

Verify eligibility and request removal before changing the scan’s early exit.

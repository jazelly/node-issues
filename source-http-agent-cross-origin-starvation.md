# Continue looking for an eligible queued HTTP origin

- Area: `http`.
- Assessment: **Medium fix; reproduced**.
- Checked: **2026-09-11**.
- Upstream issue: none identified; related historical PRs are references, not a claimed issue number.
- Source: [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L574).

## Observable problem or cost

When a socket closes with no same-origin waiter, removeSocket scans other origins. Encountering any active socket for the first queued origin breaks the entire scan. Another origin can remain blocked while the agent has spare global capacity.

## Narrow JavaScript change

Skip an ineligible origin and continue the scan, checking the actual per-origin and total limits. Preserve same-origin priority and existing free-event request removal. This is not a new global FIFO scheduler.

## History, objections, and ownership

[#33617](https://github.com/nodejs/node/pull/33617) introduced maxTotalSockets. Review specifically required per-origin limits and warned about dispatching a queued request twice; those constraints explain the guard, but not stopping the whole search. [#40452](https://github.com/nodejs/node/issues/40452) and its landed fix address a different historical counter bug. Current source includes that counter fix. No exact active starvation fix was found.

## Evidence and limits

A local three-origin HTTP run used maxSockets=1 and maxTotalSockets=2. After origin C completed, A still occupied one socket; A2 and B1 were queued. After 100 ms the agent had totalSocketCount=1, two pending origins, and B had received no request. All temporary servers and requests were closed. [Agent limits](https://nodejs.org/api/http.html#new-agentoptions).

Search terms used with `repo:nodejs/node`: removeSocket; maxTotalSockets starvation; maxTotalSockets pending requests. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

# Limit idle-socket timeout lookup to its owning pool

- Area: `http`.
- Assessment: **Small performance candidate; source checked**.
- Checked: **2026-09-11**.
- Upstream issue: none identified; related historical PRs are references, not a claimed issue number.
- Source: [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L461), [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L543).

## Observable problem or cost

Each socket timeout allocates all free-pool keys and searches their arrays to determine whether that socket is idle. The socket already has an owning agent key recorded at creation, so a many-origin agent repeatedly scans unrelated pools.

## Narrow JavaScript change

Look up the recorded owning pool and check that array for the socket. Keep the current rule that only idle sockets are destroyed here, and verify that custom Agent subclasses and HTTPS pool keys still refer to the recorded owner. No additional index or cache is needed.

## History, objections, and ownership

[#32000](https://github.com/nodejs/node/pull/32000) deliberately handles reuse races involving timed-out and destroyed sockets. The current TODO about destroying active sockets is a separate behavioral proposal and is excluded. Search results for freeSockets/timeout and removeSocket showed no exact active pool-lookup optimization; current Agent-related open PRs were also screened.

## Evidence and limits

Pinned source records options._agentKey in createSocket and then does a cross-pool scan in onTimeout. This establishes avoidable search/allocation work, not an application speedup. Keep the proposal only if a many-origin workload benefits without changing timeout/reuse behavior. [Agent freeSockets](https://nodejs.org/api/http.html#agentfreesockets).

Search terms used with `repo:nodejs/node`: freeSockets timeout; agent onTimeout; removeSocket. Public issue comments, associated PR discussions, and the open-PR inventory were checked; absence of a matching result does not establish that no private or unlinked work exists. Recheck before starting. See the [research method](RESEARCH.md) for runtime and source-version limits.

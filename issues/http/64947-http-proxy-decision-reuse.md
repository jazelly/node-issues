# Reuse a proxy-routing decision within one request

- Issue: [#64947](https://github.com/nodejs/node/issues/64947) — closed when checked
- Checked: 2026-09-11
- Status: No exact active fix found; nearby proxy PRs require coordination (at that check)

## What's wrong?

HTTP request rewriting and connection creation can both check whether to use the proxy for the same destination. A long NO_PROXY list can make that repeated matching work costly.

## Where to start

- Code: [lib/_http_client.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_client.js#L275), [lib/_http_agent.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/_http_agent.js#L310)
- Docs: [Built-in proxy support](https://nodejs.org/api/http.html#built-in-proxy-support)
- Try it / evidence: Both current JS call sites invoke checkShouldUseProxy. No benchmark or equivalence proof across Agent overrides was performed. Different effective options must still be evaluated independently; the source TODO is not proof that every pair of calls can be merged. [Runtime and research limits](../../RESEARCH.md).

## What's been tried?

- [#64947](https://github.com/nodejs/node/issues/64947) closed because a TODO alone was not a bug report; a concrete proposal was invited. [#62907](https://github.com/nodejs/node/issues/62907) shows why hostname-matching rules matter.
- At the September 2026 check, [#65617](https://github.com/nodejs/node/pull/65617) was changing NO_PROXY matching and [#64528](https://github.com/nodejs/node/pull/64528) was changing configuration selection. Neither was a decision-reuse fix.

Search: `repo:nodejs/node` — 64947; checkShouldUseProxy; proxy decision cache. Recheck activity before starting.

## Suggested fix

- Idea: Reuse the decision within one request when the effective destination and configuration are the same.
- Benefit: Could avoid repeated matching without a long-lived cache.
- Risk: Custom Agent behavior may change the inputs. A cross-request cache would also need invalidation and memory limits, which are outside this small change.

## Next step

Confirm that the two call sites use equivalent inputs, then measure request-local reuse.

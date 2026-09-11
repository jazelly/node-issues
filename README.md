# Node Issues

A personal research notebook of **20 existing Node.js JavaScript logic improvements**. It covers concrete correctness, cleanup, resource-retention, and performance opportunities. It contains no test-only tasks and proposes no new public features.

Reviewed on **2026-09-11** against Node.js main at [`0de4fcceb937`](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67). There are **13 issue-backed follow-ups and 7 source-discovered candidates**. Numbered issues may be closed for inactivity; their current status and history are recorded individually.

## Twenty places to investigate

Start with fd-zero closure or Unicode Symbol deserialization. Difficulty is our assessment, not an upstream good-first-issue label. Reproduced behavior does not by itself mean the proposed replacement is compatible or accepted.

| Order | Improvement | Area | Upstream issue | Readiness / evidence |
| --- | --- | --- | --- | --- |
| 1 | [Close file descriptor zero in file streams](57905-fs-close-file-descriptor-zero.md) | `fs` | #57905 | Small fix; reproduced |
| 2 | [Preserve Unicode Symbol descriptions during error deserialization](source-worker-symbol-error-unicode.md) | `worker` | Source discovery | Small fix; reproduced |
| 3 | [Handle zero-duration timerify histogram samples](41641-perf-hooks-timerify-zero-duration.md) | `perf_hooks` | #41641 | Small fix; reproduced |
| 4 | [Reconstruct genuine native Error objects from worker failures](48716-worker-native-error-deserialization.md) | `worker` | #48716 | Small to medium fix; reproduced |
| 5 | [Separate input streams from history configuration before initialization](61526-readline-history-initialization.md) | `readline` | #61526 | Small to medium fix; reproduced |
| 6 | [Avoid duplicate inspector events from internal console delegation](31973-console-duplicate-inspector-events.md) | `console` | #31973 | Medium fix; reproduced |
| 7 | [Return an async-iterator result from events.on().throw()](source-events-iterator-throw-result.md) | `events` | Source discovery | Small fix; reproduced |
| 8 | [Discard pending values on explicit events.on() cancellation](source-events-iterator-return-buffer.md) | `events` | Source discovery | Small to medium fix; reproduced |
| 9 | [Continue looking for an eligible queued HTTP origin](source-http-agent-cross-origin-starvation.md) | `http` | Source discovery | Medium fix; reproduced |
| 10 | [Reject invalid stream roles before pipeline setup](55305-stream-pipeline-argument-validation.md) | `stream` | #55305 | Small slice of a larger fix; source checked |
| 11 | [Keep env-file watching from widening explicit watch paths](61906-watch-env-file-scope.md) | `watch` | #61906 | Small to medium fix; dormant follow-up |
| 12 | [Clean up a settled once promise when its callback is forwarded](37828-events-once-forwarded-listener-cleanup.md) | `events` | #37828 | Medium fix; reproduced; semantics review |
| 13 | [Make existing header getters reflect raw-array request headers](64405-http-raw-header-getters.md) | `http` | #64405 | Medium fix; design agreement needed |
| 14 | [Resolve inconsistent HTTP pre-socket string accounting](57985-http-buffered-string-accounting.md) | `http` | #57985 | Medium fix candidate; reproduced; compatibility gate |
| 15 | [Avoid retaining request context in lazily created global stdio](48651-async-hooks-lazy-stdio-context-retention.md) | `async_hooks` | #48651 | Medium investigation; current-source follow-up |
| 16 | [Use display width consistently for finite inspect line breaking](source-util-inspect-unicode-line-width.md) | `util` | Source discovery | Medium improvement; observed; performance gate |
| 17 | [Limit idle-socket timeout lookup to its owning pool](source-http-agent-timeout-pool-lookup.md) | `http` | Source discovery | Small performance candidate; source checked |
| 18 | [Reuse a proxy-routing decision within one request](64947-http-proxy-decision-reuse.md) | `http` | #64947 | Small to medium performance candidate |
| 19 | [Avoid a full intermediate byte aggregation in Blob.text()](42265-buffer-blob-text-intermediate-copy.md) | `buffer` | #42265 | Medium performance candidate; memory focus |
| 20 | [Collect inherited error descriptors without repeated prefix copies](source-worker-error-property-collection-cost.md) | `worker` | Source discovery | Small to medium performance candidate; isolated measurement |

The last four entries are performance candidates with explicit evidence gates; only the error-property collection candidate has a local, isolated before/after measurement. The HTTP accounting, header getters, event-forwarding, and Unicode-layout entries also need behavior or compatibility agreement. This is a researched candidate list, not twenty maintainer-approved patches.

## Scope and research

Each file identifies the current JS source, the smallest useful change, relevant Node.js documentation, prior issues/PRs, objections, current activity, and the limits of the evidence. The [research notes](RESEARCH.md) explain the source/runtime distinction and list important rejected candidates.

The owner's existing work, matching active fixes, and recent claims were excluded. Nearby open PRs for different behavior are explicitly identified where relevant. No upstream issue, comment, or PR was created, and no Node.js source was modified.

## File convention

- With an upstream issue: `<issue-number>-<subsystem>-<description>.md`.
- Without one: `source-<subsystem>-<description>.md`; related PR numbers belong inside the file.

The original test-only entries #54111 and #48767 and the unresolved new-separator/API direction #60606 were removed from the current list; they remain in Git history. Add new research directly as Markdown and link it in the table.

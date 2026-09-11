# #55305 — Validate invalid pipeline stream positions

- Upstream: [Handling invalid stream arguments in stream.pipeline()](https://github.com/nodejs/node/issues/55305)
- Checked: 2026-09-11; open, no assignee.
- Upstream labels: `stream`, `never-stale`.
- Area: `stream`.
- Assessment: a small JS validation slice of a larger issue; dormant-PR follow-up.

## Problem and entry point

The issue describes inconsistent validation when readable-only or writable-only streams occupy inappropriate source, transform, or destination positions. Some invalid inputs progress to an unrelated asynchronous TypeError or a pipeline that cannot produce data.

Read [pipeline.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/lib/internal/streams/pipeline.js#L295) and [test-stream-pipeline.js](https://github.com/nodejs/node/blob/0de4fcceb9372b5974f7bacc315d90f82f456d67/test/parallel/test-stream-pipeline.js). The source branch still falls through to `Duplex.from()` for values it does not recognize as readable inputs.

## Smallest useful change

Start with one reported invalid position, such as a writable-only source, and demonstrate its current failure mode. Add the appropriate JS validation and one focused regression test, preserving the existing error-delivery convention. Confirm a valid equivalent pipeline still completes. The complete matrix of classic and Web Streams is beyond this first slice; do not claim this single change resolves the whole issue.

```sh
python3 tools/test.py test/parallel/test-stream-pipeline.js
```

Read the old review before selecting the exact validation rule. No reproduction or patch was performed for this notebook.

## Activity evidence

The author of [PR #62214](https://github.com/nodejs/node/pull/62214) claimed the issue on 2026-03-10. That PR was automatically closed on 2026-08-27 after 120 days without activity; it was not merged. Its closure comment explicitly permits reopening or continuing discussion. The issue timeline and open-PR searches showed no replacement implementation PR. Record this as a follow-up to the previous author's work, not an issue that has never been taken.

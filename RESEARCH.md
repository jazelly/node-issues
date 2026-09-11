# Research notes

## What was checked

Review date: **2026-09-11**. Source reference: [`0de4fcceb9372b5974f7bacc315d90f82f456d67`](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67). Relevant files under lib/ and the matching API documentation were inspected from a read-only export of that commit. Node.js public API pages were also opened. Historical issue bodies, comments, cross-references, PR bodies/reviews, and selected inline review threads were read.

The initial live inventory contained 637 open issues. A later complete open-PR inventory returned 572 entries; titles, bodies, authors, issue references, and relevant diffs were screened. Direct GitHub search queries complemented issue timelines because some older fixes reference a different issue number. The timerify minimum-sample PR #62365, for example, was found through topic search rather than #41641's timeline.

Availability is based on observable public activity. Matching open implementation PRs were excluded even if quiet. A recent statement that an author is splitting or returning to a closed PR was treated as ongoing work. Old stale closures and explicit handoffs were treated as possible follow-ups, with that history retained. A nearby open change to different behavior is disclosed instead of silently equating all work in a file.

## Runtime evidence

Short, isolated probes ran on macOS arm64 with installed **v24.15.0** and a pre-existing local **v27.0.0-pre** binary. The latter was built before this research and is **not claimed to be built from the pinned source commit**. Reproductions establish behavior on those binaries; source inspection independently establishes the relevant code at the pinned revision. A clean build of the pinned revision and upstream CI were not run because no Node patch was made.

Both binaries reproduced fd-zero close omission, Unicode Symbol corruption, non-native deserialized errors, readline initialization failure, once-forwarder listener retention, missing iterator throw result, post-return buffered iteration, and timerify zero-sample failure. Additional local-v27 probes observed duplicate console inspector events, HTTP cross-origin starvation, HTTP string-unit mismatch, and Unicode compact-layout differences. The error-property accumulator experiment compared copied helper variants outside the Node repository; it is deliberately labeled synthetic rather than a whole-runtime speedup.

Watch restarts, raw-header getter behavior, the full pipeline matrix, current AsyncLocalStorage retention, Blob peak-memory improvement, and request-proxy/idle-pool speedups were not freshly demonstrated end to end. Their individual entries state the remaining evidence or compatibility gate. There are no generated upstream patches or permanent probe scripts in this notebook.

## Important exclusions and historical reasons

| Candidate | Why it is not a task in this list |
| --- | --- |
| Owner's [#60295](https://github.com/nodejs/node/issues/60295) / [#65457](https://github.com/nodejs/node/pull/65457), related local work, and [#57584](https://github.com/nodejs/node/issues/57584) | Existing owner work or prior owner investigation. |
| [#54111](https://github.com/nodejs/node/issues/54111), [#48767](https://github.com/nodejs/node/issues/48767) | Test-only work is outside the updated scope. |
| [#60606](https://github.com/nodejs/node/issues/60606) | The proposed configurable separator direction is a public-feature discussion. |
| [#65782](https://github.com/nodejs/node/issues/65782) / [#65783](https://github.com/nodejs/node/pull/65783) | The closed PR's author explicitly plans to return after other active work. |
| [#50885](https://github.com/nodejs/node/issues/50885) / [#65543](https://github.com/nodejs/node/pull/65543) | The author plans to revisit, and review raises the future of async_hooks. |
| Readable map cancellation and queue improvements, [#64429](https://github.com/nodejs/node/pull/64429) / [#65890](https://github.com/nodejs/node/pull/65890) | A real cancellation mismatch was observed, but the author is actively splitting a proposal that already addresses it; a closed umbrella PR is not abandoned work. |
| [#59938](https://github.com/nodejs/node/issues/59938) and multiline history navigation | Nearby active readline fixes [#64126](https://github.com/nodejs/node/pull/64126) and [#65610](https://github.com/nodejs/node/pull/65610) make this a poor independent starter choice. |
| [#55422](https://github.com/nodejs/node/issues/55422) | Recent September 2026 ownership claim; also involves native buffer bounds. |
| [#52818](https://github.com/nodejs/node/issues/52818) / [#52828](https://github.com/nodejs/node/pull/52828) | Changing general string-stream buffering to bytes was rejected for read(n) compatibility; [#52842](https://github.com/nodejs/node/pull/52842) documents code-unit behavior. The separate HTTP entry must address this precedent explicitly. |
| [#56766](https://github.com/nodejs/node/issues/56766) | Windows drive-relative versus absolute semantics are an unresolved compatibility/design issue. |
| [#58487](https://github.com/nodejs/node/issues/58487) | require.resolve paths are resolution starting points, not replacement node_modules directories. |
| [#50707](https://github.com/nodejs/node/issues/50707) | pipeTo returns a promise whose rejection must be handled; changing that is not a simple missing catch in core. |
| [#45189](https://github.com/nodejs/node/issues/45189) | Remaining crypto input-encoding validation has explicit legacy-application compatibility pushback. |
| [#62897](https://github.com/nodejs/node/issues/62897), [#58892](https://github.com/nodejs/node/issues/58892), [#64261](https://github.com/nodejs/node/issues/64261), [#65244](https://github.com/nodejs/node/issues/65244) | Relevant corrections have already landed. |
| [#52677](https://github.com/nodejs/node/issues/52677), [#47645](https://github.com/nodejs/node/issues/47645), [#54180](https://github.com/nodejs/node/issues/54180), [#12606](https://github.com/nodejs/node/issues/12606) | Matching open fixes: #62338, #65634, #54367, and #62525 respectively. |

## How to read the list

No empty search result is treated as proof of universal absence, and no stale closure is treated as maintainer approval. The proposed scope deliberately names constraints that prior attempts encountered. Recheck current comments, assignments, and PRs before taking any candidate; decide on a narrow implementation only after its remaining evidence gate is met.

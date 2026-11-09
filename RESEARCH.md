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

## 2026-11-09 stalled-PR sweep

Source reference: [`128f247ea4cb`](https://github.com/nodejs/node/commit/128f247ea4cb52da405f6cc35dec46c088be8af3). Probes ran on a local `v28.0.0-pre` build of that commit on macOS arm64.

Inputs: open `good first issue` (none were open) and `help wanted` issues; open `confirmed-bug` issues in JavaScript subsystems, each searched for an open PR that mentions it; and about 300 PRs closed unmerged since March 2026 after the stale bot's "closed after 30 days of inactivity" comment. Approved PRs closed unmerged were also screened. Most of those had in fact landed as a squashed or renamed commit, so a PR's closed state alone was not trusted.

Listed as entries 24–28: [#62516](issues/fs/62516-fs-file-mode-int32-abort.md), [#62814](issues/querystring/62814-querystring-lone-surrogate-encoding.md), [test_runner signal exit codes](issues/test_runner/source-test-runner-signal-exit-code.md), [#62418](issues/fs/62418-fs-openasblob-sync-throw.md), and [#59938](issues/repl/59938-repl-multiline-scroll-up.md).

Checked but not listed:

| Item | Reason |
| --- | --- |
| [#55422](https://github.com/nodejs/node/issues/55422) `Buffer.concat` / `copy` past 4 GiB | Five PRs closed as stale. Not reproduced on main: `copy` and `concat` past the 4 GiB boundary returned correct bytes. [#63828](https://github.com/nodejs/node/pull/63828) changed the binding to `size_t` and used `Refs`, so the issue stayed open. Follow-up is closing it or a backport, not a new fix. |
| [#62904](https://github.com/nodejs/node/issues/62904) https proxy `'readable'` listener buildup | Current `establishTunnel` uses `once('readable')`; looks fixed. |
| [#62199](https://github.com/nodejs/node/issues/62199) webstream `writev` `error.filter` | The unguarded call is gone from the adapters. |
| [#58892](https://github.com/nodejs/node/issues/58892) recursive `readdir` with a Buffer path | All three variants returned results on main. |
| [#55077](https://github.com/nodejs/node/issues/55077) `Duplex.from` source not destroyed | Fixed on main. |
| [#33684](https://github.com/nodejs/node/issues/33684) `end(cb)` after end | Closed as resolved after retesting. |
| `validatePort('0')` ([#62836](https://github.com/nodejs/node/pull/62836)) | `dgram.connect('0')` already throws `ERR_SOCKET_BAD_PORT`. |
| [#66349](https://github.com/nodejs/node/issues/66349) `Duplex.from(async () => {})` errors (v26 regression) | Reproduced, but open PR [#66243](https://github.com/nodejs/node/pull/66243) has one approval. |
| [#60295](https://github.com/nodejs/node/issues/60295) `fs.realpath` loops forever on `c/../d` links | Reproduced (sync and callback hang; `.native` works), but open PR [#65457](https://github.com/nodejs/node/pull/65457) is active. |
| [#12606](https://github.com/nodejs/node/issues/12606), [#59431](https://github.com/nodejs/node/issues/59431), [#27880](https://github.com/nodejs/node/issues/27880), [#58654](https://github.com/nodejs/node/issues/58654), [#61958](https://github.com/nodejs/node/issues/61958), [#62981](https://github.com/nodejs/node/issues/62981), [#42743](https://github.com/nodejs/node/issues/42743) | Each has an open PR: #67158, #65610, #65512, #65940, #66673, #62983, #65120. |
| [#39137](https://github.com/nodejs/node/issues/39137) keep-alive timeout after a synchronous POST response | `help wanted`; only a 2021 draft PR exists. Not re-run this time. |

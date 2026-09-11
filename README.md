# Node Issues

A small, manually curated notebook of Node.js core issues worth investigating through JavaScript and focused tests. One Markdown file per upstream issue; this repository does not claim or assign upstream work.

## First five candidates

Checked on **2026-09-11**. Start with #54111, then the narrow validation slice of #55305. Difficulty is our assessment, not an upstream `good first issue` label.

| Priority | Issue | Area | Smallest useful scope | Assessment |
| --- | --- | --- | --- | --- |
| 1 | [#54111](54111-net-test-connect-without-callback.md) | `net`, `test` | Remove an unintended localhost dependency from one socket test | Small test fix; reproduce the host configuration first |
| 2 | [#55305](55305-stream-pipeline-argument-validation.md) | `stream` | Reject one invalid pipeline position with a focused regression test | Small JS slice; follow up on a stale, closed PR |
| 3 | [#48767](48767-net-test-autoselectfamily.md) | `net`, `test` | Investigate the attempted-address assertion in one existing test | Small test surface; timing diagnosis may take longer |
| 4 | [#64405](64405-http-raw-header-getters.md) | `http` | Cover raw-array headers missing from request getters | JS follow-up; getter semantics need agreement |
| 5 | [#60606](60606-readline-jsonl-unicode-separators.md) | `readline`, `repl` | Cover JSONL records containing U+2028/U+2029 | JS follow-up; separator API design remains unresolved |

The last two are conditional follow-ups, not ready-to-code beginner fixes. Five equally straightforward, uncontested fixes were not established in this pass. No C++ changes are proposed here, and no patches or reproduction runs were performed.

## How availability was checked

- Read each open issue, all returned comments, and its cross-reference/connection timeline; none of the five results required another page.
- Checked associated PR states and searched open Node.js PRs by issue number and related terms. No matching open implementation PR was found for these five; unrelated search hits were inspected separately.
- Excluded the owner's current open Node.js PR and issue numbers represented by local work branches.
- Excluded recent ownership claims and existing open implementation PRs. A closed old attempt is a follow-up opportunity, not proof that its author has abandoned the topic.
- Inspected relevant upstream source at [`0de4fcceb9372b5974f7bacc315d90f82f456d67`](https://github.com/nodejs/node/commit/0de4fcceb9372b5974f7bacc315d90f82f456d67).

These are public-activity snapshots, not guarantees about private or unlinked work. Recheck the upstream issue and PRs before starting. Source inspection establishes a plausible JS/test entry point, not a reproduced bug or an approved implementation.

## File convention

Use `<upstream-issue-number>-<subsystem-label>-<short-description>.md`. Keep the exact upstream labels inside each file, along with the issue link, checked date, scope, source entry points, and activity evidence. Add future entries directly to this repository and the table above.

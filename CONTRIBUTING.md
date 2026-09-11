# Contributing research

This repository collects actionable improvements to existing Node.js JavaScript
logic. Contributions are Markdown research entries, corrections, or updates to
existing entries. Keep repository content in English.

## Add or update an entry

1. Search the README and `issues/` for the same issue number or behavior. Update
   an existing entry instead of duplicating it.
2. Copy [templates/issue.md](templates/issue.md) into the appropriate subsystem
   folder and follow the filename convention below.
3. Check the current source, relevant Node.js documentation, previous issues,
   discussions, PRs, and review comments. Search closed work as well as open work.
4. Fill every template section. Keep unknowns explicit, distinguish observations
   from hypotheses, and compare the benefits and risks of plausible approaches.
5. Check assignments, recent claims, matching active PRs, and authors' statements
   that they plan to return to a closed attempt. Record the actual check date.
6. Add or update the README table, its counts, and the affected folder count.
7. Open a PR **in this notebook repository** with a short description of the
   research change and supporting links. Check that Markdown and relative links
   render correctly.

No build, dependency installation, or generated index is needed for a Markdown
contribution. Publishing a notebook entry does not claim an upstream issue or
authorize a Node.js PR, comment, or code contribution; follow Node.js contribution
rules separately when proceeding upstream.

## Folder taxonomy and filenames

Use `issues/<primary-subsystem>/`. Choose the subsystem that owns the code or
behavior the proposed change would primarily modify. If several areas are involved,
keep one entry in one primary folder and record secondary upstream labels inside
it. Do not duplicate a candidate across folders.

The existing folders are `async_hooks`, `buffer`, `console`, `events`, `fs`,
`http`, `perf_hooks`, `readline`, `stream`, `util`, `watch`, and `worker`.
`buffer` includes Blob internals, `watch` covers watch-mode orchestration, and
`worker` includes worker error transport. These are notebook categories; exact
upstream issue labels are recorded independently and may differ.

Add a new lowercase subsystem folder only when the entry does not fit an existing
one, and list its scope and count in the README. Avoid additional status, priority,
difficulty, date, or issue-source subfolders. Those properties can change without
moving an entry.

- Issue-backed entry: `<issue-number>-<area>-<short-kebab-case-description>.md`.
- Source discovery: `source-<area>-<short-kebab-case-description>.md`.
- Do not substitute a historical PR number for a missing issue number.
- When a source discovery gets an upstream issue, rename it using the issue
  number, update repository links, and retain its history in the same entry.

## Required entry standard

Use the template's seven headings in order:

| Section | Required content |
| --- | --- |
| Problem and impact | Trigger, actual result/cost, affected behavior, and why a logic change matters. |
| Evidence and reproduction | Runtime/platform, procedure or source evidence, expected/actual results, and limits. |
| Previous discussion and attempts | Linked history table with date precision, relationship, outcome, and rejection/stall reasons. |
| Solution approaches | At least two meaningful choices with explicit pros and cons; separate editorial analysis from historical positions. |
| Recommendation and boundaries | Preferred approach or unresolved decision, minimal scope, and behavior to preserve. |
| Availability and next step | Dated activity check, linked ownership/work, notebook claim, search scope, and a concrete next action. |
| References | Node.js documentation, pinned source, and any additional evidence. |

Keep the metadata fields: **Area, Kind, Assessment, Last researched, Upstream,
Upstream labels, and Source**. For source discoveries, write `None identified`
for Upstream and `Not applicable` for upstream labels. Use one primary Kind:

- `correctness`: incorrect output, error delivery, validation, or existing behavior.
- `resource-lifecycle`: cleanup, closure, listener retention, or context lifetime.
- `performance`: reduction in measured or plausibly avoidable runtime work or memory.

Assessment is a short plain-language statement of scope, evidence, and remaining
uncertainty. It is not an upstream label or a promise that a patch will be accepted.

## Evidence and ownership rules

- Use `Unknown` for unavailable dates or reasons; use `None identified` only with
  a stated search scope. Do not leave template placeholders in submitted entries.
- Link a previous PR even when closed. Explain whether it landed, was rejected,
  stalled, was superseded, or has an unknown outcome; cite the supporting comment
  when available. An updated timestamp is not the date of that decision.
- Record pushback even when recommending a different approach. Explain what the
  new proposal changes about the earlier tradeoff, or mark the decision unresolved.
- Keep matching active implementations and current claims out of the available
  candidate list. If work starts after an entry is added, retain the history,
  mark it as active with a link, and update its README assessment. Do not silently
  delete the research or continue calling it available.
- A claim in this notebook is separate from upstream ownership. Link any claim
  rather than assuming an author owns a task indefinitely.
- Changing organization or wording does not refresh `Last researched`. Change
  that date only after repeating the relevant source, history, and activity checks.
- Test-only work, new public features, speculative rewrites, and style-only
  changes are outside scope. Reproduction and focused verification are evidence
  for a real logic improvement, not standalone candidates.

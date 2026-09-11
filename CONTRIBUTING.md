# Contributing

Help find small improvements to existing Node.js JavaScript code. Write in
English. New features and test-only tasks are outside this list.

## Add an entry

1. Check the [list](README.md) for duplicates.
2. Copy the [template](templates/issue.md) into `issues/<area>/`, such as
   `issues/fs/` or `issues/stream/`. Use one folder per entry.
3. Answer the five questions in the template. A few sentences per section are
   usually enough. See this [filled example](issues/fs/57905-fs-close-file-descriptor-zero.md).
4. Add a link to the README and update its counts. Submit a pull request (PR)
   to this repository. No build or installation is needed.

Name files like these:

- With a Node.js issue: `57905-fs-close-file-descriptor-zero.md`
- Without one: `source-worker-symbol-error-unicode.md`

Keep the subsystem in the filename and folder. Use an issue number, not a PR
number. The README lists the available folders.

## Keep the useful evidence

- Read the relevant code and Node.js docs. Link source lines at a specific commit
  so readers can find the same code later.
- Search previous issues and PRs, including closed ones. Link what you found and
  briefly explain the outcome and any objections. Closed does not always mean
  rejected; a fix may have landed or simply stalled.
- Check for active fixes or recent claims before listing work as available.
  If someone starts later, update the status and link their work.
- Say what you actually ran and on which Node version. Write “Not reproduced”
  or “Unknown” where needed. For performance claims, include measurements.
- Explain the suggested fix's benefit and risk. A second approach is optional;
  do not invent one just to fill the template.

Use the date you actually checked the research; editing the wording does not
refresh it. More detail is welcome when it helps, but tables and extra categories
are optional. Existing longer entries can keep their supporting research.

These notes do not claim upstream work or imply maintainer approval. Follow
Node.js contribution rules separately when preparing an upstream change.

# 08-Angular — FAQs

> Quick-fire Q&A for 08-Angular. Keep answers short (2-5 lines); link to the relevant topic file for depth.

## Q1. What is the difference between `dependencies` and `devDependencies` in `package.json`?

**A:** `dependencies` lists packages required at runtime (e.g. `@angular/core`, `rxjs`) — they ship as part of the built application. `devDependencies` lists packages only needed during development/build (e.g. `@angular/cli`, TypeScript, testing/linting tools) — they aren't needed by the running app and are typically skipped with `npm install --production`.

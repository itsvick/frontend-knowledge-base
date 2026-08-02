# Commands

> Part of: 16-Git

## Overview

Git commands range from everyday porcelain commands (`add`, `commit`, `push`, `pull`) to more nuanced ones whose subtle differences (like `fetch` vs `pull`, or `merge` vs `rebase`) come up frequently in interviews and in day-to-day team workflows.

## Key Concepts

- `git fetch` only updates local remote-tracking branches (e.g. `origin/main`); it never touches your current branch or working directory.
- `git pull` is `git fetch` + an integration step (`merge` by default, or `rebase` with `--rebase`) into your current branch.
- Remote-tracking branches (`origin/<branch>`) are read-only local bookmarks that reflect the last-known state of the remote; only `fetch`/`pull`/`push` update them.
- Because `pull` integrates immediately, it can trigger unexpected merge commits or conflicts; fetching first lets you inspect incoming changes before deciding how to integrate them.

## Interview Questions & Answers

### Q1. What is the difference between `git pull` and `git fetch`?

#### Answer

`git fetch` downloads all new commits, branches, and tags from the remote into your local repository's remote-tracking branches (e.g. `origin/main`), without touching your current working branch or working directory at all. It's a safe, non-destructive way to see what has changed on the remote before deciding what to do next.

`git pull` is effectively `git fetch` followed immediately by a merge (or rebase, with `git pull --rebase`) of the fetched remote branch into your current local branch. Unlike `fetch`, `pull` does modify your working directory and current branch.

Because `pull` integrates changes immediately, it can create unexpected merge commits or conflicts right when you weren't expecting them. A common, safer workflow is to `fetch` first, inspect what changed (e.g. `git log main..origin/main` or `git diff main origin/main`), and only then merge or rebase deliberately once you know what's incoming.

#### Code Example

```bash
# Fetch only: updates origin/main, does NOT touch your local main or working directory
git fetch origin
git log main..origin/main   # inspect what's new before integrating
git diff main origin/main   # see the actual changes

# Pull: fetch + merge (or rebase) into your current branch, updates working directory
git pull origin main
# equivalent to:
git fetch origin
git merge origin/main
```

#### Follow-up Questions

- What's the difference between `git pull` and `git pull --rebase`?
- What exactly is a remote-tracking branch, and how is it different from a local branch?

## Common Pitfalls

- Running `git pull` out of habit and getting an unexpected merge commit or conflict, instead of fetching first to see what's incoming.
- Assuming `git fetch` updates your working directory or current branch — it never does; only your remote-tracking branches move.
- Forgetting that after a `fetch`, your local branch and its remote-tracking branch can diverge, and only a subsequent `merge`/`rebase`/`pull` reconciles them.

# AI Coding

> Part of: 20-AI

## Overview

AI coding tools span a spectrum from inline autocomplete to fully agentic assistants that can read a repo, make multi-file changes, and run their own verification. Using them well as a frontend engineer means treating AI as an integrated part of the engineering loop — planning, execution, and testing — while keeping the same review rigor, quality gates, and architectural discipline that would apply to any other contributor's code.

## Key Concepts

- Maturity progression: inline autocomplete → agentic multi-file workflows (e.g. Claude Code) → AI applied across the whole SDLC (PR descriptions, test generation, bug investigation) → versioned prompts/instructions (e.g. `CLAUDE.md`) that persist conventions across sessions.
- AI-generated code needs the same review rigor as human-written code — reading every diff, not rubber-stamping — plus upfront constraints (style guide, conventions file) instead of after-the-fact correction.
- AI has characteristic failure modes: overly defensive code, hallucinated APIs, and confidently-wrong logic — a human reviewer should watch for these specifically.
- As AI increases the volume of code a human can produce, automated tests, CI, and smaller PRs become more important, not less, since they're the checks that don't degrade as diff size grows.
- Codebase architecture affects AI token/compute cost directly: well-isolated modules, colocated related code, and a documented conventions file reduce how much context an agent needs to load per task.

## Interview Questions & Answers

### Q1. What is your AI workflow for frontend engineering, beyond just asking an AI coding assistant for snippets?

#### Answer

I think of it as a maturity progression, and the differentiator interviewers are usually probing for is whether AI is used as an integrated part of the engineering loop or just as a fancier autocomplete.

- **Inline autocomplete/snippet generation** is the shallowest use — asking for a single function or completing a line. Useful, but it's single-shot and has no awareness of the rest of the codebase.
- **Agentic workflows** (e.g. Claude Code) are qualitatively different: the agent can read the whole repo, make coordinated multi-file changes, run the test suite or build itself, and iterate on failures without me manually shuttling errors back and forth. This turns AI from a suggestion engine into something closer to a collaborator that can plan and verify its own work.
- **AI across the surrounding SDLC**, not just code — generating and reviewing PR descriptions, drafting test cases from a spec before implementation, summarizing a large diff so a reviewer knows where to focus, or having an agent investigate a bug across the codebase (tracing a stack trace to its root cause) before a human even opens an editor.
- **Treating prompts/instructions as versioned artifacts** — a repo-level config file (like this repo's own `CLAUDE.md`, or an equivalent in a product codebase) that captures conventions, architecture, and constraints once, checked into version control, so every future session follows them instead of me re-explaining context every time.

#### Follow-up Questions

- How do you decide when a task is worth an agentic multi-file session versus a quick inline suggestion?
- What goes into a `CLAUDE.md`-style config file for a real production frontend repo?

### Q2. How do you maintain high code quality standards when generating code using AI tools?

#### Answer

I treat AI output as a strong first draft from a very fast but unreliable junior collaborator, not a finished deliverable. Concretely:

- **Never merge without full review** — I read every diff with the same rigor I'd apply to a human's PR; AI-generated code doesn't get a pass just because it compiles or looks plausible.
- **Give explicit constraints upfront** — existing conventions, a style guide, or a config file (e.g. `CLAUDE.md`) — rather than correcting style after the fact on every single generation.
- **Ask the AI to explain its own reasoning** for non-trivial changes. Having it articulate trade-offs surfaces misunderstandings (wrong assumptions about requirements, a misread of existing code) before they get merged, not after.
- **Run the existing test suite, linter, and type-checker as a hard gate.** Code that "looks right" can still silently break an edge case that a human would have caught out of habit — automated checks don't rely on that habit being triggered.
- **Watch for AI-specific failure modes**: overly defensive code (unnecessary try/catch blocks, null checks for values that can't be null), invented or hallucinated library APIs that don't actually exist, and subtly wrong logic that's written with confident, clean-looking style — which makes it easier to skim past in review than an obviously sloppy human mistake.

#### Follow-up Questions

- Can you give an example of AI-hallucinated API usage you've caught in review?
- How do you calibrate how much to trust AI output on a task you're less familiar with yourself?

### Q3. Given that AI can generate massive pull requests and hundreds of lines of code, what is your approach to frontend testing?

#### Answer

Test coverage matters more, not less, once AI raises the volume of code a human can produce — a human can no longer manually verify every line of a large AI-generated diff, so automated checks have to carry more of that weight.

- **Generate tests alongside the feature code**, in the same session/PR — most AI agents can write reasonable unit tests directly from the implementation. I still review those tests carefully, since AI-written tests can end up mirroring the implementation's logic rather than actually asserting on behavior.
- **Prefer smaller, incremental AI-assisted PRs** over one giant AI-generated PR. A huge diff is hard to review regardless of who or what wrote it; splitting the work keeps each change reviewable and keeps blast radius small if something's wrong.
- **Rely on CI as a strict, non-negotiable gate** — existing unit, integration, and E2E suites must pass. CI is the one check that doesn't degrade as diff size grows, unlike manual review attention.
- **For UI-specific changes, add visual regression testing** (e.g. Chromatic, Percy) or at minimum manually exercise the feature in a browser. Layout issues and real interaction behavior are exactly the class of bug that both unit tests and AI self-review tend to miss.

#### Follow-up Questions

- How do you tell whether an AI-generated test is actually testing behavior versus just re-asserting the implementation?
- What's your threshold for splitting an AI-generated PR into smaller ones?

### Q4. How do you structure frontend architecture to keep AI token spend/compute cost low?

#### Answer

Token cost scales with how much context an agent needs to load per task, so architecture choices that reduce required context directly reduce cost.

- **Smaller, well-isolated modules/components** beat a tangled, highly-coupled codebase — an agent can understand and modify one file or folder without needing to read half the repo to build enough context to make a safe change.
- **Clear naming and colocated related code** (a component with its styles and tests in one folder) reduce the number of exploratory searches an agent needs before it can act, since it doesn't have to hunt across the tree to find everything relevant to a change.
- **A documented architecture/conventions file** — this repo's own `CLAUDE.md` pattern is exactly this idea applied to a knowledge base — that an agent reads once up front is cheaper than having it rediscover conventions by trial and error every session.
- **Caching-friendly workflows** — reusing an agent's already-loaded context for a multi-step task (staying in one long session doing related work) is cheaper than many short sessions that each re-establish context from scratch, since most agent platforms cache/reuse recent context cheaply within a session but not across fresh ones.

#### Follow-up Questions

- How would you refactor a highly-coupled legacy module specifically to make it cheaper for an AI agent to work in?
- What's the trade-off between one long agent session and several shorter, more focused ones?

## Common Pitfalls

- Treating AI as only a snippet generator and never progressing to agentic, multi-file, self-verifying workflows — missing most of the productivity gain available.
- Rubber-stamping AI-generated PRs because the diff "looks clean," skipping the same review rigor applied to human-written code.
- Letting AI-generated diff volume grow unchecked instead of splitting work into smaller, independently reviewable PRs.
- Treating AI-written tests as sufficient without checking whether they actually assert on behavior rather than just mirroring the implementation.
- Working in a highly-coupled codebase with no conventions file, forcing every agent session to re-discover context (and re-spend tokens) from scratch.

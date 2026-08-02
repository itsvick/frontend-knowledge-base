# frontend-knowledge-base

## What this is

A personal knowledge base for frontend interview preparation, organized as
plain Markdown notes. It's a learning/reference repo, not a codebase — there
is no app to build or run here.

## Goal

Build up a structured, comprehensive set of notes covering frontend
(and adjacent) interview topics, from fundamentals through system design and
real-world scenarios, so the content can be reviewed and expanded over time.

## Structure

- Numbered top-level folders (`00-Behavioral` through `29-Interview-Coding`)
  each cover one subject area (JavaScript, TypeScript, React, System Design,
  DSA, LLD/HLD, Security, DevOps, etc.). Some have their own subfolders per
  topic (e.g. `01-JavaScript/Closures`, `01-JavaScript/Promises`).
- `INDEX.md` — table of contents linking to every section.
- `ROADMAP.md` — suggested phase-by-phase learning order (Fundamentals →
  Frameworks → Backend/Platform → Depth → Interview Specific → Ongoing).
- `RESOURCES.md` — external books/blogs/videos/docs worth revisiting.
- `templates/` — Markdown templates to keep new notes consistent:
  - `Question-Template.md` — interview Q&A format (Question, Answer,
    Follow-ups, Code Example, References).
  - `Project-Template.md` / `System-Design-Template.md` — for project write-ups
    and system-design breakdowns.

## Conventions

- When adding a new question, first figure out where it belongs: identify the
  correct top-level section, then the correct subfolder/file within it (e.g.
  "What are the different data types in JavaScript?" belongs in
  `01-JavaScript/01-Basics/README.md`). Then check the existing questions
  already in that file and insert the new one at the appropriate numbered
  position among them based on topic/difficulty ordering, rather than
  defaulting to appending it at the end — renumbering subsequent `### Qn.`
  headings if needed. Only skip this placement/ordering step if the user
  explicitly says where the question should go.
- Before adding a new question, check whether it (or a close variant of it)
  already exists in that file/section. If it does, don't add a duplicate —
  decide instead based on what the new question adds:
  - If the existing question already covers the same ground, skip adding a
    new heading; instead fold in any new value.
  - If the new phrasing/angle is clearly better (e.g. sharper wording, more
    interview-relevant framing) or supersedes the old one, replace the
    existing question heading/content rather than adding a second entry.
  - Apply the same logic to answers: if the existing `#### Answer` is
    missing a point, nuance, or example that the newly given answer
    includes, merge that missing point into the existing answer rather than
    duplicating or fully overwriting it — improve in place.
  - Only keep both as separate questions if they are genuinely distinct
    (different sub-topics or angles worth testing separately), not just
    reworded duplicates.
- Prefer consolidating notes into the section/subfolder's `README.md` rather
  than creating a separate `.md` file per topic — the user does not want a
  proliferation of small files. Only split a topic into its own file if the
  user explicitly asks for it or the section already has real per-file
  content (e.g. `03-HTML/04-Browser.md`, which is itself the whole note, not
  a folder of files).
- If a topic grows large/deep enough that a dedicated file seems genuinely
  worth it (e.g. `README.md` is getting unwieldy, or the topic warrants
  significant depth of its own), don't just create it — ask the user first
  and let them decide before splitting it out.
- When a `README.md` covers multiple Q&A topics, keep one shared
  `## Overview`, one merged `## Key Concepts`, and one merged
  `## Common Pitfalls` for the whole file — but apply
  `Question-Template.md`'s shape **per question, not per file**: each
  question is a `### Qn. {title}` heading under
  `## Interview Questions & Answers`, nesting its own subsections in this
  order: `#### Answer`, `#### Code Example`, `#### Follow-up Questions`,
  `#### References`. Do not pool all code snippets into one shared
  `## Code Examples` section or all sources into one shared `##
  References` section — each question owns its own.
- Per question, only `Answer` is mandatory alongside the question heading
  itself. `Code Example`, `Follow-up Questions`, and `References` are
  optional — omit the subsection heading entirely (don't leave an empty
  `-` placeholder) when there's no real content for it.
- A short sample snippet illustrating the answer can just live inline inside
  `#### Answer` — it doesn't need its own `#### Code Example` subsection every
  time. Reach for a separate `#### Code Example` only when the code is
  substantial enough (e.g. a longer runnable example) to warrant standing on
  its own apart from the explanation.
- New/updated notes should follow the existing template shape: Overview, Key
  Concepts, Interview Q&A (per-question Answer/Follow-ups/Code
  Example/References), Common Pitfalls.
- Most section folders currently contain only a stub `README.md` describing
  suggested files to add — treat these as scaffolding to fill in with the
  above consolidated format, not as a prompt to create one file per
  suggestion.

## When helping here

- Prioritize accuracy and interview-relevance over exhaustive theory.
- Match the existing template structure when adding new notes rather than
  inventing new formats.
- Keep `INDEX.md`/`ROADMAP.md` in sync if new top-level sections are added.

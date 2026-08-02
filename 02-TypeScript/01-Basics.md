# Basics

> Part of: 02-TypeScript

## Overview

TypeScript is a statically-typed superset of JavaScript that compiles down to plain JavaScript, adding an optional type system on top of the language without changing its runtime behavior. It's mainly a compile-time tool: types are checked (and erased) before the code ever runs.

## Key Concepts

- TypeScript is a strict superset of JavaScript — any valid `.js` file is (almost always) valid TypeScript, and TypeScript compiles down to plain JS with no new runtime semantics.
- Type checking happens at compile time, not runtime — types are erased entirely once compiled, so they can't validate genuinely untyped external data (e.g. an API response) without an explicit runtime check.
- Types double as documentation and as a contract between functions/modules/teams, which is especially valuable as codebases and teams grow.
- Adoption can be incremental — `any` and gradual typing let a JS codebase convert to TS file by file rather than all at once.

## Interview Questions & Answers

### Q1. Why do we use TypeScript, and what advantages does it offer over JavaScript?

#### Answer

TypeScript adds a static type system on top of JavaScript, and its main advantages come from moving a class of bugs from runtime to compile time:

- **Catches bugs earlier** — wrong types, typos in property names, and wrong function arguments are flagged by the compiler before the code ever ships, instead of surfacing as a runtime error in production.
- **Better tooling/DX** — because the IDE knows a variable's actual shape, it can offer accurate autocomplete, inline documentation (hovering shows a function's real signature), and reliable "find all references" or safe rename refactors across a large codebase.
- **Types as living documentation** — a function's signature tells you what it expects and returns without needing to read its implementation.
- **Easier collaboration at scale** — types act as a contract between modules or teams, so an integration mistake (e.g. an API response shape changing) is caught at the call site at compile time rather than discovered in production.
- **Incremental adoption** — TypeScript is a superset of JavaScript that compiles down to plain JS, so it introduces no new runtime behavior or language to run, just a compile-time layer. A `.js` file can usually become `.ts` gradually, and `any`/gradual typing let a team migrate a codebase file by file instead of all at once.

This isn't a free win, though: TypeScript doesn't eliminate runtime errors entirely — it can't validate a genuinely untyped external API response without explicit runtime validation (e.g. a schema check) — and it adds a build step plus some ceremony from writing and maintaining type annotations. For most production codebases the bug-prevention and tooling benefits outweigh that cost, but it's a trade-off, not a strictly free upgrade.

```ts
function getUser(id: number) {
  return { id, name: "Alice" };
}

getUser("42"); // TS compile error: Argument of type 'string' is not assignable to parameter of type 'number'.
// In plain JS this would silently run, coercing/mismatching types until something breaks downstream.
```

#### Follow-up Questions

- What is structural typing, and how does it differ from the nominal typing used by languages like Java or C#?
- What does TypeScript compile to, and where does type information go at runtime?
- How would you incrementally introduce TypeScript into an existing large JavaScript codebase?

## Common Pitfalls

- Assuming TypeScript prevents all runtime errors — it can't validate data it has no visibility into, like an untyped third-party API response, without an explicit runtime check.
- Overusing `any` to silence the compiler, which quietly opts that code out of type checking and defeats the purpose of using TypeScript in the first place.
- Treating type annotations as a substitute for tests — types catch shape/argument mismatches, not logic errors.

# RxJS

> Part of: 08-Angular

## Overview

RxJS underpins much of Angular — the HTTP client, reactive forms, and the router all return observables. `Subject` and `BehaviorSubject` are two of its most commonly used multicast primitives, and the difference between them comes up constantly in state-sharing scenarios.

## Key Concepts

- Both `Subject` and `BehaviorSubject` are multicast: every subscriber shares the same underlying execution, unlike a plain cold `Observable`.
- `Subject` has no initial value and does not replay anything to a subscriber that subscribes late — it only sees values emitted *after* it subscribes.
- `BehaviorSubject` requires an initial value at construction and always holds the "current value," which it immediately replays to any new subscriber.
- `BehaviorSubject` exposes `.getValue()` to synchronously read the current value outside of a subscription.
- `BehaviorSubject` is well suited to representing state (e.g. current user, current theme); `Subject` is well suited to representing discrete events (e.g. a button click, a "refresh" signal).

## Interview Questions & Answers

### Q1. What is the difference between `Subject` and `BehaviorSubject`?

#### Answer

A `Subject` is a multicast observable with no concept of a "current value" — it simply forwards whatever is emitted via `.next()` to all currently-subscribed observers. A late subscriber misses anything emitted before it subscribed.

A `BehaviorSubject` is a `Subject` that also stores the last emitted value and requires an initial value up front. Any new subscriber immediately receives that current value on subscription, even if it subscribed after the value was set, and `.getValue()` lets you read it synchronously without subscribing at all. This makes `BehaviorSubject` the natural choice for representing state that components need to read on demand (e.g. "what's the logged-in user right now"), while a plain `Subject` fits one-off events that only matter to listeners who were already subscribed when they happened.

#### Code Example

```ts
const subject = new Subject<number>();
subject.subscribe((v) => console.log("A:", v));
subject.next(1); // A: 1
subject.subscribe((v) => console.log("B:", v)); // B subscribes late
subject.next(2); // A: 2, B: 2 — both get future emissions, B missed 1

const behaviorSubject = new BehaviorSubject<number>(0); // initial value required
behaviorSubject.subscribe((v) => console.log("A:", v)); // A: 0 (current value replayed)
behaviorSubject.next(1); // A: 1
behaviorSubject.subscribe((v) => console.log("B:", v)); // B: 1 — gets current value immediately
console.log(behaviorSubject.getValue()); // 1
```

#### Follow-up Questions

- When would you reach for `ReplaySubject` or `AsyncSubject` instead of either of these?
- Why is `BehaviorSubject` a common backbone for shared state services in Angular?

## Common Pitfalls

- Using a plain `Subject` for state and being surprised that a newly created component never receives the "current" value.
- Forgetting to unsubscribe from a `Subject`/`BehaviorSubject` in a long-lived service consumer, causing memory leaks.
- Calling `.getValue()` on a `BehaviorSubject` from unrelated code as a shortcut instead of subscribing — it works, but bypasses reactivity and can hide stale-data bugs.

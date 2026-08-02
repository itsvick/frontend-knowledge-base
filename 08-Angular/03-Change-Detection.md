# Change Detection

> Part of: 08-Angular

## Overview

Change detection is how Angular keeps the DOM in sync with component state. The strategy a component uses — `Default` or `OnPush` — controls how aggressively Angular re-checks it.

## Key Concepts

- `Default` strategy checks every component in the tree, top-down, on every change detection cycle (triggered by events, timers, XHR/fetch completions, etc. via Zone.js).
- `OnPush` only re-checks a component when: an `@Input()` reference changes, an event originates from within the component (or a child), an `Observable` bound via the `async` pipe emits, or `markForCheck()`/`detectChanges()` is called manually.
- `OnPush` requires treating inputs as immutable — mutating an object/array in place won't trigger a re-check since the reference stays the same.
- `OnPush` is a common performance optimization for large component trees, since it skips subtrees that can't have changed.

## Interview Questions & Answers

### Q1. What is the difference between Default and OnPush change detection strategies?

#### Answer

With the `Default` strategy, Angular re-checks the component on every change detection cycle regardless of whether its data actually changed — safe but potentially wasteful in a large tree.

With `OnPush`, Angular skips a component unless one of these happens: an `@Input()` receives a new object reference, a DOM event fires inside the component's own template, an `async`-piped observable emits a new value, or change detection is triggered manually via `ChangeDetectorRef.markForCheck()`. This requires immutable data patterns — passing a new object/array reference on every update — but can significantly reduce the number of components checked per cycle.

#### Code Example

```ts
@Component({
  selector: "app-user-list",
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<div *ngFor="let u of users">{{ u.name }}</div>`,
})
class UserListComponent {
  @Input() users: User[] = [];
}

// Parent must pass a new array reference for OnPush to detect the change
this.users = [...this.users, newUser]; // triggers re-check
// this.users.push(newUser); // would NOT trigger re-check — same reference
```

#### Follow-up Questions

- How does `OnPush` interact with the `async` pipe?
- When would you manually call `markForCheck()` vs `detectChanges()`?

## Common Pitfalls

- Mutating an `@Input()` object/array in place under `OnPush` and expecting the view to update.
- Forgetting that events from child components with `Default` strategy still bubble up and trigger checks on an `OnPush` ancestor.
- Overusing `detectChanges()` where `markForCheck()` (which just schedules a check) would be enough.

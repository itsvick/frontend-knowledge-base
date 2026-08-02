# Components

> Part of: 08-Angular

## Overview

Angular components are the building blocks of the UI, each with its own template, class, and metadata. Communication between a parent and child component is central to composing them into a working tree.

## Key Concepts

- `@Input()` passes data down from parent to child; `@Output()` + `EventEmitter` emits events up from child to parent.
- A template reference variable (`#child`) lets the parent template call the child's public methods/properties directly.
- `@ViewChild()`/`@ViewChildren()` gives the parent class (not just its template) a direct reference to the child instance.
- A shared service (often backed by an RxJS `Subject`/`BehaviorSubject`) decouples communication between siblings or components with no direct parent-child relationship.
- `ng-content` (content projection) lets a parent project template content into a child.
- Newer Angular versions offer signal-based `input()`, `output()`, and two-way `model()` functions as an alternative to the decorator-based API.

## Interview Questions & Answers

### Q1. How do parent and child components communicate in Angular? Besides `@Input()` and `@Output()`, what other approaches can be used?

#### Answer

The most common approach is `@Input()` for passing data down and `@Output()` (with `EventEmitter`) for emitting events up. Beyond that pair, Angular offers several other ways for components to talk to each other:

- **Template reference variable** — the parent template assigns `#child` to the child element and can then call its public methods or read its properties directly, without any TS code.
- **`@ViewChild()` / `@ViewChildren()`** — the parent's *class* gets a direct reference to the child component instance, so it can call methods or read/write properties imperatively (useful when the interaction can't be expressed as simple input/output bindings).
- **Shared service** — both components inject the same singleton service; the service exposes state via a `Subject`/`BehaviorSubject` that either side can subscribe to or push updates through. This is the standard pattern for sibling components or components that aren't in a direct parent-child relationship.
- **Content projection (`ng-content`)** — the parent projects arbitrary template markup into the child's view, giving it a way to influence the child's rendered output beyond passing plain data.
- **Signal-based `input()` / `output()` / `model()`** — a newer, decorator-free API; `model()` in particular provides built-in two-way binding without manually wiring an `@Input()`/`@Output()` pair.

#### Code Example

```ts
// Parent → child via @ViewChild, and a shared service for unrelated components
@Component({ selector: "app-child", template: `{{ label }}` })
class ChildComponent {
  label = "Hello from child";
  refresh() {
    this.label = "Refreshed!";
  }
}

@Component({
  selector: "app-parent",
  template: `<app-child #child></app-child>
    <button (click)="child.refresh()">Refresh via template ref</button>
    <button (click)="refreshViaViewChild()">Refresh via ViewChild</button>`,
})
class ParentComponent {
  @ViewChild(ChildComponent) child!: ChildComponent;

  refreshViaViewChild() {
    this.child.refresh(); // direct call from the parent class
  }
}

// Shared service for sibling / unrelated components
@Injectable({ providedIn: "root" })
class MessageService {
  private message$ = new BehaviorSubject<string>("");
  message = this.message$.asObservable();
  send(msg: string) {
    this.message$.next(msg);
  }
}
```

#### Follow-up Questions

- When would you reach for a shared service instead of `@Input()`/`@Output()`?
- What's the difference between `@ViewChild()` and a template reference variable?

## Common Pitfalls

- Overusing `@ViewChild()` for things that could be simpler `@Input()`/`@Output()` bindings — it creates tighter coupling between parent and child classes.
- Forgetting that `@ViewChild()` references aren't available until `ngAfterViewInit()`.
- Using a shared service with a plain `Subject` when a `BehaviorSubject` is needed (e.g. a late subscriber never gets the current state).

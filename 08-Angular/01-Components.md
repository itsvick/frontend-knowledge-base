# Components

> Part of: 08-Angular

## Overview

Angular components are the building blocks of the UI, each with its own template, class, metadata, and lifecycle. This note covers how components are decorated and structured, how a parent and child component communicate, the pipes used to transform data in their templates, and how components/services are organized architecturally across an application.

## Key Concepts

- `@Input()` passes data down from parent to child; `@Output()` + `EventEmitter` emits events up from child to parent.
- A template reference variable (`#child`) lets the parent template call the child's public methods/properties directly.
- `@ViewChild()`/`@ViewChildren()` gives the parent class (not just its template) a direct reference to the child instance.
- A shared service (often backed by an RxJS `Subject`/`BehaviorSubject`) decouples communication between siblings or components with no direct parent-child relationship.
- `ng-content` (content projection) lets a parent project template content into a child.
- Newer Angular versions offer signal-based `input()`, `output()`, and two-way `model()` functions as an alternative to the decorator-based API.
- Decorators attach metadata Angular reads to know how to treat a class or member: class decorators (`@Component`, `@Injectable`, `@NgModule`, `@Directive`, `@Pipe`), property decorators (`@Input`, `@Output`, `@ViewChild`, `@ContentChild`, `@HostBinding`), and method decorators (`@HostListener`).
- The component lifecycle runs, in order: `ngOnChanges` → `ngOnInit` → `ngDoCheck` → `ngAfterContentInit` → `ngAfterContentChecked` → `ngAfterViewInit` → `ngAfterViewChecked` → ... → `ngOnDestroy`.
- Pipes (`@Pipe` + `PipeTransform`) transform data for display; pure pipes (the default) only re-run when the input reference changes, impure pipes re-run on every change detection cycle.
- A typical app splits into `core` (singletons/app-wide), `shared` (reusable dumb building blocks), and `features/<name>` (lazy-loaded, domain-scoped) folders, with a smart/container vs. dumb/presentational split within features.

## Interview Questions & Answers

### Q1. What are decorators in Angular, and what are some commonly used ones?

#### Answer

A decorator is a design-time TypeScript feature — a function prefixed with `@`, applied to a class, property, method, or parameter — that attaches metadata for Angular's compiler and runtime to read, telling it how to treat that class or member. Angular's decorators fall into three broad categories:

- **Class decorators** mark what "kind" of Angular building block a class is: `@Component` (a class with its own template/view), `@Injectable` (a class that can be injected via DI, typically a service), `@NgModule` (a module grouping components/directives/pipes/providers), `@Directive` (adds behavior to existing elements without its own template), `@Pipe` (a class implementing `PipeTransform` for template value transformation).
- **Property decorators** bind or expose a class property in a particular way: `@Input()`/`@Output()` (parent-child data/event binding — see Q2), `@ViewChild()`/`@ViewChildren()` (reference to a child component/element in the view — see Q2), `@ContentChild()`/`@ContentChildren()` (reference to content projected via `ng-content`), `@HostBinding()` (binds a class property to a property/attribute/class/style on the component's *own* host element, e.g. `@HostBinding('class.active') isActive = true;`).
- **Method decorators** — `@HostListener()` binds a class method to an event on the host element (or a global target like `window`/`document`), e.g. `@HostListener('click', ['$event'])` runs the decorated method whenever the host element is clicked.

#### Code Example

```ts
@Directive({ selector: "[appHighlight]" })
class HighlightDirective {
  @HostBinding("style.backgroundColor") bgColor = "";

  @HostListener("mouseenter") onEnter() {
    this.bgColor = "yellow";
  }

  @HostListener("mouseleave") onLeave() {
    this.bgColor = "";
  }
}
```

#### Follow-up Questions

- What's the difference between `@ContentChild()` and `@ViewChild()`?
- How would you combine `@HostBinding()` and `@HostListener()` to build a custom "click outside" or hover-highlight directive?

### Q2. How do parent and child components communicate in Angular? Besides `@Input()` and `@Output()`, what other approaches can be used?

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

### Q3. What are the major Angular lifecycle hooks and their execution order?

#### Answer

Angular calls a sequence of lifecycle hook methods on every component/directive as it's created, checked, and eventually destroyed. In order:

1. **`ngOnChanges(changes: SimpleChanges)`** — called before `ngOnInit`, and again on every subsequent change detection run in which at least one `@Input()`-bound property changed. Receives a `SimpleChanges` object with the previous/current values.
2. **`ngOnInit()`** — called once, right after the first `ngOnChanges` (or immediately if there are no bound inputs). The right place for initialization logic (e.g. fetching initial data), rather than the constructor.
3. **`ngDoCheck()`** — called on every change detection run. Used for custom change-detection logic Angular's default check wouldn't catch on its own (e.g. detecting an in-place mutation of an object/array).
4. **`ngAfterContentInit()`** — called once, after Angular has fully initialized the component's projected content (`ng-content`).
5. **`ngAfterContentChecked()`** — called after every check of the projected content.
6. **`ngAfterViewInit()`** — called once, after the component's own view and all child views have been initialized. This is the first point where `@ViewChild()`/`@ViewChildren()` references are guaranteed to be available.
7. **`ngAfterViewChecked()`** — called after every check of the component's view and its child views.
8. **`ngOnDestroy()`** — called just before Angular destroys the component/directive. Used for cleanup: unsubscribing from Observables/Subscriptions, clearing `setTimeout`/`setInterval`, detaching manually-added event listeners — to avoid memory leaks.

#### Code Example

```ts
@Component({ selector: "app-demo", template: `{{ label }}` })
class DemoComponent
  implements OnChanges, OnInit, DoCheck, AfterContentInit, AfterViewInit, OnDestroy
{
  @Input() label = "";

  ngOnChanges(changes: SimpleChanges) {
    console.log("ngOnChanges", changes);
  }
  ngOnInit() {
    console.log("ngOnInit");
  }
  ngDoCheck() {
    console.log("ngDoCheck");
  }
  ngAfterContentInit() {
    console.log("ngAfterContentInit");
  }
  ngAfterViewInit() {
    console.log("ngAfterViewInit");
  }
  ngOnDestroy() {
    console.log("ngOnDestroy");
  }
}
```

#### Follow-up Questions

- Why can't you reliably read `@ViewChild()` references inside `ngOnInit()`?
- When would you need `ngDoCheck()` instead of relying on `ngOnChanges()`?
- What's a common source of memory leaks that `ngOnDestroy()` is meant to prevent?

### Q4. What are custom pipes, and what is the difference between pure and impure pipes?

#### Answer

A pipe is a class implementing the `PipeTransform` interface (its `transform()` method), decorated with `@Pipe({ name: '...' })`, and used in a template as `value | pipeName:arg1:arg2`. Pipes are meant for presentation-only transformations of data (formatting dates/currency, filtering/sorting a list, etc.) — they shouldn't mutate the underlying data, only return a transformed view of it.

Angular pipes come in two flavors, controlled by the `pure` metadata option:

- **Pure pipes** (`pure: true`, the default) — `transform()` only re-runs when Angular's change detection sees the *input reference* change: a new primitive value, or a new array/object reference. This is cheap, since Angular can skip re-running the pipe on most change detection cycles. The catch: it won't detect an in-place mutation of the same array/object (e.g. calling `.push()` on the same array reference) — the pipe simply won't re-run, because the reference didn't change.
- **Impure pipes** (`pure: false`) — `transform()` re-runs on every single change detection cycle, regardless of whether the input actually changed. This does catch in-place mutations, but it's a real performance risk: the pipe re-executes constantly — on every keystroke, every unrelated state change — even when its own input is unchanged.

A commonly cited example is a custom `filterPipe` marked impure so it "reacts" to in-place array mutations — this is generally considered an anti-pattern precisely because of the performance cost. The recommended fix is either to keep the pipe pure and always produce a new array/object reference on updates (immutable updates), or to move the filtering logic into the component/service instead of a pipe.

#### Code Example

```ts
@Pipe({ name: "filterList", pure: true })
class FilterListPipe implements PipeTransform {
  transform(items: string[], search: string): string[] {
    if (!search) return items;
    return items.filter((item) => item.toLowerCase().includes(search.toLowerCase()));
  }
}
```

```html
<!-- Re-runs only when `items` or `search` reference/value changes -->
<li *ngFor="let item of items | filterList: searchTerm">{{ item }}</li>
```

#### Follow-up Questions

- Why is a pure pipe generally preferred over an impure one for filtering/sorting?
- How would you make a pure pipe react correctly to array mutations without turning it impure?
- Which built-in Angular pipes are impure (e.g. `async`), and why does that make sense for them?

### Q5. Can you explain the folder structure and architectural design of an Angular application?

#### Answer

A common convention (module-based or standalone-component era alike) organizes an app around three top-level concerns:

- **`core/`** — singleton services and app-wide config instantiated once for the whole app: the auth service, HTTP interceptors, app-level guards, the root layout/shell. Nothing here should be feature-specific, and features shouldn't reach into each other's internals through it.
- **`shared/`** — reusable, presentation-only building blocks used across multiple features: dumb/presentational components, common pipes, common directives, generic utility functions. Nothing feature-specific lives here either.
- **`features/<feature-name>/`** — one folder per business feature/domain (e.g. `features/orders`, `features/users`), each typically lazy-loaded (via a lazy-loaded route/module or a route-level dynamic `import()`), self-contained, and depending only on `core`/`shared` — not directly on other features.

Within each feature, a further split is common: **smart/container components** fetch data (via services/store) and manage state; **dumb/presentational components** just receive `@Input()`s and emit `@Output()`s, with no knowledge of where the data comes from — they're easier to test in isolation and reuse across features.

With standalone components (Angular 14+, and the default since Angular 17), `NgModule`-based folder boundaries are less rigid — there's no `SharedModule`/`FeatureModule` wrapping things up — but the same `core`/`shared`/`feature-by-domain` convention still holds; it's now driven by which components directly `import` which other standalone components/directives/pipes, rather than which `NgModule` declares them.

A related but double-edged convention is the **barrel file** (`index.ts` re-exporting everything in a folder), which makes imports shorter and more stable when internals move around — but at scale it risks circular-import issues (a barrel in module A imports from module B, which imports back through module A's barrel) and can hurt tree-shaking/build performance, since importing one thing from a barrel can pull in its whole re-exported surface.

#### Code Example

```
src/app/
├── core/
│   ├── interceptors/
│   ├── guards/
│   └── services/
├── shared/
│   ├── components/
│   ├── pipes/
│   └── directives/
└── features/
    ├── orders/
    │   ├── containers/       (smart components)
    │   ├── components/       (dumb components)
    │   └── orders.routes.ts  (lazy-loaded route)
    └── users/
        └── ...
```

#### Follow-up Questions

- How do you decide whether a component belongs in `shared/` versus inside a specific feature folder?
- What's a concrete downside you've seen from over-using barrel files in a large Angular app?
- With standalone components, what replaces the role an `NgModule` used to play in defining a feature's boundary?

## Common Pitfalls

- Overusing `@ViewChild()` for things that could be simpler `@Input()`/`@Output()` bindings — it creates tighter coupling between parent and child classes.
- Forgetting that `@ViewChild()` references aren't available until `ngAfterViewInit()`.
- Using a shared service with a plain `Subject` when a `BehaviorSubject` is needed (e.g. a late subscriber never gets the current state).
- Marking a pipe impure instead of keeping it pure with immutable input updates — impure pipes re-run on every change detection cycle, which is a real performance cost.
- Mutating an array/object in place and expecting a pure pipe (or `OnPush` change detection) to pick it up — both only check for a new input *reference*.
- Forgetting to clean up in `ngOnDestroy()` (unsubscribing Observables, clearing timers) — a common source of memory leaks.
- Over-using barrel files (`index.ts` re-exports) in a large app — convenient imports, but a common source of circular-import bugs and slower builds.

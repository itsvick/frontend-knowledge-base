# Core

> Subsection of 05-React.

## Overview

The core building blocks of React: the component model, JSX, the virtual DOM,
and the props/state data flow that everything else in React is built on top
of.

## Key Concepts

- React apps are built from **components** — functions (or classes) that
  return JSX describing what the UI should look like for a given state.
- The **virtual DOM** is React's in-memory representation of the UI, used to
  minimize expensive direct DOM operations.
- Data flows **unidirectionally**: state lives in a component and flows down
  to children via **props**; children communicate back up via callbacks.
- **Props** are immutable inputs from a parent; **state** is internal,
  mutable (via setters), and owned by the component.

## Interview Questions & Answers

### Q1. What is the Virtual DOM in React? How does it work, and what are its benefits and downsides?

#### Answer

The Virtual DOM is a lightweight, in-memory copy of the real DOM. When state
or props change, React builds a new virtual DOM tree, compares (diffs) it
against the previous one, and computes the minimal set of changes needed —
then applies only those changes to the real DOM.

- **Benefits**: avoids costly direct DOM manipulation, batches and
  optimizes updates, and makes UI code declarative and predictable — you
  describe the desired UI for a given state rather than manually mutating
  the DOM.
- **Downsides**: the diffing/reconciliation process itself has some CPU and
  memory overhead, and for very dynamic, hand-tuned UIs, manual DOM
  manipulation can sometimes outperform it.

#### Follow-up Questions

- What is the diffing algorithm, and how does it decide what changed? (see [[07-Reconciliation]])
- What is reconciliation, and how does it relate to the Virtual DOM?

### Q2. Which function is used to create the root of a React application in React 18+, and how does it differ from the legacy ReactDOM.render?

#### Answer

`createRoot` (from `react-dom/client`) creates a root for a React 18+
application. It replaces the legacy `ReactDOM.render(<App />, container)` API
and opts the app into React 18's concurrent renderer — enabling automatic
batching, transitions (`useTransition`), and the newer Suspense behavior,
none of which the legacy root supports. `ReactDOM.render` still works in
React 18 for backward compatibility, but logs a deprecation warning and runs
the app under the old, non-concurrent renderer.

#### Code Example

```jsx
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

#### Follow-up Questions

- What is `hydrateRoot` used for, and how does it differ from `createRoot`?

### Q3. What is the difference between Shadow DOM and Virtual DOM?

#### Answer

They solve different problems and aren't related implementation-wise:

- **Shadow DOM** is a browser/web-components standard for encapsulating a
  subtree of the real DOM — scoping its styles and markup away from the
  rest of the page. It's about **encapsulation**.
- **Virtual DOM** is a library-level (React) concept: an in-memory tree used
  to diff old vs. new UI state and apply minimal updates to the real DOM.
  It's about **rendering performance**, not encapsulation.

### Q4. What are React Fragments used for?

#### Answer

Fragments (`<>...</>` or `<React.Fragment>`) let a component return multiple
sibling elements without wrapping them in an extra DOM node. This avoids
unnecessary wrapper `div`s that can break CSS layout (e.g. flex/grid) or add
meaningless nesting to the DOM.

#### Code Example

```jsx
function Row() {
  return (
    <>
      <td>Cell 1</td>
      <td>Cell 2</td>
    </>
  );
}
```

### Q5. How are event handlers typically named and attached in JSX?

#### Answer

JSX event props use **camelCase** (`onClick`, `onChange`, `onSubmit`),
mirroring the DOM's event names, and take a **function reference** rather
than an HTML-style string of code. The convention (not enforced by React
itself) is to name the handler function `handleX` and, when a custom
component exposes its own event-like prop, to name that prop `onX` (e.g. a
`Modal` accepting `onClose`).

#### Code Example

```jsx
function Form({ onSave }) {
  function handleClick() {
    onSave();
  }

  return <button onClick={handleClick}>Save</button>;
}
```

### Q6. What are props in React? How are they different from state, and how do you pass data from a parent to a child component?

#### Answer

**Props** ("properties") are inputs passed from a parent component to a
child to configure it — they are read-only from the child's perspective and
owned by the parent. **State** is data owned and managed internally by a
component, which can change over time (typically due to user interaction)
and triggers a re-render when updated. A parent passes data down simply by
setting attributes on the child element in JSX; the child reads them via its
`props` parameter (or destructured arguments).

| | Props | State |
|---|---|---|
| Owner | Parent | The component itself |
| Mutability | Immutable (from child's view) | Mutable via setters |
| Purpose | Configure a component | Track data that changes over time |

#### Code Example

```jsx
function Child({ name }) {
  return <p>Hello, {name}</p>;
}

function Parent() {
  return <Child name="Ada" />;
}
```

### Q7. What are stateless components?

#### Answer

Stateless components don't manage or hold any internal state — they simply
receive data via props and render UI based on that data. They're typically
functional components used for presentational purposes.

### Q8. What are stateful components?

#### Answer

Stateful components manage and hold their own internal state. They can
update their state in response to user interactions or other events, and
React re-renders them whenever that state changes.

### Q9. What is the difference between React's class components and functional components?

#### Answer

Class components are ES6 classes extending `React.Component` that hold
state and lifecycle methods (`componentDidMount`, `componentDidUpdate`,
etc.) directly. Functional components are plain functions that take props
and return JSX; with Hooks (`useState`, `useEffect`, ...), they can now
manage state and side effects too, making them just as capable as class
components while staying simpler and more composable. Functional components
with hooks are the modern default.

### Q10. Explain the React component lifecycle methods in class components.

#### Answer

Class component lifecycle methods fall into three phases:

- **Mounting**
  - `constructor` — initializes state or binds methods.
  - `componentDidMount` — runs once after the component mounts; used for
    API calls, subscriptions, etc.
- **Updating**
  - `shouldComponentUpdate` — decides whether the component should
    re-render.
  - `componentDidUpdate` — runs after an update; used for side effects that
    depend on the new props/state.
- **Unmounting**
  - `componentWillUnmount` — cleanup (removing event listeners, canceling
    timers/subscriptions).

The Hooks equivalent is `useEffect` (mount/update) with a cleanup function
returned from it (unmount).

### Q11. What is the purpose of the callback function argument format of setState() in React class components, and when should it be used?

#### Answer

Instead of passing an object directly, `setState` can take a function of
`(prevState, props) => newState`. This guarantees the update is based on the
most up-to-date state, which matters because `setState` calls can be
batched/asynchronous — reading `this.state` directly in that scenario can
use a stale value. Use the callback form whenever the new state depends on
the previous state (e.g. incrementing a counter).

#### Code Example

```jsx
// Unsafe if called multiple times in the same batch
this.setState({ count: this.state.count + 1 });

// Safe — always based on the latest state
this.setState((prevState) => ({ count: prevState.count + 1 }));
```

### Q12. Explain what happens internally when setState is called in React.

#### Answer

1. **State update is scheduled** — `setState` doesn't mutate state
   synchronously; it enqueues an update object on the fiber for that
   component and calls `scheduleUpdateOnFiber` to tell React work is
   pending.
2. **Batching** — React may batch multiple `setState` calls (e.g. several
   in the same event handler) into a single update for performance, so all
   the enqueued updates are processed together rather than one render per
   call.
3. **Render phase** — React walks the fiber tree from that component down,
   computing the new state (via the queued updates) and building a new
   work-in-progress tree, diffing it against the current one.
4. **Commit phase** — the computed changes are flushed to the real DOM in
   one synchronous pass, and lifecycle methods/effects tied to the commit
   (`componentDidUpdate`, `useLayoutEffect`, `useEffect`) run.
5. **Asynchronous** — because updates are scheduled and batched rather than
   applied instantly, code immediately after `setState` should not assume
   `this.state` already reflects the change.

#### Follow-up Questions

- What's the difference between the render phase and the commit phase? (see [[06-Rendering]])

### Q13. How would you lift state up in a React application, and why is it necessary?

#### Answer

**Lifting state up** means moving state from a child component to its
nearest common ancestor, then passing it back down as props (along with
callbacks to update it) to the components that need it. It's necessary when
two or more sibling components need to share or stay in sync with the same
piece of state, since components can't share state directly with siblings.

### Q14. Explain prop drilling.

#### Answer

Prop drilling is passing data from a parent down through several
intermediate components via props, purely so it can reach a deeply nested
child — even though the intermediate components themselves never use that
data. It makes components harder to refactor and adds noise. Context,
composition, or state management libraries are common ways to avoid it.

### Q15. What is the difference between createElement and cloneElement?

#### Answer

- **`React.createElement(type, props, ...children)`** creates a brand-new
  React element — this is what JSX compiles down to.
- **`React.cloneElement(element, newProps)`** clones an *existing* element,
  merging in new/overridden props while preserving its original children
  (and internal state, since it's still conceptually the same element).
  Useful when you need to inject extra props into a child without
  recreating it from scratch (e.g. inside a component that enhances
  `children`).

#### Code Example

```jsx
React.createElement("div", { className: "container" }, "Hello World");

const element = <button className="btn">Click Me</button>;
const clonedElement = React.cloneElement(element, { className: "btn-primary" });
```

### Q16. What is the role of PropTypes in React?

#### Answer

`PropTypes` provides runtime type-checking for a component's props. During
development, if a prop doesn't match the declared type (or a required prop
is missing), React logs a console warning — catching bugs early without
needing TypeScript.

### Q17. What is the purpose of the `<React.StrictMode>` component?

#### Answer

`StrictMode` is a development-only wrapper that helps surface potential
problems early by intentionally double-invoking certain functions (component
render, state updater functions, and effects along with their cleanup — see
[[02-Hooks]] Q3) and by warning on deprecated/unsafe patterns: legacy string
refs, the legacy context API, and unsafe lifecycle methods
(`componentWillMount`, `componentWillReceiveProps`,
`componentWillUpdate`). It renders no visible UI of its own and has **no
effect in production builds** — it exists purely to catch bugs during
development before they reach users.

## Common Pitfalls

- Mutating `state` directly instead of going through `setState`/a state
  setter — React won't detect the change and won't re-render.
- Reading `this.state` right after calling `setState` and assuming it's
  already updated — updates can be batched/asynchronous.
- Excessive prop drilling instead of reaching for Context or composition
  once nesting gets deep.
- Wrapping JSX in unnecessary extra `div`s when a Fragment would do.

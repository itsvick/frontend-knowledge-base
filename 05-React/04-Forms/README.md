# Forms

> Subsection of 05-React.

## Overview

Handling form input in React comes down to a choice between letting React
state drive the input's value (controlled) or letting the DOM manage it
directly (uncontrolled).

## Key Concepts

- **Controlled components**: the input's value is driven by component
  state, updated via an `onChange` handler — state is the single source of
  truth.
- **Uncontrolled components**: the DOM itself holds the input's value,
  accessed on demand via a `ref`.
- Controlled components make validation, conditional disabling, and
  testing easier since everything flows through state; uncontrolled
  components require less boilerplate for simple cases.

## Interview Questions & Answers

### Q1. What is the difference between controlled and uncontrolled React components?

#### Answer

In a **controlled** component, the form field's value is stored in
component state and updated through an event handler (`onChange`) —
state is the definitive source of truth, and every keystroke goes through
React. In an **uncontrolled** component, the DOM manages the field's value
internally, and React reads it only when needed via a `ref`.

Controlled components offer more control (validation, conditional
behavior) and are easier to test predictably; uncontrolled components need
less code and are fine for simple, one-off forms.

#### Code Example

```jsx
// Controlled
function ControlledInput() {
  const [value, setValue] = useState("");
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
}

// Uncontrolled
function UncontrolledInput() {
  const inputRef = useRef(null);
  const handleSubmit = () => console.log(inputRef.current.value);
  return <input ref={inputRef} />;
}
```

## Common Pitfalls

- Mixing controlled and uncontrolled behavior on the same input (e.g.
  giving an input a `value` prop that can become `undefined`), which
  triggers a React warning.
- Reading a `ref`'s value during render instead of in an event handler or
  effect — the DOM node may not reflect the latest state yet.

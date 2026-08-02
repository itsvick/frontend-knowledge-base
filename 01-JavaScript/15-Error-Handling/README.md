# Error Handling

> Subsection of 01-JavaScript.

## Overview

How JavaScript surfaces and handles runtime errors: `try`/`catch`/`finally`, throwing custom errors, handling errors in async code, and catching whatever slips through as a last resort.

## Key Concepts

- `try`/`catch`/`finally` — `catch` runs only if `try` throws; `finally` always runs, even if `catch` returns or re-throws
- `throw` can throw any value, but throwing an `Error` (or subclass) is preferred — it captures a stack trace, which plain values don't
- Custom error classes (`extends Error`) let you attach domain-specific data and distinguish error types with `instanceof`
- Async code needs its own handling: `try`/`catch` around `await`, or `.catch()` on a Promise chain
- Global handlers (`window.onerror`, `window.onunhandledrejection`) are a last-resort safety net, not a substitute for handling errors where they occur

## Interview Questions & Answers

### Q1. How do you handle errors in JavaScript?

#### Answer

**`try`/`catch`/`finally`** is the basic mechanism: code in `try` runs first; if it throws, control jumps to `catch`; `finally` always runs afterward regardless of whether an error was thrown, caught, or even if `catch` itself returns.

**Throwing your own errors** — use `throw` to signal a failure the caller should handle. You can throw any value, but throwing an `Error` object (or subclass) is strongly preferred over a plain string, since `Error` captures a stack trace that makes debugging much easier.

**Custom error classes** — extending `Error` lets you attach domain-specific data (e.g. which field failed validation) and distinguish error types at the catch site with `instanceof`, so different kinds of errors can be handled differently instead of lumping everything into one generic `catch`.

**Async error handling** — `try`/`catch` works around `await` inside an `async` function, since a rejected awaited Promise throws just like a synchronous error. For a raw Promise chain (no `async`/`await`), use `.catch()` instead.

**Global handlers** — `window.onerror` catches otherwise-uncaught synchronous errors, and `window.onunhandledrejection` catches Promise rejections nobody called `.catch()` on. These are a last-resort safety net (e.g. for logging/monitoring) — they don't replace handling errors close to where they actually happen.

#### Code Example

```js
// 1. try / catch / finally
try {
  const result = JSON.parse("invalid json");
} catch (error) {
  console.log("Something went wrong:", error.message);
} finally {
  console.log("This always runs"); // runs even if catch returns
}

// 2. Throwing your own errors
throw new Error("Invalid input"); // preferred — captures a stack trace
// throw "Invalid input";          // works, but loses the stack trace — avoid this

// 3. Custom error classes
class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.field = field;
  }
}

function registerUser(user) {
  if (!user.email) {
    throw new ValidationError("email", "Email is required");
  }
  if (user.password.length < 6) {
    throw new ValidationError("password", "Password too short");
  }
  return "User registered";
}

try {
  registerUser({ email: "", password: "123" });
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(`Error in ${error.field}: ${error.message}`);
  } else {
    console.log("Unexpected error:", error.message);
  }
}

// 4. Async error handling
async function getData() {
  try {
    const res = await fetch("invalid-url");
    const data = await res.json();
    return data;
  } catch (error) {
    console.log("Fetch failed:", error.message);
  }
}

// Same thing, as a Promise chain instead of async/await
fetch("invalid-url")
  .then((res) => res.json())
  .catch((err) => console.log("Fetch failed:", err.message));

// 5. Global error handlers — last-resort safety net
window.onerror = function (message, source, lineno, colno, error) {
  console.log("Global error:", message);
};
window.onunhandledrejection = function (event) {
  console.log("Unhandled promise:", event.reason);
};
```

#### Follow-up Questions

- Why does `finally` still run even if `catch` (or `try`) contains a `return`?
- Why is throwing an `Error` object preferred over throwing a plain string or other value?
- How would you distinguish and handle multiple custom error types in one `catch` block?

## Common Pitfalls

- Empty `catch` blocks (`catch (e) {}`) — the error is silently swallowed, making bugs far harder to track down later.
- Throwing plain strings/values instead of `Error` objects, losing the stack trace that makes debugging tractable.
- Forgetting that a Promise chain without `.catch()` (or an `await` without surrounding `try`/`catch`) results in an unhandled rejection.
- Relying on global handlers (`window.onerror`/`window.onunhandledrejection`) as the primary error-handling strategy instead of handling errors close to where they occur.

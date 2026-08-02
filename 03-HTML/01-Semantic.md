# Semantic

> Part of: 03-HTML

## Overview

Semantic HTML elements (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`, `<figure>`, `<time>`, etc.) describe the meaning and role of the content they wrap, rather than just how it should look — unlike generic `<div>`/`<span>` containers, which carry no inherent meaning. Using the right semantic element gives assistive technology, search engines, and other developers a clear, self-documenting structure to work with.

## Key Concepts

- Semantic elements describe meaning/role (`nav`, `header`, `main`, `article`, `section`, `aside`, `footer`, `figure`, `time`), while `<div>`/`<span>` are generic containers with no built-in meaning, used only as styling/scripting hooks.
- Accessibility: screen readers expose semantic elements as navigable "landmarks," letting keyboard/screen-reader users jump straight to nav/main/footer instead of reading through a flat wall of divs.
- SEO: search engine crawlers give more meaningful weight to content inside `<article>`, heading tags (`<h1>`-`<h6>`), etc. than to generic containers, improving how content is indexed and understood.
- Maintainability: a semantically structured page documents its own layout intent — a developer can scan tag names to understand the page's structure instead of relying on class names alone.
- Semantic tags carry no styling of their own by default — they still need CSS, but their meaning is independent of their appearance.

## Interview Questions & Answers

### Q1. What are semantic tags in HTML, and why do we use them?

#### Answer

Semantic HTML elements — `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`, `<figure>`, `<time>`, etc. — describe the meaning and role of the content they contain, not just how it should be styled or positioned. A `<nav>` tells the browser (and anyone reading the markup) "this block is site navigation," whereas a `<div class="nav">` is just a generic box styled to look like navigation — the browser and assistive tech have no way to know its purpose.

This matters for three main reasons:
- **Accessibility** — screen readers expose semantic elements as navigable landmarks, so a screen reader user can jump straight to the main content or navigation instead of reading through a flat structure of divs to find what they need.
- **SEO** — search engine crawlers give more meaningful weight to content inside `<article>`, heading tags, etc. than to generic containers, which can improve how a page's content is indexed and understood.
- **Maintainability** — a page built with semantic tags is self-documenting: a developer can scan the tag names and immediately understand the page's structural intent, rather than inferring it from arbitrary class names.

#### Code Example

```html
<!-- Before: div soup -->
<div class="header">
  <div class="nav">
    <div class="nav-item">Home</div>
    <div class="nav-item">About</div>
  </div>
</div>
<div class="content">
  <div class="post">
    <div class="post-title">My Post</div>
    <div class="post-body">...</div>
  </div>
</div>
<div class="footer">© 2026</div>

<!-- After: semantic equivalent -->
<header>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>
<main>
  <article>
    <h1>My Post</h1>
    <p>...</p>
  </article>
</main>
<footer>© 2026</footer>
```

#### Follow-up Questions

- What's the difference between `<section>` and `<article>`?
- Is it ever fine to use a `<div>` instead of a semantic element?

## Common Pitfalls

- Using `<div>`/`<span>` for everything and retrofitting ARIA roles instead of reaching for the native semantic element that already carries that meaning.
- Overusing `<section>` as a generic styling wrapper — it's meant for grouping thematically related content that would make sense in a document outline, not just a div replacement.
- Including more than one `<main>` (or multiple top-level `<header>`/`<footer>`) on a single page.
- Assuming semantic tags alone make a page accessible — they help, but don't replace proper ARIA usage, keyboard support, and focus management for interactive widgets.

# 03-HTML — FAQs

> Quick-fire Q&A for 03-HTML. Keep answers short (2-5 lines); link to the relevant topic file for depth.

## Q1. What is a MIME type?

**A:** A MIME type (e.g. `text/html`, `application/json`, `image/png`) is a string that tells a browser/HTTP client how to interpret a resource's content, independent of its file extension. The browser decides how to render/parse a response based on its `Content-Type` header, not the URL's file extension. Servers set this header so browsers know whether to render a response as HTML, parse it as JSON, display it as an image, or prompt a download.

## Q2. What is the purpose of `<!DOCTYPE html>`?

**A:** It's a required instruction (not an HTML tag) at the very top of an HTML document that tells the browser to render the page in standards mode (following the modern HTML5/CSS spec) rather than "quirks mode" — an old backwards-compatibility rendering mode that mimics inconsistent legacy browser behavior, e.g. different box-model calculations. Omitting it, or using a malformed doctype, can trigger quirks mode and cause subtly broken layout.

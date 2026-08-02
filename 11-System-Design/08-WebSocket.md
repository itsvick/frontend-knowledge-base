# WebSocket

> Part of: 11-System-Design

## Overview

WebSocket is a full-duplex, persistent, bidirectional connection protocol — one option among several (alongside polling and Server-Sent Events) for pushing real-time updates from a server to a client, each with different tradeoffs in complexity and directionality of data flow.

## Key Concepts

- Polling repeatedly requests on an interval — simplest to implement, but wasteful (requests fire even when nothing has changed) and adds latency (average delay of half the poll interval).
- WebSockets provide a full-duplex, persistent, bidirectional connection, needed when the client must send frequent messages back to the server over the same connection.
- WebSockets add operational complexity: connection state, reconnection/backoff logic, and a stateful server (or a pub/sub layer like Redis) when scaling horizontally across multiple instances.
- SSE (Server-Sent Events) is a one-way, server-to-client stream over a regular long-lived HTTP connection — simpler than WebSockets, auto-reconnects natively via `EventSource`, and works through standard HTTP infrastructure/proxies more easily.
- Most LLM streaming APIs (e.g. OpenAI/Anthropic streaming completions) use SSE-style streaming under the hood, since the interaction pattern — one prompt in, a token-by-token response out — is inherently one-directional per exchange.

## Interview Questions & Answers

### Q1. If integrating an LLM-powered chatbot into a frontend, which real-time protocol would you choose — Polling, WebSockets, or Server-Sent Events (SSE) — and why?

#### Answer

**Polling** (repeatedly requesting on an interval) is the simplest option but is rarely the right choice for a chat-like UX: it's wasteful (constant requests fire even when there's nothing new) and adds latency, since the average delay is half the poll interval.

**WebSockets** provide a full-duplex, persistent, bidirectional connection. They're necessary when the client needs to send frequent messages back to the server over the same connection — e.g. typing indicators, or multi-user chat where other users' messages need to be pushed to you. The tradeoff is added complexity: you have to manage connection state, reconnection/backoff logic, and a stateful server (or a pub/sub layer like Redis) if you scale horizontally across multiple server instances.

**SSE (Server-Sent Events)** is a one-way, server-to-client stream over a regular long-lived HTTP connection. It's a strong fit specifically for LLM chatbot responses, because the actual interaction pattern is "client sends one prompt, server streams back a token-by-token response" — inherently one-directional per exchange. SSE is simpler than WebSockets: it's built on plain HTTP, auto-reconnects natively via the browser's `EventSource` API, and works through standard HTTP infrastructure/proxies more easily. It's also what most LLM streaming APIs (e.g. OpenAI/Anthropic streaming completions) use under the hood.

**Recommendation**: SSE is usually the better default for an LLM chatbot's streamed responses specifically, since the interaction is fundamentally request-then-stream-response rather than truly bidirectional live messaging. WebSockets become the better choice only if the product also needs genuinely bidirectional real-time features beyond the LLM response itself — e.g. multi-user presence, or live collaborative editing alongside the chat.

#### Follow-up Questions

- How would you handle reconnection if an SSE stream drops mid-response from the LLM?
- If the product later adds multi-user presence alongside the chatbot, would you migrate the whole chat connection to WebSockets, or run SSE and WebSockets side by side?

## Common Pitfalls

- Defaulting to WebSockets for LLM chat streaming without needing bidirectional messaging, taking on reconnection/scaling complexity (sticky sessions, pub/sub fan-out) that SSE would have avoided.
- Using polling for chat-like features, causing both wasted requests and perceptibly laggy delivery (up to a full poll interval of latency).
- Forgetting that SSE is one-directional — if the UI later needs the server to receive frequent client-originated events on the same channel (not just the initial prompt), SSE alone won't cover it.

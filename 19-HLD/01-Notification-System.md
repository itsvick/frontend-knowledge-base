# Notification System

> Part of: 19-HLD

## Overview

A real-time notification system (Slack, Discord) pushes events — new
messages, mentions, presence changes — to connected clients with low
latency, in the right order, and without losing anything for a client that
was disconnected when the event happened. The hard parts aren't the happy
path; they're reconnection, ordering, and delivering to a client that
wasn't there when the event fired.

## Key Concepts

- **Transport choice** — WebSocket (persistent, bidirectional) vs SSE
  (one-way, server-to-client, auto-reconnecting) vs polling (simplest,
  highest latency/waste). See [WebSocket](../11-System-Design/08-WebSocket.md)
  for the general trade-offs.
- **Reconnection strategy** — exponential backoff with jitter, a heartbeat
  to detect a dead connection faster than TCP's own timeout, and a
  "resume from last seen event" handshake on reconnect instead of a full
  state refetch.
- **Message ordering** — server-assigned monotonic sequence numbers (not
  client timestamps, which are subject to clock skew), applied per-channel
  so ordering only needs to hold within a conversation, not globally.
- **Offline delivery** — a durable per-user queue server-side for
  reconnect-and-catch-up, plus mobile push (APNs/FCM) as the wake-up
  mechanism for a client that's fully offline (app closed).
- **Fan-out** — a pub/sub layer (Redis pub/sub, Kafka) between the event
  source and the WebSocket gateway tier, so an event can reach a user's
  connection regardless of which gateway server instance is holding it.

## Interview Questions & Answers

### Q1. How would you design a real-time notification system like Slack or Discord?

#### Answer

**Transport.** WebSocket is the right default here — unlike a pure
notification feed, Slack/Discord also need client→server traffic on the
same connection (typing indicators, presence, read receipts), which rules
out SSE's one-way model. Plain polling is only acceptable as a last-resort
fallback for networks/proxies that block WebSocket upgrades. The trade-off
is operational complexity: a stateful connection per client means the
server tier has to track *which* server instance holds *which* user's
socket.

**Reconnection strategy.** Networks drop connections constantly (mobile
handoff, laptop sleep, proxy timeouts), so reconnection isn't an edge case
— it's core to the design:

- **Exponential backoff with jitter** on reconnect attempts, capped at a
  max delay, so a server-side outage doesn't get hammered by every client
  retrying in lockstep.
- **Heartbeat (ping/pong)** on an interval shorter than typical
  intermediary/proxy idle-timeouts, so a half-dead connection (TCP still
  "open" but nothing flowing) is detected and torn down quickly instead of
  silently dropping messages until the OS-level timeout finally fires.
- **Resume, don't restart** — on reconnect, the client sends the last
  sequence number/event id it successfully processed; the server replays
  everything after that point from its durable queue rather than the
  client re-fetching full state from scratch.

**Message ordering.** Ordering is scoped **per channel/conversation**, not
globally — the server assigns a monotonically increasing sequence number
per channel when it accepts an event (never trust client-supplied
timestamps, which drift with clock skew). The client buffers incoming
events and applies them in sequence order; if it detects a gap (sequence
5 then 8), it knows two events were missed and requests a backfill for the
missing range instead of silently rendering out of order.

**Offline delivery.** Two distinct "offline" cases need different
handling:

- **Client reconnects while the app is still open/backgrounded** — the
  resume handshake above (replay from last-seen sequence number out of a
  durable per-user queue) covers this.
- **Client is fully closed (mobile app killed, laptop off)** — a
  WebSocket obviously can't reach it. Mobile push (APNs/FCM) is used purely
  as a wake-up signal ("you have new activity"); the app then does a normal
  fetch-and-catch-up on foreground rather than trying to deliver full
  message content through the push payload itself.

**Fan-out architecture.** A single event (a message posted in a channel)
may need to reach many recipients' sockets, potentially held on different
gateway server instances. A pub/sub layer (Redis pub/sub or Kafka) sits
between the event source and the WebSocket gateway tier: the gateway
holding a given user's connection subscribes to that user's/channel's
topic, so the event source doesn't need to know which physical server
holds which socket — it just publishes once and every subscribed gateway
instance delivers to its own local connections.

#### Follow-up Questions

- How would you dedupe a notification if the reconnect-replay and a live
  push both deliver the same event to the client?
- How would you extend this design to support read receipts / delivery
  receipts?
- How does presence (online/offline/typing) fit into this same
  architecture, and does it need the same ordering guarantees as messages?

## Common Pitfalls

- Using client timestamps for ordering instead of a server-assigned
  sequence number, which breaks under clock skew between clients.
- Treating reconnection as a full state refetch every time instead of a
  targeted "resume from last sequence number" replay, which doesn't scale
  as channel history grows.
- No heartbeat, so a half-dead connection sits "open" for minutes before
  the OS-level TCP timeout finally triggers a reconnect.
- Trying to deliver full notification content through a mobile push
  payload instead of treating push as just a wake-up signal.
- A single WebSocket gateway instance with no pub/sub fan-out layer,
  which can't scale horizontally since it has no way to reach a
  connection held by another instance.

## References

- [MDN: WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- [MDN: Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)

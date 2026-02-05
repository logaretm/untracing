# 🔭 Untracing

> Naming Pattern Registry for Diagnostics Channels

Untracing provides a unified naming registry and standard to ensure that library authors and observability tools speak
the same language when using [Diagnostics Channels](https://nodejs.org/api/diagnostics_channel.html) and [Tracing Channels](https://nodejs.org/api/diagnostics_channel.html#class-tracingchannel).

> [!NOTE]
> This document is a work in progress and not finalized yet.

---
 
## Diagnostics Channels

Node.js Diagnostics Channels (also supported in [Bun](https://bun.com/reference/node/diagnostics_channel), [Cloudflare Workers](https://developers.cloudflare.com/workers/runtime-apis/nodejs/diagnostics-channel/) 
and [Deno](https://docs.deno.com/api/node/diagnostics_channel/)) offer two ways to emit telemetry:

**1. Diagnostic Channel (Atomic)**

A single, named event stream from the `Channel` class. Useful for specific, point-in-time notifications(e.g., `undici:request:bodySent`).

- **Naming:** Arbitrary, and can be chosen freely (e.g., `undici:request:bodySent`)
- **Untracing Goal:** Standardize these to follow the same prefixing as Tracing Channels.

**2. Tracing Channel (Lifecycle)**

A collection of five related `Channel` instances representing the execution lifecycle of a single traceable action.
When you create a `TracingChannel` with the name `my-op`, Node.js automatically manages:

- `tracing:my-op:start`
- `tracing:my-op:end`
- `tracing:my-op:asyncStart`
- `tracing:my-op:asyncEnd`
- `tracing:my-op:error`

The prefix (`tracing:`) and the `eventType` suffix (`:start`, `:end`, etc.) are **hardcoded** by the Node runtime.

## Naming Pattern

To allow Application Performance Monitoring (APM) tools (OpenTelemetry, etc.) to automatically track operations across different libraries,
all `Channel` instances (standalone or as part of `TracingChannel`) should follow this strict pattern:

```
tracing:{namespace}.{operation}:{eventType}
```

**Pattern Breakdown**

- **Namespace:** The package or module name (e.g., `unstorage`, `h3`, `undici`).
- **Operation:** The entity being acted upon (e.g., `request`, `file`, `query`). Use full nouns, not abbreviations.
- **Event Type:** The lifecycle hook (e.g., `start`, `end`).

Example:

```ts
import diagnostics_channel from 'node:diagnostics_channel';

const standaloneChannel = diagnostics_channel.channel('tracing:{namespace}.{operation}:{eventType}');
// or
const tracingChannel = diagnostics_channel.tracingChannel('{namespace}.{operation}');

```

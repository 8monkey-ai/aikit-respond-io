# @8monkey/aikit-respond-io

A library to help setup webhook for respond.io integration using a clean, event-based API.

## Motivation

Integrating with third-party webhooks often requires repetitive boilerplate: verifying signatures to ensure security, handling strict timeouts, and parsing untyped payloads. This library abstracts those complexities, providing a secure, type-safe, and resilient toolkit.

## Features

- **[Webhook Handler](#webhook-handler)**: Securely process incoming webhooks with signature verification, automatic background processing, and type-safe payloads.
- **[Integrations](#integrations)**: Ready-to-use examples for popular platforms, including [Vercel AI SDK](#vercel-ai-sdk), [Express](#express), [Hono](#hono), [ElysiaJS](#elysiajs) and [AWS Lambda Function URL](#aws-lambda-function-url).

> **Note:** This package is designed to complement the official [`@respond-io/typescript-sdk`](https://www.npmjs.com/package/@respond-io/typescript-sdk). While the official SDK provides a comprehensive API client, this package focuses on robust webhook handling and integration utilities.

## Installation

```bash
bun add @8monkey/aikit-respond-io
```

## Compatibility

This package is published as an ESM-only module. It is optimized for use with modern frameworks and bundlers (e.g., Next.js, Vite, Bun) that handle module resolution. If you are using it in a pure Node.js environment, ensure your runtime configuration supports ESM and the appropriate module resolution strategy.

## Webhook Handler

The `webhook` factory creates secure handlers that verify signatures and process events in the background.

### Basic Usage

Create a handler and define your logic.

```ts
import {
  webhook,
  MessageReceivedPayload,
} from "@8monkey/aikit-respond-io/webhook";

// 1. Create the handler
const onMessage = webhook<MessageReceivedPayload>({
  signingKey: process.env.RESPOND_IO_SIGNING_KEY!,
  handle: async (payload) => {
    console.log("New message from contact", payload.contact.id);
    // Your logic here...
  },
  onError: (err) => console.error("Webhook failed:", err),
});

// 2. Use it in your server (e.g., Hono, Next.js, Workers)
// It exposes a standard Fetch API compatible method
app.mount("/webhook/message-received", onMessage.fetch);
```

### Supported Events

The `webhook` handler supports various event types from Respond.io. Here are some of the most common ones:

- `MessageReceivedPayload`: When a new message is received from a contact.
- `ConversationClosedPayload`: When a conversation is closed.
- `MessageSentPayload`: When a message is successfully sent to a contact.
- `ContactAssigneeUpdatedPayload`: When a contact's assignee changes.

To handle a specific event, specify its payload type, example for new incoming message event:

```ts
const onMessage = webhook<MessageReceivedPayload>({
  signingKey: process.env.RESPOND_IO_SIGNING_KEY!,
  handle: async (payload) => {
    console.log("New message from contact", payload.contact.id);
  },
});
```

## Integrations

### Vercel AI SDK

Easily convert Respond.io payloads into AI SDK messages.

```ts
import { toModelMessage } from "@8monkey/aikit-respond-io/vercel-ai";
import { generateText } from "ai";

// Inside your webhook handle function:
handle: async (payload) => {
  const userMessage = toModelMessage(payload.message.message, "user");

  const { text } = await generateText({
    model: yourModel,
    messages: [userMessage],
  });
};
```

### Express

Integrate with an Express.js application.

```ts
import {
  webhook,
  MessageReceivedPayload,
} from "@8monkey/aikit-respond-io/webhook";

import { createMiddleware } from "@hattip/adapter-node";
import express from "express";

const onMessage = webhook<MessageReceivedPayload>({
  signingKey: process.env.RESPOND_IO_SIGNING_KEY!,
  handle: async ({ contact }) => console.log(`Message from ${contact.id}`),
});

const app = express();
app.use("/webhook/message-received", createMiddleware(onMessage.fetch));

app.listen(3000);
```

### Hono

Integrate with a Hono application.

```ts
import {
  webhook,
  MessageReceivedPayload,
} from "@8monkey/aikit-respond-io/webhook";
import { Hono } from "hono";

const onMessage = webhook<MessageReceivedPayload>({
  signingKey: process.env.RESPOND_IO_SIGNING_KEY!,
  handle: async ({ contact }) => console.log(`Message from ${contact.id}`),
});

const app = new Hono();
app.mount("/webhook/message-received", onMessage.fetch);

export default app;
```

### ElysiaJS

Integrate with an ElysiaJS application.

```ts
import {
  webhook,
  MessageReceivedPayload,
} from "@8monkey/aikit-respond-io/webhook";
import { Elysia } from "elysia";

const onMessage = webhook<MessageReceivedPayload>({
  signingKey: process.env.RESPOND_IO_SIGNING_KEY!,
  handle: async ({ contact }) => console.log(`Message from ${contact.id}`),
});

new Elysia().mount("/webhook/message-received", onMessage.fetch).listen(3000);
```

### AWS Lambda Function URL

Deploy your webhook handler as a standalone AWS Lambda function.

```ts
import {
  webhook,
  MessageReceivedPayload,
} from "@8monkey/aikit-respond-io/webhook";

import awsLambdaAdapter from "@hattip/adapter-aws-lambda";

const onMessage = webhook<MessageReceivedPayload>({
  signingKey: process.env.RESPOND_IO_SIGNING_KEY!,
  handle: async ({ contact }) => console.log(`Message from ${contact.id}`),
  // In serverless, we should wait for the handler to finish before returning the response
  waitForCompletion: true,
});

export const handler = awsLambdaAdapter(onMessage.fetch);
```

## Contributing

We welcome contributions!

- **Bug Reports & Feature Requests**: Please use the [issue tracker](https://github.com/8monkey-ai/aikit-respond-io/issues) to report bugs or suggest features.
- **Pull Requests**: Specific fixes and improvements are welcome. Please open a PR.

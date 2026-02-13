---
name: claude-sdk
description: "Build features using @anthropic-ai/sdk: tool-use patterns, prompt caching, multi-turn conversations, streaming, and error handling. Use when integrating Claude API programmatically or creating new agent tools."
metadata:
  {
    "openclaw":
      {
        "emoji": "🤖",
      },
  }
---

# Claude SDK Integration

Patterns for building features with `@anthropic-ai/sdk` inside OpenClaw.

## Setup

The SDK is already a dependency. Import:

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});
```

## Tool-Use Pattern

Define tools with JSON schema, call them in a loop:

```typescript
const tools: Anthropic.Tool[] = [
  {
    name: "search_files",
    description: "Search for files matching a pattern",
    input_schema: {
      type: "object" as const,
      properties: {
        pattern: { type: "string", description: "Glob pattern" },
      },
      required: ["pattern"],
    },
  },
];

const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 4096,
  tools,
  messages: [{ role: "user", content: prompt }],
});

// Handle tool calls
for (const block of response.content) {
  if (block.type === "tool_use") {
    const result = await executeToolCall(block.name, block.input);
    // Continue conversation with tool result
  }
}
```

## Prompt Caching

Cache expensive system prompts (saves tokens + latency):

```typescript
const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 4096,
  system: [
    {
      type: "text",
      text: largeSystemPrompt,
      cache_control: { type: "ephemeral" },
    },
  ],
  messages,
});
```

Cache breakpoints: place on content blocks ≥ 1024 tokens for benefit.

## Streaming

```typescript
const stream = client.messages.stream({
  model: "claude-sonnet-4-20250514",
  max_tokens: 4096,
  messages,
});

for await (const event of stream) {
  if (event.type === "content_block_delta") {
    process.stdout.write(event.delta.text ?? "");
  }
}
```

## Error Handling

```typescript
import { APIError, RateLimitError, AuthenticationError } from "@anthropic-ai/sdk";

try {
  const response = await client.messages.create(/* ... */);
} catch (error) {
  if (error instanceof RateLimitError) {
    // Exponential backoff
    await sleep(error.headers?.["retry-after"] ?? 30);
  } else if (error instanceof AuthenticationError) {
    // Invalid API key
  } else if (error instanceof APIError) {
    // Check error.status and error.message
  }
}
```

## OpenClaw-Specific Integration

### Adding a New Pi-Embedded Tool

Create tool in `src/agents/tools/`:

```typescript
import type { AnyAgentTool } from "@mariozechner/pi-coding-agent";

export function createMyTool(): AnyAgentTool {
  return {
    name: "my_tool",
    type: "custom",
    description: "What this tool does",
    parameters: {
      type: "object",
      properties: {
        input: { type: "string", description: "Tool input" },
      },
      required: ["input"],
    },
    execute: async ({ input }) => {
      // Implementation
      return { type: "text", text: JSON.stringify(result) };
    },
  };
}
```

Register in `src/agents/pi-tools.ts` within `createOpenClawCodingTools()`.

### Model Selection in OpenClaw

OpenClaw handles model routing via auth profiles. Use the config system:

```typescript
import type { OpenClawConfig } from "../config/config.js";

// Access model settings from config
const model = config?.models?.primary ?? "claude-sonnet-4-20250514";
```

## Best Practices

- Always set `max_tokens` explicitly
- Use `cache_control` for system prompts > 1024 tokens
- Implement exponential backoff for rate limits
- Stream responses for long outputs
- Validate tool inputs with zod schemas
- Log token usage for cost tracking

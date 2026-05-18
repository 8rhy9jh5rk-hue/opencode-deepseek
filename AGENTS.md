## Project Purpose

This fork of [opencode](https://github.com/anomalyco/opencode) optimizes the project for **DeepSeek API**, with primary focus on **prompt caching** support. Add a `deepseek` provider, implement prompt cache integration per DeepSeek API guide, and ensure existing opencode features work correctly with DeepSeek models.

## How to Test / Lint / Typecheck

Tests cannot run from repo root (guard: `do-not-run-tests-from-root`). Run from package dirs:
```bash
bun test   # in packages/opencode
tsc build && bun turbo typecheck  # type-check if available
```

## Architecture Essentials

This is a Bun monorepo using Effect + Turborepo. Packages live at `packages/*`. Key directories:
- `packages/opencode/src/` — core opencode logic (agents, sessions, providers, tools)

## Provider Integration Flow

### Phase 1: Schema Registration — `src/provider/schema.ts` (~line 10)

Add `deepseek: schema.make("deepseek")` to the ProviderID.withStatics block alongside other well-known providers.

### Phase 2: Provider Loader — `src/provider/provider.ts` (`custom()` function ~lines 150-842)

Add deepseek provider loader entry using OpenAI-compatible SDK, following pattern of alibaba provider:
```ts
import * as OpenAIC from "@ai-sdk/openai-compatible"
// Provider config with openaiCompatible package reference
```

### Phase 3: Message Transform + Cache Control — `src/provider/transform.ts` 🔑 **CRITICAL**

#### 3a. Add deepseek to cache control map (`applyCaching()` ~line 345)

```ts
const providerOptions = {
  // existing options...
  deepseek: { cacheControl: { type: "ephemeral" } },
}
```

#### 3b. Enable caching for DeepSeek models — `message()` guard (~lines 432-444)

Current condition only calls `applyCaching()` for anthropic/google/vertex providers, leaving deepseek without cache support:

```ts
if (model.providerID === "anthropic" || model.id.includes("claude") || 
    /* other existing check */) && !model.api.npm === "@ai-sdk/aws-gateway") {
  msgs = applyCaching(msgs, model)
}
```

**Fix: Add deepseek branch:**
```ts
if (model.providerID === "deepseek" || 
    (model.api.id.toLowerCase().includes("deepseek-chat") && 
     model.api.npm !== "@ai-sdk/aws-gateway")) {
  msgs = applyCaching(msgs, model)
}
```

### Phase 4: Prompt Templates — `src/session/system.ts` (~line 19-32)

In the provider() function, add branch for deepseek-specific prompt routing:
```ts
if (model.api.id.toLowerCase().includes("deepseek")) {
  return [PROMPT_DEEPSEEK]  // or PROMPT_DEFAULT 
}
```

## Agent-Centric Gotchas

### Provider ID Convention
ProviderIDs are string enums registered in schema.ts. Use "deepseek" for DeepSeek providerID registration. When model id contains "deepseek", routing will use deepseaker prompt path.

### Style Conventions (inherited from parent opencode)
- Avoid unnecessary destructuring; prefer dot notation (`obj.a`)
- Prefer `const` over `let`, early returns over `else`
- Inline single-use helpers, extract when reusable or named conceptually

### Effect Patterns
- Do not return `Effect` from helpers doing synchronous work (parsing, validation)
- Prefer Effect schema helpers (`Schema.UnknownFromJsonString`, `Schema.decodeUnknownOption`) over manual JSON.parse with try/catch 

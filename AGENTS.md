## Project Purpose

This fork of [opencode](https://github.com/anomalyco/opencode) optimizes the project for **DeepSeek API**, with primary focus on **prompt caching** support. Add a `deepseek` provider, implement prompt cache integration per DeepSeek API guide, and ensure existing opencode features work correctly with DeepSeek models.

## How to Test / Lint / Typecheck

- Tests cannot run from repo root (guard: `do-not-run-tests-from-root`). Run from package dirs:
  ```bash
  bun test   # in packages/opencode or other package dir
  ```
- Typecheck from specific packages, not root. Use `tsc` per package or turbo scripts.

## Architecture Essentials

This is a [Bun](https://bun.sh) monorepo using [Effect Go](https://.effectful.net/) and Turborepo. Packages live at `packages/*`. Key directories:
- `packages/opencode/src/` — core opencode logic (agents, sessions, providers, tools)

## Provider Integration

To add a new provider to opencode:
1. **Register in schema** (`src/provider/schema.ts`): Add a well-known `ProviderID` entry. For example:
   ```ts
   deepseek: schema.make("deepseek"),
   ```
2. Implement corresponding SDK provider integration and transform layer in `provider/transform.ts`

## Prompt Caching Work

Prompt caching optimization touches these files:
- `src/session/prompt.ts` — constructs messages sent to LLM
- `src/provider/transform.ts` — transforms messages before sending to providers (add DeepSeek cache config here)
- `src/session/system.ts` — provider-specific prompt templates
- `src/session/llm.ts` — handles streaming response with model

Reference **DeepSeek API Guide** for prompt caching parameters and usage patterns.

## Agent-Centric Gotchas

### Style Conventions (inherited from parent opencode)
- Avoid unnecessary destructuring; prefer dot notation (`obj.a`)
- Prefer `const` over `let`, early returns over `else`
- Inline single-use helpers, extract when reusable or named conceptually
- Use snake_case for Drizzle schema fields matching DB columns
- Add comments for non-obvious constraints only

### Effect Patterns
- Do not return `Effect` from helpers doing synchronous work (parsing, validation)
- Prefer Effect schema helpers (`Schema.UnknownFromJsonString`, `Schema.decodeUnknownOption`) over manual `JSON.parse` with try/catch

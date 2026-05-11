# io-mcp-server-capabilties

MCP **prompts** and **resources** consumed by [`celigo/io-mcp-server`](https://github.com/celigo/io-mcp-server) (the Ora MCP server).

This repository is the single source of truth for the markdown bodies and metadata that the MCP server advertises through the protocol's `prompts/list`, `prompts/get`, `resources/list`, and `resources/read` requests. Splitting this content out of the server keeps prompt/resource authoring lightweight: a content change is one PR here, with no Node build cycle in `io-mcp-server`.

## Layout

```
manifest.yaml              # registry of prompts + resources (single source of truth)
prompts/<name>.md          # body for each prompt
resources/<name>.md        # body for each resource
```

`io-mcp-server` clones this repo at build time (sparse checkout, modeled on its existing `scripts/fetch-specs.sh`), copies it into the runtime image, and the prompt/resource handlers load `manifest.yaml` plus the referenced `.md` files at startup.

## `manifest.yaml` schema

```yaml
prompts:
  - name: <unique snake-or-kebab-case identifier>
    description: <1-2 sentence summary shown in prompts/list>
    file: prompts/<name>.md            # path relative to this manifest
    arguments:                          # optional
      - name: <argName>
        description: <what it is>
        required: <true|false>

resources:
  - uri: celigo://resources/<slug>
    name: <human-readable name>
    description: <1-2 sentence summary>
    mimeType: text/markdown             # currently always text/markdown
    file: resources/<name>.md
```

### Templating in prompt bodies

Prompt bodies may contain two kinds of substitutions, evaluated by the server's `renderTemplate` before the prompt is returned:

| Syntax | Behavior |
| --- | --- |
| `{{var}}` | Replaced with the argument value, **only if `var` is in the prompt's `arguments` list**. Unknown names are left untouched (so literal Handlebars examples in `writing-handlebars.md` / `writing-sql.md` are preserved). |
| `{{#if var}}…{{else}}…{{/if}}` | Truthy when the argument is non-empty after trim. Same `arguments`-only guard. |

When in doubt, use **triple braces** (`{{{firstName}}}`) for Handlebars examples — those are deliberately not touched by the substitution regex.

## Adding a new prompt

1. Create `prompts/<your-prompt>.md`.
2. Append a new entry under `prompts:` in `manifest.yaml` with the same `name` and `file:` path. Declare `arguments:` for any `{{var}}` placeholders.
3. Open a PR. Once merged, `io-mcp-server`'s next build will pick it up automatically.

## Adding a new resource

1. Create `resources/<your-resource>.md`.
2. Append a new entry under `resources:` in `manifest.yaml`. Pick a stable `celigo://resources/<slug>` URI — clients pin to it.
3. PR the change.

## Why a separate repo?

- **Faster iteration** — content authors don't need to wait on a TypeScript build, codegen, or linter.
- **Clean review surface** — prompt/resource diffs are not buried under code changes.
- **Reusable** — other MCP servers (or non-server consumers) can read this manifest as-is.

## Related

- Consumer: [celigo/io-mcp-server](https://github.com/celigo/io-mcp-server) (`scripts/fetch-capabilities.sh`)
- Sister pattern: [celigo/integrator-api-specs](https://github.com/celigo/integrator-api-specs) (OpenAPI specs, fetched the same way)

# io-mcp-server-capabilties

MCP **prompts** and **resources** consumed by [`celigo/io-mcp-server`](https://github.com/celigo/io-mcp-server) (the Ora MCP server).

This repository is the single source of truth for the markdown bodies and metadata that the MCP server advertises through the protocol's `prompts/list`, `prompts/get`, `resources/list`, and `resources/read` requests. Each `.md` file is self-describing — metadata lives as YAML frontmatter at the top of the file, so adding or editing a prompt/resource is a single-file change with no separate registry to maintain.

## Layout

```
prompts/<name>.md          # prompt body with YAML frontmatter
resources/<name>.md        # resource body with YAML frontmatter
```

`io-mcp-server` clones this repo at build time (sparse checkout), copies it into the runtime image, and the prompt/resource handlers scan each directory, parse the frontmatter, and build the in-memory registry at startup.

## Frontmatter schema

### Prompts (`prompts/*.md`)

```yaml
---
name: <unique snake-or-kebab-case identifier>
description: <1-2 sentence summary shown in prompts/list>
arguments:                          # optional — omit if no arguments
  - name: <argName>
    description: <what it is>
    required: <true|false>
---
<prompt body in Markdown>
```

### Resources (`resources/*.md`)

```yaml
---
uri: celigo://resources/<slug>
name: <human-readable name>
description: <1-2 sentence summary>
mimeType: text/markdown             # currently always text/markdown
---
<resource body in Markdown>
```

### Templating in prompt bodies

Prompt bodies may contain two kinds of substitutions, evaluated by the server's `renderTemplate` before the prompt is returned:

| Syntax | Behavior |
| --- | --- |
| `{{var}}` | Replaced with the argument value, **only if `var` is in the prompt's `arguments` list**. Unknown names are left untouched (so literal Handlebars examples in `writing-handlebars.md` / `writing-sql.md` are preserved). |
| `{{#if var}}…{{else}}…{{/if}}` | Truthy when the argument is non-empty after trim. Same `arguments`-only guard. |

## Adding a new prompt

1. Create `prompts/<your-prompt>.md` with the frontmatter block at the top (see schema above).
2. Open a PR. Once merged, `io-mcp-server`'s next build will auto-discover it.

## Adding a new resource

1. Create `resources/<your-resource>.md` with the frontmatter block. Pick a stable `celigo://resources/<slug>` URI — clients pin to it.
2. PR the change.

## Why a separate repo?

- **Faster iteration** — content authors don't need to wait on a TypeScript build, codegen, or linter.
- **Clean review surface** — prompt/resource diffs are not buried under code changes.
- **Reusable** — other MCP servers (or non-server consumers) can scan these files directly.

## Related

- Consumer: [celigo/io-mcp-server](https://github.com/celigo/io-mcp-server) (`scripts/fetch-capabilities.sh`)
- Sister pattern: [celigo/integrator-api-specs](https://github.com/celigo/integrator-api-specs) (OpenAPI specs, fetched the same way)

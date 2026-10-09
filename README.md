# opencode2

Home of opencode v2 developments. Monorepo (Bun workspaces).

## Layout

- `agents/` — agent definitions (empty stub)
- `models/` — model presets / configs (empty stub)
- `skills/` — agent skills (empty stub)
- `themes/` — TUI themes (empty stub)
- `commands/` — slash-commands (empty stub)
- `plugins/` — opencode plugins
  - `plugins/todo/` — `opencode2-todo` plugin (restores `todowrite` + TUI sidebar). See `plugins/todo/README.md`.

## Dev

```sh
bun install
bun test ./plugins/todo
npx tsc --noEmit -p plugins/todo/tsconfig.json
# or via root scripts:
bun run test
bun run typecheck
```

Plugin entrypoints:

```json
{ "plugins": ["/absolute/path/to/opencode2/plugins/todo/src/index.ts"] }
```

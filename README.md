# gen-guard

Stops Claude from hand-editing files that a tool generates: lockfiles, build output, vendored or
installed dependencies, protobuf/codegen output, and anything with a `DO NOT EDIT` / `@generated`
header or `linguist-generated` in `.gitattributes`. The deny message names the regenerating command or
the source file: the `//go:generate` directive, the `.proto`, the `build` script, or `pnpm install`
for `pnpm-lock.yaml`.

**Hook:** `PreToolUse` on `Write|Edit|MultiEdit|NotebookEdit`.

**Config:** `.claude/gen-guard.json` (project) or `~/.claude/gen-guard.json` (global):

```json
{"mode": "deny", "allow": ["dist/vendor-patch.js"], "extra_generated": ["src/api/client/**"],
 "check_headers": true}
```

`mode` is `deny` (the default), `ask` (you approve each edit) or `warn` (Claude is told but not stopped).

## Install

From Anthropic's plugin directory, once listed: search for `gen-guard` in `/plugin directory` in Claude Code, or under **Customize > Plugins** on claude.ai.

To try it from a clone for one session: `claude --plugin-dir /path/to/gen-guard`.

This plugin is for Claude Code: it is built on hooks, which claude.ai chat does not load.

## Configuration

Settings are read from `~/.claude/gen-guard.json` and then `<project>/.claude/gen-guard.json`. The project file wins, objects merge by key, and a broken file is skipped and logged.

## What it runs and stores

It is Python 3.9+ standard library only, makes no network requests, and sends nothing anywhere. Every hook fails open: an internal error is logged to `~/.claude/gen-guard/errors.log` and the hook exits 0.

- Reads the target file's header and the project's `.gitattributes`. It runs no commands and writes nothing except the error log.

## Development

```
python3 -m pytest -q
claude plugin validate --strict .
```

`docs/HOOK_SCHEMA.md` records the hook payloads this plugin depends on.

## License

MIT. See `LICENSE`.

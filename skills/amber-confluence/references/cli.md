# Working through the CLI

Use this when the MCP tools are not available but a shell is. The CLI has the same
behaviour and the same refusals.

| Command | Effect |
|---|---|
| `amber confluence pull <page>` | writes the page to `~/amber/pages/<id>.md` |
| `amber confluence diff <page>` | shows what a push would change |
| `amber confluence push <page> -m "why"` | publishes the edited file |
| `amber confluence status [page]` | compares local copies with the server |
| `amber confluence doctor [page]` | checks the setup, writes nothing |

`<page>` is a page id or a Confluence URL containing one. Add `--json` to any
command for output a script can read.

## Workflow

1. `amber confluence pull <page>`. The output names the file and lists the markers.
2. Edit the Markdown file. Leave the header between the `---` lines untouched, and
   leave the hidden sidecar file next to it alone: it holds the opaque blocks.
3. `amber confluence diff <page>`. Show the diff to the user before publishing
   anything larger than a small fix.
4. `amber confluence push <page> -m "what changed"`.

## Exit status

- `0`: done.
- `2`: a refusal. The page changed on the server, or a marker is missing. Read the
  message and handle it as SKILL.md describes.
- Anything else: a failure. Run `amber confluence doctor`.

`--force` overrides a refusal. The same rule applies as for the MCP tools: only
after the user has agreed for this page and this refusal.

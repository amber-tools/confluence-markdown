---
name: amber-confluence
description: Read and edit pages on a self-hosted Confluence (Server or Data Center) as Markdown without breaking macros, panels, layouts or attachments. Use when the user gives a Confluence page id or URL on their own domain (not *.atlassian.net) and asks to read, summarize, fix, update or rewrite the page, or when the confluence_pull, confluence_push, confluence_replace or confluence_status tools are available. Covers setup of the amber-confluence MCP server and CLI, safe editing around opaque block markers, and what to do when a publish is refused. Not for Confluence Cloud.
license: MIT
---

# amber-confluence

Edit self-hosted Confluence pages as Markdown. Everything Markdown cannot express
(macros, panels, status lozenges, attachments, layouts, complex tables) is lifted out
before conversion and restored byte for byte on publish. In the Markdown it appears as
a marker such as `⟦macro.warning#1⟧`.

The tool is open source (MIT): https://github.com/amber-tools/confluence-markdown

## When to use this

- The page lives on Confluence Server or Data Center, on the company's own domain.
- The user wants to read, summarize, correct or rewrite a single existing page.

Do not use it for Confluence Cloud (`*.atlassian.net`). Atlassian's own connector
covers that. It also cannot create pages, pull page trees or upload attachments. Say
so instead of improvising.

## Before the first call

1. Check that the tools `confluence_pull`, `confluence_push`, `confluence_replace`
   and `confluence_status` exist. If they do not, follow
   [references/setup.md](references/setup.md), or fall back to the CLI described in
   [references/cli.md](references/cli.md).
2. Require version 0.2.0 or newer. Version 0.1.0 can lose tables on publish. Check
   with `npm ls -g amber-confluence`; if it is older, upgrade before any push.
3. Never ask the user to paste a token or password into the chat, and never write
   one into a file. Credentials go into the OS keychain; setup.md has the commands
   for the user to run themselves.

## Choosing the operation

| The user wants | Call |
|---|---|
| To read or summarize a page | `confluence_pull` |
| A small change: a sentence, a number, a table cell | `confluence_replace` |
| A rewrite, a new section, a restructure | `confluence_pull`, edit, `confluence_push` |
| To know whether a local copy is still current | `confluence_status` |

`page` is a page id or any Confluence URL that contains one.

Prefer `confluence_replace` for small edits. It pulls, substitutes and publishes in
one step, so the whole page never has to pass through your context. `old_string`
must match the pulled Markdown exactly and occur once; extend it with surrounding
text until it is unique, or pass `replace_all: true` when every occurrence should
change.

For `confluence_push`, send the complete Markdown of the page, not a fragment: it
replaces the body entirely. Always pull in the same session first, and edit the text
that the pull returned.

Pass `message` on every publish. It becomes the version comment in the page history,
so write what changed and why in one line.

## Rules for markers

A marker looks like `⟦macro.toc#1⟧`, `⟦macro.status#2⟧` or `⟦table#1⟧`, sometimes with a suffix
such as `⟦macro.info#1~k3f⟧`.

- Copy markers character for character. Never retype, renumber or "fix" one.
- Never invent a marker. A new macro cannot be created this way; tell the user to
  add it in the Confluence editor.
- Moving a marker moves the block. Deleting a marker deletes the block from the
  page, so delete one only when the user asked for that block to go.
- Leave the marker where it was when you rewrite the text around it. A status
  lozenge inside a table cell stays in that cell.
- A marker may stand for a whole table that Markdown cannot carry (merged cells,
  lists inside cells). Its contents cannot be edited here. Say so.

When you describe the page to the user, call these blocks what they are ("the
warning panel", "the table of contents"), not by marker name.

## When a publish is refused

A refusal comes back as readable tool output. It is an answer, not a crash. Read it
and act on it; do not retry the same call.

**"The page changed after you pulled it."** Someone else edited the page. Pull again,
reapply your edit to the new text, publish. Do not use `force`: it would erase the
other person's work.

**A marker is missing.** The message names each missing block. If you dropped it by
accident, put the marker back where it was and publish again. If the user asked to
remove that block, confirm with them and then repeat with `force: true`.

**"Nothing to publish."** The Markdown equals what is on the server. No version was
created. Tell the user that nothing changed, and check whether your edit was applied.

`force: true` overrides both refusals. Use it only after the user has said, for this
page and this refusal, that they want it. Report afterwards exactly what was
overwritten or removed; the tool output lists it.

## After publishing

Report the title, the version change (for example v12 to v13) and the page URL, all
of which the tool returns. Mention any block the user should check by eye.

## Other errors

- 401 with a credential the user is sure about: instances older than Data Center
  7.9 have no personal access tokens and need the password instead.
- "Confluence returned HTML instead of JSON": a login page, usually single sign-on
  in front of the instance or an expired session.
- TLS errors: the company certificate is not trusted. See `ca_file` in setup.md.
  Certificate checks cannot be switched off, and should not be.

For anything else, ask the user to run `amber confluence doctor <page>`. It checks
the certificate, the version, the credential and a full round trip without writing
anything. `--share` prints a report with no host, account or page content.

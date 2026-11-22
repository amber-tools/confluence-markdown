# Setup

The user runs these steps on their own machine. Walk them through it; do not handle
their credential yourself.

## 1. Install

Node 18 or newer. No other dependencies.

```bash
npm install -g github:amber-tools/confluence-markdown
```

This installs the current release straight from the repository. Check the version:

```bash
npm ls -g amber-confluence
```

It must be 0.2.0 or newer. Do not install from the npm registry until it serves
0.2.0 (`npm view amber-confluence version`): 0.1.0 there can lose tables on publish.

## 2. Point it at the instance

Create `~/.config/amber/config.toml` (on Windows `%USERPROFILE%\.config\amber\config.toml`):

```toml
[confluence]
url  = "https://wiki.example.com"
user = "you@example.com"          # leave out when using a token
```

Optional keys: `ca_file` (path to a company root certificate in PEM), `timeout`
(seconds, default 30), `pages_dir` (where local copies live, default `~/amber/pages`).

A config file that contains a token or a password is rejected on purpose.

## 3. Store the credential

| Confluence | Credential |
|---|---|
| Data Center 7.9 or newer | Personal access token: Profile, Settings, Personal Access Tokens |
| Older | The account password |
| Public wiki | None |

macOS, token:

```bash
security add-generic-password -s amber-confluence-token -a 'you@example.com' -w
```

macOS, password:

```bash
security add-generic-password -s amber-confluence -a 'you@example.com' -w
```

Linux, token:

```bash
secret-tool store --label=amber service amber-confluence-token account you@example.com
```

Linux, password:

```bash
secret-tool store --label=amber service amber-confluence account you@example.com
```

Windows, or for one terminal session anywhere: set `AMBER_CONFLUENCE_TOKEN` or
`AMBER_CONFLUENCE_PASSWORD` in the environment.

## 4. Check it

```bash
amber confluence doctor <page id or URL>
```

Every line should carry a tick. The doctor never writes to Confluence.

## 5. Register the MCP server

Claude Code:

```bash
claude mcp add confluence -- amber-confluence-mcp
```

Any other MCP client (Claude Desktop, Cursor, VS Code and others), in its MCP
configuration file:

```json
{
  "mcpServers": {
    "confluence": {
      "command": "amber-confluence-mcp"
    }
  }
}
```

The server reads the same config file and keychain as the CLI. To configure it
without a config file, add `"env": { "AMBER_CONFLUENCE_URL": "...", "AMBER_CONFLUENCE_USER": "..." }`.
Do not put a token in that block unless the user decides to; the keychain is the
intended place.

Restart the client. Four tools should appear: `confluence_pull`, `confluence_push`,
`confluence_replace`, `confluence_status`.

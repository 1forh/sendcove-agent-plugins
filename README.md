# Sendcove for AI agents

Plugins that connect Claude Code, Cursor and Gemini CLI to [Sendcove](https://www.sendcove.app/features/mcp), the social media scheduler for X, Instagram, TikTok, Threads, Bluesky, YouTube, Facebook Pages and LinkedIn. Each one adds:

- the Sendcove MCP server (`https://www.sendcove.app/api/mcp`): accounts, posts, media, analytics and inbox tools
- the `sendcove` skill (`skills/sendcove/SKILL.md`): how to use those tools well (drafts first, per-platform text, threads, media uploads, safe retries)

There's no code here, only manifests: the server is hosted by Sendcove and you sign in with OAuth. On the consent screen you choose **Drafts only** (the default) or **Allow publishing**.

## Install

**Claude Code**

```
/plugin install sendcove --marketplace 1forh/sendcove-agent-plugins
```

On Claude Code before 2.1.275, add the marketplace first:

```
/plugin marketplace add 1forh/sendcove-agent-plugins
/plugin install sendcove@sendcove
```

Then run `/mcp`, pick `sendcove` and sign in to Sendcove in your browser.

**Cursor**

Install **Sendcove** from the Cursor Marketplace (Customize > Plugins), then sign in when Cursor asks. To try it before it's listed, clone this repo to `~/.cursor/plugins/local/sendcove` and run **Developer: Reload Window**.

**Gemini CLI**

```
gemini extensions install https://github.com/1forh/sendcove-agent-plugins
```

Then run `/mcp auth sendcove` and sign in.

**GitHub Copilot in VS Code**

Add the server to `.vscode/mcp.json` (or your user `mcp.json`):

```json
{
  "servers": {
    "sendcove": { "type": "http", "url": "https://www.sendcove.app/api/mcp" }
  }
}
```

Start the server when VS Code asks and allow it to sign in, then use Sendcove from Copilot agent mode. Guide: https://www.sendcove.app/features/mcp/github-copilot

**Any other MCP app** (ChatGPT, Claude, Codex, Amp, Windsurf, Zed...): add `https://www.sendcove.app/api/mcp` as a remote MCP server. Setup guides for each app: https://www.sendcove.app/features/mcp

## Plans

Works on every paid Sendcove plan. On Basic the connection is drafts only; scheduling, publishing and replying need Pro or Max and choosing **Allow publishing**. Disconnect anytime in Sendcove under Account > API keys.

## Layout

| File | Used by |
| --- | --- |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code |
| `.cursor-plugin/plugin.json`, `mcp.json` | Cursor |
| `gemini-extension.json`, `GEMINI.md` | Gemini CLI |
| `skills/sendcove/SKILL.md`, `assets/logo.png` | All three |

Support: https://www.sendcove.app/contact

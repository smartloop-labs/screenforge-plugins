# ScreenForge for Claude Code and other agents

Record your screen and edit recordings into polished videos from your own agent: cuts, speed,
zooms, backgrounds, captions, narration and export. The plugin connects the MCP server that
ships with the [ScreenForge](https://screenforge.studio) app, and adds a skill that teaches the
agent how to record and edit. Everything runs on your computer, and ScreenForge spends no AI
tokens of its own.

You need ScreenForge 0.1.66 or later, installed on macOS from
[screenforge.studio](https://screenforge.studio) or with `brew install zot24/tap/screenforge`
(macOS and Linux).

## Claude Code

Run these in Claude Code one at a time. First add the marketplace:

```
/plugin marketplace add smartloop-labs/screenforge-plugins
```

Then install the plugin:

```
/plugin install screenforge@screenforge
```

Or the server alone, without the skill:

```sh
claude mcp add screenforge -- screenforge-cli mcp     # Homebrew or .deb
claude mcp add screenforge -- /Applications/ScreenForge.app/Contents/MacOS/sf mcp
```

## Codex and Copilot CLI

```sh
codex plugin marketplace add smartloop-labs/screenforge-plugins
copilot plugin marketplace add smartloop-labs/screenforge-plugins
```

## Cursor, VS Code and others

The ScreenForge app sets these up for you: open Settings → AI agents. Any other MCP client can
start the server as a local stdio command:

```json
{ "mcpServers": { "screenforge": { "command": "/Applications/ScreenForge.app/Contents/MacOS/sf", "args": ["mcp"] } } }
```

## Try

> Record my screen while I demo the signup flow. … Done. Cut the pauses, speed up the typing,
> zoom on the buttons I click, put it on a dark gradient with captions, and export it.

On macOS, recording needs Screen Recording, and for click zooms also Input Monitoring. Grant
them to the app that runs your agent, such as your terminal.

## License

ScreenForge-plugins is source-available under the [Business Source License 1.1](LICENSE) (`BUSL-1.1`).

- You may read, modify, self-host and use it in production, including inside your company.
- You may not offer it to third parties as a hosted or embedded commercial product that competes with Smartloop's paid offerings.
- Each version converts to the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0) four years after it is published.

For a commercial license or questions, contact [hello@smartloop.ai](mailto:hello@smartloop.ai).

Versions released before November 27, 2026 remain available under the MIT License to anyone who received them under it.

# TomYum plugin template

A starter for writing a **TomYum** plugin, shipped as a standalone repo and wired into the main
TomYum server as a git submodule at `templates/plugin-template`.

Two shapes are shown — pick one:

| Shape | Folder | Deploy |
|-------|--------|--------|
| **Compiled** (`.dll`) | [`compiled/`](compiled/) | build → drop `MyPlugin.dll` into the server's `plugins/` |
| **Loose source** (`.cs`) | [`loose-source/`](loose-source/) | drop the whole folder into `plugins/`; the server compiles it at runtime |

A plugin references **only** `TomYum.Plugins.Abstractions` — never the server or infrastructure.

## Quick start (compiled)

1. Copy `compiled/` somewhere and rename `MyPlugin`.
2. Wire the `TomYum.Plugins.Abstractions` reference (see the comments in `MyPlugin.csproj`).
3. `dotnet build -c Release`
4. Copy `bin/Release/net10.0/MyPlugin.dll` into the server's `plugins/` directory and restart.
5. In-game: `!plugins` lists it; `!myplugin` runs its command.

## Quick start (loose source)

1. Copy `loose-source/` into the server's `plugins/` directory (e.g. `plugins/my-loose-plugin/`).
2. Edit `plugin.json` (`id`, `entryPoint`) and `code.cs`.
3. Restart the server — it compiles and loads the folder. Requires `Plugins:AllowLooseSource: true`.

## What a plugin can do

Everything flows through `IPluginContext`:

- `Config` — your settings, bound from `Plugins:<your-id>` in `config/tomyum.yaml`.
- `Logger` — structured logging scoped to your plugin id.
- `Commands` — register `!`commands (routed by BanchoBot).
- `Events` — subscribe to server events (`ActionChanged`, `ScoreSubmitting` (vetoable), …).
- `Http` — register HTTP routes, optionally on your own subdomain (Session-style frontends).
- `GetHostService<T>()` — inspect live state, e.g. `IHostPlayerView` (who's online, what they're doing).
- `Data` — raw DB access, **only** if the operator marks your plugin `trusted`.

## Full authoring guide

See **[`docs/writing-plugins.md`](docs/writing-plugins.md)** — the complete human guide to the plugin
contract, events, config, and deployment. (A real-world example built on this contract is the
`tomyum.session` plugin in the TomYum server repo.)

## This repo ↔ TomYum

This is the standalone home of the template; TomYum consumes it as a submodule. Typical wiring, once
this repo has a remote:

```bash
# inside the TomYum repo, replacing the in-tree copy:
git rm -r templates/plugin-template
git commit -m "chore: move plugin template to its own repo"
git submodule add <this-repo-url> templates/plugin-template
git commit -m "chore(plugins): add plugin template as a submodule"

# cloning TomYum afterwards:
git clone --recurse-submodules <tomyum-url>
# or, in an existing checkout:
git submodule update --init --recursive
```

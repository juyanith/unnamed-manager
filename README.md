# Unnamed Manager

A keyboard-first manager for your desktop: press a hotkey or a leader
sequence to start an app, send a shortcut, or move, resize, focus,
fill, arrange and minimize windows. Actions are modules, so more can come
later (volume, display resolution).

There are two native apps, one per platform, with one config format:

| Repo | Platform | Stack | Built with |
| --- | --- | --- | --- |
| `unnamed-manager-mac` | macOS 26+ | Swift 6, SwiftUI | [BAMA](https://github.com/daveebbelaar/bama) |
| `unnamed-manager-windows` | Windows 11 | C# 14, .NET 10, WinUI 3 | [BAWA](https://github.com/daveebbelaar/bawa) |

The Mac app grew out of
[Unnamed Launcher](https://github.com/juyanith/unnamed-launcher) and keeps
its history.

> [!WARNING]
> **Work in progress.** Both apps and this config format are still being built. Features, fields and rules can change or break between versions. If you choose to use the apps, you do so at your own risk: they come as is, with no warranty of any kind (see the [license](LICENSE)).

## This repo

This repo holds what both apps share. It has no app code.

| File | What it is |
| --- | --- |
| [CONFIG-SPEC.md](CONFIG-SPEC.md) | The config format, what each field means, and every validation rule with its id. |
| [config.schema.json](config.schema.json) | JSON Schema for editors. |
| [fixtures/](fixtures/) | Sample configs. [manifest.json](fixtures/manifest.json) says which must load and which rule each invalid one breaks. |
| [GLOSSARY.md](GLOSSARY.md) | One name per idea, for the UI on both platforms. |
| [CHANGELOG.md](CHANGELOG.md) | Spec versions. |

## How the apps use it

Each app repo has this repo as a git submodule at `spec/`, pinned to a
tag. Each app's unit tests run every fixture in `spec/fixtures/`.

```bash
git clone --recursive <app repo>       # or, in an existing clone:
git submodule update --init
```

To change the config format:

1. Change `CONFIG-SPEC.md`, the schema and the fixtures here, add a line
   to `CHANGELOG.md`, and tag `spec-vN`.
2. In each app repo, move the submodule to the tag:
   `git -C spec fetch --tags && git -C spec checkout spec-vN`, then commit.
   The fixture tests fail until the app supports the change.

## License

MIT. See [LICENSE](LICENSE).

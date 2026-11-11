# Unnamed Manager config spec

**Status:** spec v1, draft. Not tagged yet.

This is the contract the Mac and Windows apps share: the config file's
format, what each field means, and the rules that make a file valid. Each
app implements it in its own language. The fixtures in `fixtures/` are the
test: both apps must accept every valid fixture and reject every invalid
one, for the rule the manifest names.

Spec v1 is the config format of Unnamed Launcher 0.2.0, unchanged and
written down for both platforms. The Mac app passes it today. New
features (window actions, the new file location, Windows app identifiers)
are proposals in [Planned for v2](#planned-for-v2) until they're settled
and moved up into the spec.

## The file

- **Format:** JSON, UTF-8. Keys are case-sensitive. An unknown field is an
  error, so a typo never silently does nothing.
- **Location (v1):** `~/.config/unnamed-launcher/config.json`. See v2 for
  the move to `unnamed-manager`.
- **Test override:** the `APP_STORE` environment variable names a config
  file to use instead. Tests, snapshots and self-tests always set it and
  never touch the real file. (BAMA and BAWA both use `APP_STORE`.)
- **Reload:** the app watches the file and reloads it when it changes. A
  reload with errors keeps the last valid config and reports the errors.
- **Writing:** when the app saves, it writes keys in a stable order, so
  diffs stay small and a dotfiles tool can manage the file.
- **Schema:** [config.schema.json](config.schema.json) gives editors
  validation and autocomplete. A config points to it with
  `"$schema": "./config.schema.json"`. The schema can't express every rule,
  so passing it doesn't make a config valid. [Validation](#validation) does.

```json
{
  "$schema": "./config.schema.json",
  "version": 1,
  "runtime": { "cancelKey": "Escape", "backKey": "Backspace", "showHud": true, "hudDelayMs": 500, "timeoutMs": 5000 },
  "ui": { "theme": "system", "position": "center", "errorFlash": true, "errorBell": true },
  "entries": [
    { "keys": "Hyper+Space", "title": "Commands" },
    { "keys": "Hyper+Space E", "title": "Editor", "action": "open-editor" },
    { "keys": "Hyper+Space F", "title": "Find", "action": "search" },
    { "keys": "Hyper+T", "title": "Terminal", "action": "open-terminal" }
  ],
  "actions": {
    "open-editor":   { "type": "application", "bundleId": "dev.zed.Zed" },
    "open-terminal": { "type": "application", "bundleId": "com.mitchellh.ghostty" },
    "search": {
      "type": "contextual",
      "variants": { "browser": { "type": "send", "keys": "Primary+L" } },
      "fallback": { "type": "send", "keys": "Primary+F" }
    }
  },
  "contexts": [
    { "id": "browser", "apps": ["com.google.Chrome", "org.mozilla.firefox"] }
  ]
}
```

## Top-level fields

| Field | Type | Required | Default | Meaning |
| --- | --- | --- | --- | --- |
| `$schema` | string | no | none | Path to the schema, for editors. Kept as written. |
| `version` | integer | yes | | Format version. v1 accepts only `1`. |
| `runtime` | object | no | see below | Sequence keys and timing. |
| `ui` | object | no | see below | How the Guide looks. |
| `entries` | array | yes | | Menus and bindings, in order. At least one. |
| `actions` | object | no | `{}` | Action id to action. |
| `contexts` | array | no | `[]` | Named sets of apps for Context Aware actions. |

### `runtime`

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `cancelKey` | key name | `Escape` | Ends a sequence and runs nothing. |
| `backKey` | key name or `""` | `Backspace` | Goes up one menu. `""` turns Back off. |
| `showHud` | boolean | `true` | Whether the Guide appears at all. |
| `hudDelayMs` | integer ≥ 0 | `0` | Idle time in a sequence before the Guide appears. Each key press restarts it. |
| `timeoutMs` | integer ≥ 0 | `0` (off) | Idle time with the Guide up before the sequence ends. |

### `ui`

| Field | Values | Default | Meaning |
| --- | --- | --- | --- |
| `theme` | `system`, `light`, `dark` | `system` | The Guide's appearance. Case-insensitive. |
| `position` | `center`, `top`, `bottom` | `center` | Where the Guide sits on the display of the app in front. Case-insensitive. |
| `errorFlash` | boolean | `true` | Flash the Guide on a key that isn't bound. |
| `errorBell` | boolean | `true` | Play the alert sound on a key that isn't bound. |

## Keys

### Syntax

An entry's `keys` is a **key path**: a chord, then zero or more step keys,
separated by spaces. `Hyper+Space A T` is the chord `Hyper+Space`, then
`A`, then `T`.

A chord is modifiers and one key joined by `+`: `Ctrl+Alt+L`. Names are
case-insensitive. On the Mac, the symbols `⌃⌥⇧⌘` also work, joined with
`+` or written straight before the key (`⌃⌥⇧T`, `⌘+F`, `Opt+⌘K`). Apps
always write names, never symbols.

### Modifiers

| Name in the file | Mac | Windows |
| --- | --- | --- |
| `Ctrl`, `Control` | ⌃ Control | Ctrl |
| `Alt`, `Opt`, `Option` | ⌥ Option | Alt |
| `Shift` | ⇧ Shift | Shift |
| `Cmd`, `Command` | ⌘ Command | Win (the key in the same place) |
| `Primary` | ⌘ Command | Ctrl |
| `Hyper` | ⌃⌥⇧⌘ | Ctrl+Alt+Shift+Win |
| `Meh` | ⌃⌥⇧ | Ctrl+Alt+Shift |
| `CAG` | ⌃⌥⌘ | Ctrl+Alt+Win |

- `Primary` is the platform's main shortcut modifier. Use it in Send
  Shortcut so one config works on both: `Primary+L` focuses a browser's
  address bar on either platform.
- Groups can combine with other modifiers: `Meh+Cmd` is `Hyper`.
- When an app writes a chord, a set of modifiers that exactly equals a
  group is written as the group's name: `Ctrl+Alt+Shift+T` becomes `Meh+T`.
- The Windows column is the v1 reading for the Windows app. `Win` and
  `Super` as names are a v2 proposal.

### Key names

`A`–`Z`, `0`–`9`, `F1`–`F24`, `Space`, `Tab`, `Enter`, `Backspace`,
`Delete`, `Escape`, `Left`, `Right`, `Up`, `Down`, `Home`, `End`,
`PageUp`, `PageDown`, `Grave`, `Minus`, `Equal`, `LeftBracket`,
`RightBracket`, `Backslash`, `Semicolon`, `Quote`, `Comma`, `Period`,
`Slash`.

- **Names mean physical positions on the US ANSI layout.** A different
  keyboard layout doesn't change which key a name means. On Windows this
  means matching on scan codes, not virtual keys.
- **A key the platform's keyboard doesn't have** (F21–F24 on a Mac) is a
  warning, not an error, so one config can serve both platforms.

### Reserved chords

The OS takes some chords before any app can see them. A leader or direct
binding on one is an error **on that platform only**:

| Platform | Reserved |
| --- | --- |
| Mac | `Hyper+Comma`, `Hyper+Period`, `Hyper+Slash` (sysdiagnose) |
| Windows | Not decided yet. `Cmd+L` (Win+L, lock) is reserved for certain; see v2 open questions. |

## Entries

Every entry is a **menu** (no `action`) or a **binding** (an `action`).

| Field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `keys` | key path | yes | Where it sits in the tree. |
| `title` | string, not blank | yes | What the Guide and the Bindings window show. |
| `action` | action id | no | Makes it a binding. |
| `icon` | string | no | Its icon. Mac: an SF Symbol name, or a path starting with `/` or `~` for that file's icon. Windows: v2 open question. `""` is the same as leaving it out. |

- **Entries are a flat, ordered list keyed by their full path.** The tree
  is built from the paths, and children keep file order.
- **A top-level entry is a leader** when it's a menu, and a **direct
  binding** when it has an action. A config can mix both, or have only
  direct bindings.
- **Only the first chord has modifiers.** Step keys are bare keys.

## Behavior both apps share

The full rules are in the Mac app's `docs/SPEC.md`. These are the ones a
user would notice if the two apps differed:

- **Each key is decided as soon as it's pressed.** No timers decide which
  entry a key picks, so there are no ambiguous prefixes.
- **In a sequence, every key is consumed** until the sequence ends,
  including repeats and key-ups. Outside one, only leader and direct
  binding chords are consumed.
- **Held leader modifiers are ignored on step keys**, and never leak into
  Send Shortcut.
- **A child menu's key** opens it. **A binding's key** runs its action and
  ends the sequence. **Back** goes up, **Cancel** ends. **Any other key**
  is an error flash or sound, and the sequence stays where it is.
- **Home** is the menu above the leaders. Back from a leader's menu opens
  it when the config has two or more top-level entries.
- **A direct binding** runs at once with no Guide. If its Context Aware
  action has nothing for the app in front, the chord passes through to
  that app untouched.
- **The Guide never takes focus.** The app that was in front when the
  leader was pressed stays in front, and actions go to it.

## Actions

`actions` maps an id to an action. Every action has a `type`; the other
fields depend on it. Unknown fields are errors.

| `type` | UI name | Fields | Behavior |
| --- | --- | --- | --- |
| `application` | Use Application | `bundleId` (required), `mode` | `focusOrStart` (default): bring the app forward, starting it if needed. `focus`: only if it's running. `start`: start a new copy even if one is running. Focusing returns to the app's last focused window. |
| `open` | Open | `target` (required) | Opens a file, folder or URL with its default app. A leading `~` is the home folder. |
| `exec` | Run Command | `command` (required, array, at least one item), `cwd`, `env` | Starts a process directly: no shell, no expansion except a leading `~`. `cwd` defaults to the home folder. Doesn't wait, but reports a failure to start. |
| `send` | Send Shortcut | `keys` (required, key path) | Presses one or more chords in the app that was in front. |
| `contextual` | Context Aware | `variants` (context id to action), `fallback` | Runs the variant for the first context, in `contexts` order, that matches the app in front. Otherwise runs `fallback`. With no fallback, the action is unavailable for that app. Variants and the fallback can't be `contextual`. |
| `noop` | Do Nothing | none | Does nothing. |
| `help` | Show Help | none | Opens the Help page of the Bindings window. |

- `bundleId` is a macOS identifier. How the Windows app names apps is
  the first v2 open question.
- When an action fails (app not installed, command can't start), the
  sequence ends and the failure shows in the Guide and the log.

## Contexts

| Field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `id` | string | yes | Name used as a key in a Context Aware action's `variants`. Unique. |
| `apps` | array of strings | yes | App identifiers. v1: macOS bundle ids. |

## Validation

Each rule has an id. Apps should attach the id to each issue they report,
so the fixtures can check that the right rule fired. Messages are each
app's own, in plain sentences.

A config that breaks any **error** rule doesn't load. **Warnings** never
stop a config from loading.

### Errors

| Rule id | The config is rejected when |
| --- | --- |
| `invalid-json` | The file isn't valid JSON. |
| `version` | `version` isn't `1`. |
| `unknown-field` | An object has a field the format doesn't have. |
| `missing-field` | A required field is missing (`version`, `entries`, an entry's `keys` or `title`, a context's `id` or `apps`, an action's `type` or a field its type requires). |
| `wrong-type` | A field has the wrong kind of value, such as `"version": "1"`. |
| `no-entries` | `entries` is empty. |
| `empty-title` | An entry's title is blank. |
| `bad-keys` | A key path can't be parsed: unknown key or modifier name, or bad syntax. |
| `duplicate-path` | Two entries have the same path, compared case-insensitively. |
| `no-parent` | An entry's parent path has no entry. |
| `parent-is-binding` | An entry's parent is a binding, not a menu. |
| `empty-menu` | A menu (a leader included) has no children. |
| `step-modifiers` | A step key has modifiers. |
| `step-is-runtime-key` | A step key is the cancel or back key. |
| `bad-runtime-key` | `cancelKey` or `backKey` isn't a key name. |
| `same-cancel-back` | The cancel and back keys are the same. |
| `negative-delay` | `hudDelayMs` or `timeoutMs` is negative. |
| `bad-ui-value` | `theme` or `position` isn't one of its values. |
| `reserved-chord` | A leader or direct binding uses a chord this platform reserves. |
| `undefined-action` | A binding names an action that isn't in `actions`. |
| `unknown-action-type` | An action's `type` isn't one of the types above. |
| `bad-app-mode` | An `application` action's `mode` isn't one of its values. |
| `undefined-context` | A Context Aware variant names a context that isn't in `contexts`. |
| `nested-contextual` | A Context Aware variant or fallback is itself `contextual`. |
| `duplicate-context` | Two contexts share an `id`. |

### Warnings

| Rule id | Reported when |
| --- | --- |
| `unused-action` | No binding uses an action. |
| `unused-context` | No action uses a context. |
| `key-not-on-platform` | An entry uses a key this platform's keyboards don't have. |

## Fixtures

`fixtures/manifest.json` lists every fixture and what an app must do with
it:

```json
{
  "file": "invalid/duplicate-path.json",
  "valid": false,
  "errors": [{ "rule": "duplicate-path", "subject": "Hyper+Space A" }],
  "platforms": ["mac", "windows"]
}
```

- `valid`: whether the config must load.
- `errors` (invalid fixtures): the rule that must be reported, and its
  `subject` (the entry path, action id or context id) when it has one.
  Each invalid fixture breaks exactly one rule, because a file that fails
  to decode stops at its first problem.
- `warnings` (valid fixtures): the warnings that must be reported. A
  fixture with no `warnings` must load with none.
- `platforms`: which apps the fixture applies to. Leave it out for both.
- Subjects are written as the app writes key paths (`Meh+T`, not
  `ctrl+alt+shift+t`).

Each app's unit tests read the manifest from the `spec/` submodule and run
every fixture for its platform. An app that doesn't report rule ids yet
checks valid against invalid, and subjects where it can.

## Planned for v2

Proposals, not rules yet. Each one moves up into the spec, with fixtures,
when it's decided.

### Window actions

The main new feature. Proposal: new action families use a dotted `type`,
one family per module, so a new kind of action is a new module and a new
row here, never a change to existing ones. The v1 types keep their names.

| `type` | Fields (proposal) | Does |
| --- | --- | --- |
| `window.focus` | `direction`: `left`, `right`, `up`, `down`, `next`, `previous` | Focuses the nearest window in that direction, or cycles windows. |
| `window.maximize` | `toggle` (default `true`) | Fills the screen's usable area (not full screen). Again restores. |
| `window.minimize` | none | Minimizes the focused window. |
| `window.move` | `to`: `{ "x", "y" }` as fractions of the usable area, or `by`: `{ "dx", "dy" }` in points | Moves the focused window. |
| `window.resize` | `to`: `{ "width", "height" }` as fractions, or `by`: `{ "dw", "dh" }` in points, `anchor` | Resizes the focused window. |
| `window.place` | `frame`: `{ "x", "y", "width", "height" }` as fractions | Moves and resizes in one step (halves, thirds, corners). |

Later families follow the same shape: `audio.volume`, `display.resolution`.

- **Fractions of the usable area** (without the menu bar, Dock or
  taskbar) make one config work on any screen and on both platforms.
  Fixed sizes use points on the Mac and effective pixels on Windows, which
  match in practice.
- **The target is the focused window of the app in front**, the same app
  the Guide leaves in front.

### Other proposals

- **New location:** `~/.config/unnamed-manager/config.json` on both
  platforms (`%USERPROFILE%\.config\unnamed-manager\config.json` on
  Windows), so one dotfiles repo can carry one config for both. On first
  run, the Mac app offers to copy the Unnamed Launcher config.
- **`Win` and `Super`** as modifier names, the same as `Cmd` on both
  platforms.
- **Actions a platform doesn't support** load with a warning
  (`unsupported-action`) and show as unavailable, rather than rejecting a
  config shared between platforms.

### Open questions

1. **How does a config name an app on both platforms?** Options:
   (a) per-platform fields on one action,
   `{ "type": "application", "bundleId": "dev.zed.Zed", "windows": { "exe": "Zed.exe" } }`;
   (b) separate actions picked by platform, like a Context Aware action
   whose variants are `mac` and `windows`;
   (c) one config file per platform. Contexts' `apps` lists have the same
   question.
2. **Windows app identity:** exe name, full path, or AppUserModelID
   (Store and packaged apps have no plain exe path).
3. **Windows reserved chords.** `Win+L` and `Ctrl+Alt+Delete` can't be
   intercepted. On Windows, `Ctrl+Alt+Shift+Win` (Hyper) alone opens
   Office, and Hyper+W, T, X, D, O, P, L, N, Y and others open Office apps.
   Should Hyper+those letters be reserved, or a warning that points to the
   registry fix?
4. **Windows icons:** what `icon` names when SF Symbols don't exist
   (Segoe Fluent Icons code points, a file path, or both).
5. **One `version` or two:** should v2 configs say `"version": 2`, or stay
   at 1 with new optional fields that old apps reject as unknown?

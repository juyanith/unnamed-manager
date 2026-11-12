# Unnamed Manager config spec

**Status:** spec v2, tagged `spec-v2`. [CHANGELOG.md](CHANGELOG.md) lists what changed since `spec-v1`.

This is the contract the Mac and Windows apps share: the config file's
format, what each field means, and the rules that make a file valid. Each
app implements it in its own language. The fixtures in `fixtures/` are the
test: both apps must accept every valid fixture and reject every invalid
one, for the rule the manifest names.

Spec v1 was the config format of Unnamed Launcher 0.2.0, written down for
both platforms. Spec v2 adds window actions, the new file location, and
what happens to an action a platform can't do. What's still open, mostly
for the Windows app, is in [Planned](#planned) until it's settled and moved
up into the spec.

## The file

- **Format:** JSON, UTF-8. Keys are case-sensitive. An unknown field is an
  error, so a typo never silently does nothing.
- **Location:** `~/.config/unnamed-manager/config.json` on the Mac, and
  `%USERPROFILE%\.config\unnamed-manager\config.json` on Windows. On
  first run, the Mac app offers to copy an Unnamed Launcher config from
  `~/.config/unnamed-launcher/config.json`.
- **One file per platform.** Both apps share the format and the rules,
  not the file: actions and contexts name apps each platform's own way.
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
| `version` | integer | yes | | Format version. Only `1`, in spec v1 and v2: see [Versions](#versions). |
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
  `Super` as names are a proposal (see [Planned](#planned)).

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
| Windows | Not decided yet. `Cmd+L` (Win+L, lock) is reserved for certain; see [Planned](#planned). |

## Entries

Every entry is a **menu** (no `action`) or a **binding** (an `action`).

| Field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `keys` | key path | yes | Where it sits in the tree. |
| `title` | string, not blank | yes | What the Guide and the Bindings window show. |
| `action` | action id | no | Makes it a binding. |
| `icon` | string | no | Its icon. Mac: an SF Symbol name, or a path starting with `/` or `~` for that file's icon. Windows: an open question (see [Planned](#planned)). `""` is the same as leaving it out. |

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
  still open (see [Planned](#planned)).
- When an action fails (app not installed, command can't start), the
  sequence ends and the failure shows in the Guide and the log.

### Window actions

Spec v2. Every window action works on the **focused window of the app in
front**, the same app the Guide leaves in front. With no focused window,
the action fails, and the failure shows as for any other action.

| `type` | UI name | Fields | Does |
| --- | --- | --- | --- |
| `window.maximize` | Maximize | `toggle` (default `true`) | Fills the usable area. Not full screen. With `toggle`, a window that already fills it goes back to the frame it had before. |
| `window.minimize` | Minimize | none | Minimizes the window, as the platform's own shortcut does (⌘M, Win+Down). Focus moves the same way. |
| `window.place` | Place Window | `preset` or `frame` | Moves and resizes the window to a part of the usable area. |
| `window.move` | Move Window | `to` or `by` | Moves the window. Its size stays the same. |
| `window.resize` | Resize Window | `to`, `by` or `step`; `anchor`; `widths` | Resizes the window. |
| `window.focus` | Focus Window | `direction` (required) | Focuses the nearest window in a direction, or the next or previous one. |

Window families use a dotted `type`, one family per module. Later
families follow the same shape, such as `audio.volume` and
`display.resolution`.

**Geometry both apps share:**

- **Coordinates** start at the top left of the main display. Positive `x`
  is right and positive `y` is down. Sizes are points on the Mac and
  effective pixels on Windows, which match in practice.
- **The usable area** is the screen without the menu bar, Dock or taskbar
  (`visibleFrame` on the Mac, the work area on Windows). An action uses the
  screen that holds the largest part of the window, or the main screen
  when the window is on none.
- **A fraction** is a part of the usable area's width (for `x` and
  `width`) or height (for `y` and `height`), from 0 to 1. Fractions make
  one config work on any screen.
- **Edges round, sizes follow.** An app turns a frame's edges, not its
  size, into whole points: left is `round(x × W)` and right is
  `round((x + width) × W)`. So the left half and the right half share an
  edge, with no gap and no overlap.
- **A window that won't take a size** (a minimum, a maximum, a fixed
  size) ends as close as it allows, and the action doesn't fail. It keeps
  the edge that touches the usable area's edge, or its center for a
  centered frame.
- **A window never ends off screen.** After a move or a resize, at least
  40 points of its title bar stay on the screen it's on.

#### `window.maximize`

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `toggle` | boolean | `true` | When the window fills the usable area already (within 2 points on every edge), put it back where it was before Maximize. |

The frame before Maximize is kept per window, in memory only. A window
moved or resized since then forgets it, and Maximize leaves it as it is.

#### `window.place`

Exactly one of:

| Field | Type | Meaning |
| --- | --- | --- |
| `preset` | string | One of the presets below. |
| `frame` | object `{ "x", "y", "width", "height" }`, every field required | Fractions. `x` and `y` from 0 to 1, `width` and `height` above 0 up to 1, and `x + width` and `y + height` at most 1. |

| `preset` | `x` | `y` | `width` | `height` |
| --- | --- | --- | --- | --- |
| `leftHalf` | 0 | 0 | 1/2 | 1 |
| `rightHalf` | 1/2 | 0 | 1/2 | 1 |
| `topHalf` | 0 | 0 | 1 | 1/2 |
| `bottomHalf` | 0 | 1/2 | 1 | 1/2 |
| `leftThird` | 0 | 0 | 1/3 | 1 |
| `centerThird` | 1/3 | 0 | 1/3 | 1 |
| `rightThird` | 2/3 | 0 | 1/3 | 1 |
| `leftTwoThirds` | 0 | 0 | 2/3 | 1 |
| `rightTwoThirds` | 1/3 | 0 | 2/3 | 1 |
| `topLeft` | 0 | 0 | 1/2 | 1/2 |
| `topRight` | 1/2 | 0 | 1/2 | 1/2 |
| `bottomLeft` | 0 | 1/2 | 1/2 | 1/2 |
| `bottomRight` | 1/2 | 1/2 | 1/2 | 1/2 |
| `center` | 1/6 | 1/6 | 2/3 | 2/3 |

Presets are exact fractions, so thirds line up. A `frame` that adds up to
1 within 0.001 counts as 1.

#### `window.move`

Exactly one of:

| Field | Type | Meaning |
| --- | --- | --- |
| `to` | object `{ "x", "y" }`, both required | Where the window goes, as fractions of the **free space**: the usable area's width minus the window's width, and the same for height. `{ "x": 0, "y": 0 }` is the top left corner, `{ "x": 0.5, "y": 0.5 }` is centered, and `{ "x": 1, "y": 1 }` is the bottom right corner. A window wider than the usable area has no free space and goes to its left edge. |
| `by` | object `{ "dx", "dy" }`, both default 0 | Points to move by. |

#### `window.resize`

Exactly one of `to`, `by` and `step`:

| Field | Type | Meaning |
| --- | --- | --- |
| `to` | object `{ "width", "height" }`, both required | The new size, as fractions above 0 up to 1. |
| `by` | object `{ "dw", "dh" }`, both default 0 | Points to grow by. Negative shrinks. |
| `step` | `grow` or `shrink` | Steps the width through `widths`. The height stays the same. |

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `anchor` | `topLeft`, `top`, `topRight`, `left`, `center`, `right`, `bottomLeft`, `bottom`, `bottomRight` | `topLeft` | The point of the window that stays where it is, with `to` and `by`. |
| `widths` | array of fractions, at least one, above 0 up to 1 | `[1/4, 1/3, 1/2, 2/3, 3/4]` | The widths `step` goes through. Order and duplicates don't matter. |

- `to` and `by` never make the window bigger than the usable area.
- `grow` picks the smallest width more than 2 points wider than the window
  now, and `shrink` the largest one more than 2 points narrower. Past the
  last one, the window stays as it is; it doesn't wrap. The window keeps
  its left edge, unless the new width would push it past the usable
  area's right edge, when it keeps its right edge.
- `anchor` goes only with `to` and `by`, and `widths` only with `step`.

#### `window.focus`

| Field | Type | Meaning |
| --- | --- | --- |
| `direction` | `left`, `right`, `up`, `down`, `next`, `previous` | Which window to focus. |

- **Candidates** are the standard, visible windows that aren't minimized,
  on the current Space or desktop of every display. Panels, the Guide and
  the app's own windows are skipped.
- **A direction** picks the window whose center is nearest the focused
  window's center, among those within 45° of that direction. A tie goes to
  the one that overlaps more on the other axis. With none, the action fails
  ("No window to the left.").
- **`next` and `previous`** go through the candidates in a fixed order:
  by display, left to right, then by window center, top to bottom and then
  left to right. They wrap at the ends. The same windows always give the
  same order, whatever was focused last.

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
| `missing-field` | A required field is missing (`version`, `entries`, an entry's `keys` or `title`, a context's `id` or `apps`, an action's `type` or a field its type requires, or both fields of a pair where a window action needs one). |
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
| `conflicting-fields` | A window action has fields that don't go together: both `preset` and `frame`; two of `to`, `by` and `step`; `anchor` with `step`; or `widths` without `step`. (Neither of a required pair is `missing-field`.) |
| `bad-fraction` | A fraction is out of its range, a `frame` runs past the usable area, or `widths` is empty. |
| `bad-window-value` | A window action's `preset`, `direction`, `step` or `anchor` isn't one of its values. |
| `unsupported-action` | An action's `type` is in the spec, but this platform's app can't do it yet. |

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

## Versions

`version` stays `1` in spec v2. Spec v2 only adds: new action types and
new rules. An app that doesn't know a type rejects the config with
`unknown-action-type`, which says what's wrong. `version` changes only
when an existing field changes meaning, so an old app never reads a new
file the wrong way.

A spec version is a tag in this repo, `spec-v<N>`. Each app pins its
`spec/` submodule to one.

## Planned

Proposals, not rules yet. Each one moves up into the spec, with fixtures,
when it's decided.

- **`Win` and `Super`** as modifier names, the same as `Cmd` on both
  platforms.

### Decided in spec v2

1. ~~How does a config name an app on both platforms?~~ One config file
   per platform (see [The file](#the-file)).
2. ~~One `version` or two?~~ One. See [Versions](#versions).
3. ~~What does an app do with a type it can't do?~~ It's an error on that
   platform, `unsupported-action`. With one config per platform, a file
   never needs the other platform's actions.

### Open questions

1. **Windows app identity:** exe name, full path, or AppUserModelID
   (Store and packaged apps have no plain exe path).
2. **Windows reserved chords.** `Win+L` and `Ctrl+Alt+Delete` can't be
   intercepted. On Windows, `Ctrl+Alt+Shift+Win` (Hyper) alone opens
   Office, and Hyper+W, T, X, D, O, P, L, N, Y and others open Office apps.
   Should Hyper+those letters be reserved, or a warning that points to the
   registry fix?
3. **Windows icons:** what `icon` names when SF Symbols don't exist
   (Segoe Fluent Icons code points, a file path, or both).

# Unnamed Manager config spec

**Status:** spec v23, tagged `spec-v23`. [CHANGELOG.md](CHANGELOG.md) lists what changed in each version.

This is the contract the Mac and Windows apps share: the config file's
format, what each field means, and the rules that make a file valid. Each
app implements it in its own language. The fixtures in `fixtures/` are the
test: both apps must accept every valid fixture and reject every invalid
one, for the rule the manifest names.

Spec v1 was the config format of Unnamed Launcher 0.2.0, written down for
both platforms. Spec v2 adds window actions, the new file location, and
what happens to an action a platform can't do. Spec v3 adds the gap that
Place Window leaves around windows. Spec v4 settles the Windows app: how it names apps, which chords Windows reserves, and what `icon` names. Spec v5 lets Focus Window start with no focused window. Spec v6 adds its `first` and `last`, and orders windows left to right. Spec v7 renames Maximize to Fill (`window.fill`), which now leaves the gap; `window.maximize` still loads. Spec v8 adds Arrange Windows (`window.arrange`) and the two settings that limit its grid. Spec v9 lets its focused window stay where it is and take the spare cells there. Spec v10 adds Swap Window (`window.swap`), a `scope` for Focus Window, and the alert sound when either finds no window. Spec v11 adds built-in actions, which a binding names without defining them, and renames Focus Window's `direction` to `target`, which also takes a number. Spec v12 makes Focus Window and Swap Window skip windows you can't see, find directions by edges, and order windows in rows, with three settings for the distances. Spec v13 adds the `recent` target, the window used before, from a history of focused windows the app keeps. Spec v15 adds `windows.apps`, rules for each app that leave its windows out of Focus Window, Swap Window or Arrange Windows, in place of `windows.exclude` from spec v13 and v14, which no released app read. Spec v16 adds Quit on close, for the Mac only: `windows.quitOnClose`, its wait, and a `quit` rule in `windows.apps`. Spec v17 adds Switch Space (`space.switch`), which goes to the Space or virtual desktop to the left or right, as quickly as the platform allows. Spec v18 adds held menus (`hold`), which work while the leader's modifiers are held and choose when you let go, as the platforms' app switchers do, and lists (`list`), menus the app fills when they open, starting with Recent Windows. Spec v19 lets a list swap with the chosen window instead of focusing it (`choose`). Spec v20 adds Send to Space (`space.send`), which puts the focused window on another Space or virtual desktop, and can go with it (`follow`). Spec v21 makes dialogs candidates for Focus Window and Recent Windows, and never for Swap Window or Arrange Windows. Spec v22 lets Switch Space go to a Space or desktop by number. Spec v23 makes going with a sent window its own action, Follow to Space (`space.follow`), in place of Send to Space's `follow`. What's still open
is in [Planned](#planned) until it's settled and moved
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
| `version` | integer | yes | | Format version. Only `1`, in every spec so far: see [Versions](#versions). |
| `runtime` | object | no | see below | Sequence keys and timing. |
| `ui` | object | no | see below | How the Guide looks. |
| `windows` | object | no | see below | Settings the window actions share (spec v3). |
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
| `errorBell` | boolean | `true` | Play the alert sound on a key that isn't bound, and when Focus Window or Swap Window finds no window (spec v10). |

### `windows`

Spec v3.

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `gap` | number ≥ 0 | `8` | Points that [Place Window](#windowplace) and [Fill](#windowfill) leave between a window and the usable area's edges, and between windows placed side by side. `0` places windows edge to edge. |
| `maxColumns` | integer from 1 to 8 | `2` | The most columns [Arrange Windows](#windowarrange) picks when its action doesn't give `groups` (spec v8). |
| `maxRows` | integer from 1 to 8 | `2` | The most rows it picks. With `"count": "all"`, it arranges at most `maxColumns` times `maxRows` windows. |
| `minVisible` | number ≥ 0 | `20` | Spec v12. Points on a side of the square a window's visible part must hold for [Focus Window](#windowfocus) and [Swap Window](#windowswap) to count it, and points it must share with the focused window to be in line. `0` counts any window with some part showing. |
| `rowTolerance` | number ≥ 0 | `20` | Spec v12. Windows whose tops are within this many points share a row in Focus Window's order. |
| `tieTolerance` | number ≥ 0 | `2` | Spec v12. Gaps within this many points count as the same when Focus Window picks a window in a direction. |
| `quitOnClose` | boolean | `false` | Spec v16, Mac only. [Quit on close](#quit-on-close): closing an app's last window also quits the app. |
| `quitOnCloseDelayMs` | integer ≥ 0 | `500` | Spec v16, Mac only. How long after a window closes Quit on close counts the app's windows, so an app that opens a window in place of the one it closed isn't quit. |
| `apps` | object | `{}` | Spec v15. Rules for each app's windows, keyed by app id: a bundle id on the Mac, an `exe` or an `appId` on Windows, as in a context's [`apps`](#contexts). Each value is an object of the rules below. An app that isn't listed takes part in every window action. |

Each app in `windows.apps` takes these rules, all optional:

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `focus` | boolean | `true` | `false`: [Focus Window](#windowfocus) never picks the app's windows, for any `target`. |
| `swap` | boolean | `true` | `false`: [Swap Window](#windowswap) never moves the app's windows, as the window swapped with or as the focused one. |
| `arrange` | boolean | `true` | `false`: [Arrange Windows](#windowarrange) never counts or moves the app's windows. |
| `quit` | boolean | `true` | Spec v16, Mac only. `false`: [Quit on close](#quit-on-close) never quits the app. |

- The rules are separate: an app left out of one still takes part in
  the others.
- A window that an action leaves out still covers the windows behind it.
- A rule that isn't one of these is `unknown-field`, and a rule that
  isn't `true` or `false` is `wrong-type`.
- When an app saves the file, it writes every rule its platform has for
  each app listed (four on the Mac, three on Windows), so an app with
  every rule `true` stays listed.

```json
"windows": {
  "apps": {
    "com.apple.calculator": { "focus": true, "swap": true, "arrange": false },
    "com.apple.finder": { "focus": false, "swap": false, "arrange": true }
  }
}
```

#### Quit on close

Spec v16, Mac only. With `windows.quitOnClose` on, the Mac app quits an
app when its last window closes, as Windows apps do. It never quits:

- Finder, or the Mac app itself, whatever `windows.apps` says.
- An app with `"quit": false` in `windows.apps`.
- An app that hasn't shown a window since the Mac app started watching it.
- A menu bar app or a background app (`LSUIElement` or `LSBackgroundOnly`
  in its `Info.plist`).
- An app with a window left: a minimized one, one on another Space, or one
  of a hidden app.

An app inside an app that has `"quit": false`, such as a helper app in
its bundle, is kept too. The app's windows are counted
`quitOnCloseDelayMs` after a window closes. On Windows,
`quitOnClose`, `quitOnCloseDelayMs` and the `quit` rule are
`other-platform-field`.

```json
"windows": {
  "quitOnClose": true,
  "quitOnCloseDelayMs": 800,
  "apps": {
    "com.apple.Preview": { "focus": true, "swap": true, "arrange": true, "quit": false }
  }
}
```

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
- The Windows column is how the Windows app reads each name. `Win` and
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
| Windows | `Cmd+L` (Win+L locks the PC), and `Delete` with both Ctrl and Alt held, whatever else is held (`Ctrl+Alt+Delete`, `Meh+Delete`, `CAG+Delete`, `Hyper+Delete`). Spec v4. |

**Office shortcuts on Windows** (spec v4). Windows gives some chords to
Microsoft 365: Hyper alone opens the Microsoft 365 app, and Hyper with
`W`, `X`, `P`, `O`, `T`, `D`, `N`, `Y` or `L` opens an Office app or
site. These aren't reserved. The app's keyboard hook sees them first, so
a leader or direct binding on one works while the app runs. It's an
`office-shortcut` warning, because Windows still opens Office on that
chord while the app is off, and the app offers to turn the Office
shortcuts off for the user (it sets the `ms-officeapp` handler to do
nothing). Hyper alone is never a chord in a config; the app makes sure
letting go of Hyper after a sequence opens neither Start nor Office.

## Entries

Every entry is a **menu** (no `action`) or a **binding** (an `action`).

| Field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `keys` | key path | yes | Where it sits in the tree. |
| `title` | string, not blank | yes | What the Guide and the Bindings window show. |
| `action` | action id | no | Makes it a binding. An id in `actions`, or the name of a [built-in action](#built-in-actions) (spec v11). |
| `icon` | string | no | Its icon. Mac: an SF Symbol name, or a path starting with `/` or `~` for that file's icon. Windows (spec v4): a Segoe Fluent Icons code point written `U+E756` (4 to 6 hex digits, any case), or a path for that file's icon (an `.exe`, `.ico`, `.png` or any file), starting with a drive (`C:\`), `\\` or `~`. Anything else is `bad-icon` on Windows. `""` is the same as leaving it out. |
| `hold` | boolean | no | Spec v18. Leaders only. `true` makes the leader's menu, and every menu under it, a [held menu](#held-menus). Default `false`. |
| `list` | object | no | Spec v18. Menus only. Makes the menu a [list](#lists): the app fills it with items when it opens, and it has no entries under it. |

- **Entries are a flat, ordered list keyed by their full path.** The tree
  is built from the paths, and children keep file order.
- **A top-level entry is a leader** when it's a menu, and a **direct
  binding** when it has an action. A config can mix both, or have only
  direct bindings.
- **Only the first chord has modifiers.** Step keys are bare keys.
- **A list is a menu** (spec v18): it has no `action`, and with both
  `action` and `list` an entry is `conflicting-fields`.

## Behavior both apps share

The full rules are in the Mac app's `docs/SPEC.md`. These are the ones a
user would notice if the two apps differed:

- **Each key is decided as soon as it's pressed.** No timers decide which
  entry a key picks, so there are no ambiguous prefixes. In a [held
  menu](#held-menus) (spec v18), letting go of the modifiers decides
  instead, and still no timer does.
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

## Held menus

Spec v18. A held menu works the way ⌘Tab and Alt+Tab do. Hold the
leader's modifiers, move a highlight with keys, and let go to choose
the highlighted item. A quick press and release chooses the first item,
with no Guide.

```json
{ "keys": "CAG+A", "title": "Apps", "hold": true },
{ "keys": "CAG+A T", "title": "Terminal", "action": "terminal" },
{ "keys": "CAG+A B", "title": "Browser", "action": "browser" }
```

Here CAG+A, let go, opens Terminal. Holding CAG and pressing A, then B,
then letting go opens Browser. Holding CAG and pressing A twice, then
letting go, opens Browser too.

**Which menus are held.** `"hold": true` goes on a leader. Its menu
and every menu under it are held menus, while the leader's modifiers
stay held. Every other menu works as before, so you can press its leader,
let go, and then press its keys.

**The words.**

- **The highlight** is on one item of the menu at a time. The Guide
  shows which.
- **The repeat key** of a menu is the key that opened it: the leader
  chord's key for the leader's menu (`A` in `CAG+A`), and a menu's own
  step key for a menu under it (`B` in `CAG+A B`).
- **Cancel** is the first item of every held menu. It has no key and
  runs nothing.

**While the modifiers are held:**

- **Opening a held menu** puts the highlight on its first item after
  Cancel. In a menu with no items but Cancel, the highlight is on Cancel.
- **The repeat key** moves the highlight to the next item. After the
  last item it goes to Cancel, then to the first again.
- **Shift with the repeat key** moves the highlight back, when Shift
  isn't one of the leader's modifiers. With a leader that has Shift,
  such as `Hyper`, only the repeat key and Up and Down move the
  highlight.
- **Down and Up** move the highlight forward and back, as the repeat key
  and Shift do, unless the menu has an item on that key. Then the item
  wins. Left and Right do nothing.
- **A key held down** moves the highlight once. Its repeats do nothing,
  so the highlight never runs past where you stopped.
- **A binding's key** moves the highlight to that binding. It doesn't
  run yet.
- **A menu's key** opens that menu at once, as a held menu with the
  highlight on its first item. Its key becomes the repeat key.
- **Cancel** (`runtime.cancelKey`) ends the sequence at once, and runs
  nothing.
- **Back** (`runtime.backKey`) goes up one menu, with the highlight on
  the menu you came from. From the leader's menu, Back opens Home, as in
  any sequence. Home and what follows from it aren't held.
- **Any other key** is an error flash or sound, as in any sequence, and
  the highlight stays where it is.
- **No `timeoutMs`** while the modifiers are held. The Guide appears
  after `hudDelayMs`, as in any sequence. Each key restarts that wait.

**Letting go:** the first of the leader's modifiers to come up chooses.
An app compares which modifiers are held, not the order they come up
in, so a key remapper that lets go of several at once and a hand that
lets go of one at a time choose the same item. Then the app does what
the highlighted item does:

- **A binding** runs its action, and the sequence ends.
- **A menu** opens as an ordinary menu. The sequence goes on in it with
  no modifiers held, as after any leader, and `timeoutMs` applies.
- **Cancel** ends the sequence, and runs nothing. That isn't a failure.
- **A Context Aware binding with nothing to run for the app in front**
  stays in its menu, now an ordinary menu, and shows that, as in any
  sequence.

Letting go before the Guide appears chooses as well, and the Guide never
appears. A sequence is never left waiting because a key-up was missed:
when an app finds that the leader's modifiers are no longer held, it
chooses then. It checks at least every 200 ms while a held menu is open.

**A held menu's keys can't be its repeat key.** A step key under a held
leader that's the same as its menu's repeat key is `step-is-repeat-key`.
So `CAG+D D` is fine unless `CAG+D` has `hold`. Up and Down are
fine as step keys: the item wins. A held list's own key can't be `0` to
`9` either, since its items have those keys.

**`hold` needs modifiers.** A leader with `hold` must have at least one
modifier, since letting go of it chooses. Without one it's
`hold-no-modifiers`. `hold` on a direct binding or on a menu that isn't
a leader is `hold-not-leader`. A held leader's menu under Home works as a
held menu only when its leader was pressed. From Home it's an ordinary
menu.

**On Windows**, a held leader can be `Alt+Tab`. The app's keyboard hook
sees it before Windows does, so the app's menu replaces the Windows
switcher while the app runs. Where the hook sees nothing (over a window
of an app running as administrator, when the app isn't, and on the
secure desktop of UAC and Ctrl+Alt+Delete), Windows' own switcher still
works. Letting go of Alt after a held menu opens neither the window's
menu bar nor Start. On the Mac, the Dock takes ⌘Tab: see
[Planned](#planned).

## Lists

Spec v18. A list is a menu the app fills when it opens, from what's
on screen, instead of from entries. Its `list` object says what it holds.

```json
{ "keys": "CAG+H", "title": "Recent Windows", "hold": true, "list": { "type": "windows.recent" } }
```

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `type` | `windows.recent` | required | What the list holds. |
| `count` | integer from 1 to 99 | `9` | The most items it holds. |

- **No entries under a list.** An entry whose parent is a list is
  `list-has-children`. A list is never `empty-menu`.
- **Items are fixed when the list opens.** A window that opens while the
  list is up isn't added, and the order doesn't change.
- **Keys:** items get the keys `1` to `9`, in order. With more than 9,
  the list has pages of 9, and `0` is **More**, which goes to the next
  page, where the keys are `1` to `9` again. The last page has no More.
  Back goes to the page before. From the first page, Back works as in any
  menu.
- **In a held list**, the repeat key and Down go from item 9 onto the
  next page, so the highlight reaches every item. After the last item it
  goes to Cancel, on the first page. The Guide shows the page the
  highlight is on. More is a key, not an item: the highlight never rests
  on it.
- **When an item goes away while the list is up** (its window closes),
  it leaves the list, and the keys after it move up. If it had the
  highlight, the highlight goes to the next item, else the one before,
  else Cancel.
- **In the Guide,** an item shows its own title and icon.
- **In a menu that isn't held**, a list works as any menu: press an
  item's key and it runs. It has no Cancel item and no highlight.

### `windows.recent`

Recent Windows: the windows used last, most recent first, from the
[focus history](#window-actions).

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `minimized` | boolean | `true` | Include minimized windows. |
| `hidden` | boolean | `true` | Include the windows of hidden apps. Mac only: on Windows it's `other-platform-field`. |
| `spaces` | `all`, `current` | `all` | `all`: windows on every Space or virtual desktop the app can bring forward. `current`: only the current Space or desktop of each display. |
| `scope` | `all`, `display` | `all` | As for Focus Window: windows on every display, or only on the focused window's display. |
| `choose` | `focus`, `swap` | `focus` | Spec v19. What choosing an item does: `focus` goes to its window, `swap` trades places with it, as [Swap Window](#windowswap) does. |

- **Cancel is the focused window.** In a held list, Cancel shows the
  focused window's title and icon, so choosing it changes nothing. With
  no focused window, it's Cancel.
- **The items** are the windows Focus Window could pick, for `minimized`,
  `hidden`, `spaces` and `scope`, without the focused window, in the
  order of the focus history. Whether you can see them doesn't matter.
  After the windows in the history come the windows the app can find
  that the history doesn't hold yet, the one in front first. Windows of
  an app with `"focus": false` in [`windows.apps`](#windows) are never
  items.
- **An item's title** is its window's title, or its app's name when the
  window has none. Its icon is its app's. The Guide marks a window
  that's minimized, hidden, on another Space or desktop, or on another
  display.
- **Choosing an item** focuses its window as Focus Window's `recent`
  does: it brings back a minimized window, shows a hidden app, and goes
  to the window's Space or desktop. The window goes first in the focus
  history as soon as it's chosen, so pressing the leader again right
  away goes back.
- **Nothing to list:** a held list with no items has only Cancel, and
  choosing it does nothing. A list that isn't held with no items shows
  that it's empty, and every key but Back and Cancel is an error flash
  or sound.
- **On the Mac**, a window on another Space is only found once it has
  had focus since the app started. It's in the focus history then.

#### `"choose": "swap"`

Spec v19. Choosing an item trades places with its window: each takes the
other's frame, as Swap Window does, and the focused window keeps focus.

```json
{ "keys": "CAG+Tab", "title": "Swap With", "hold": true, "list": { "type": "windows.recent", "choose": "swap" } }
```

- **The items** are the windows on the current Space or desktop of each
  display that aren't minimized and whose app isn't hidden, for `scope`,
  without the focused window, in the same order as for `focus`. Whether
  you can see them doesn't matter. Windows of an app with
  `"swap": false` in [`windows.apps`](#windows) are never items; an
  app's `focus` rule doesn't apply. [Dialogs](#dialogs) are never items
  (spec v21).
- **`minimized`, `hidden` and `spaces` don't go with it:** with
  `"choose": "swap"`, any of them is `conflicting-fields`. `count` and
  `scope` work as for `focus`.
- **The focus history doesn't change**, so pressing the leader again
  right away swaps the same two windows back.
- **With no focused window** there's nothing to swap: a held list has
  only Cancel. When the focused window's app has `"swap": false`,
  choosing an item fails as Swap Window does ("Calculator is left out of
  Swap Window.").

## Actions

`actions` maps an id to an action. Every action has a `type`; the other
fields depend on it. Unknown fields are errors.

| `type` | UI name | Fields | Behavior |
| --- | --- | --- | --- |
| `application` | Use Application | Mac: `bundleId` (required). Windows: `exe` or `appId`, exactly one. `mode` | `focusOrStart` (default): bring the app forward, starting it if needed. `focus`: only if it's running. `start`: start a new copy even if one is running. Focusing returns to the app's last focused window, which can be a dialog (spec v21). |
| `open` | Open | `target` (required) | Opens a file, folder or URL with its default app. A leading `~` is the home folder. |
| `exec` | Run Command | `command` (required, array, at least one item), `cwd`, `env` | Starts a process directly: no shell, no expansion except a leading `~`. `cwd` defaults to the home folder. Doesn't wait, but reports a failure to start. |
| `send` | Send Shortcut | `keys` (required, key path) | Presses one or more chords in the app that was in front. |
| `contextual` | Context Aware | `variants` (context id to action), `fallback` | Runs the variant for the first context, in `contexts` order, that matches the app in front. Otherwise runs `fallback`. With no fallback, the action is unavailable for that app. Variants and the fallback can't be `contextual`. |
| `noop` | Do Nothing | none | Does nothing. |
| `help` | Show Help | none | Opens the Help page of the Bindings window. |

- **How each platform names an app.** Mac: `bundleId`, a bundle
  identifier such as `dev.zed.Zed`. Windows (spec v4), exactly one of:
  - `exe`: the program's file name, such as `Code.exe`, or its full
    path. It must end in `.exe`, compared case-insensitively. A file name
    matches a running program with that file name in any folder; a path
    matches only that file. To start it, a file name is looked up in App
    Paths and then `PATH`.
  - `appId`: an AppUserModelID, such as
    `Microsoft.WindowsTerminal_8wekyb3d8bbwe!App`, for Store and packaged
    apps, which have no plain exe to start. Not blank. It starts through
    `shell:AppsFolder\<appId>`.
- `bundleId` on Windows is `other-platform-field`: each platform has its
  own config file, so a Mac field there is a mistake, not a fallback.
- When an action fails (app not installed, command can't start), the
  sequence ends and the failure shows in the Guide and the log.

### Built-in actions

Spec v11. A binding can name a **built-in action** instead of an id in
`actions`, so common actions need no definition:

```json
{ "keys": "Hyper+Space W H", "title": "Left Half", "action": "window-place-left-half" }
```

**The names** follow one rule, so a new value gets its built-in name
with no list to update. A name is the action's `type` with its dots as
hyphens, then, for a type with a main field, a hyphen and one of that
field's values. A value in camel case is written in lowercase words
joined by hyphens (`leftTwoThirds` is `left-two-thirds`), and a number
is written in digits with no leading zero. Every other field has its
default. Names are case-sensitive, like ids.

| `type` | Main field | Built-in names | Example |
| --- | --- | --- | --- |
| `window.fill` | none | `window-fill` | Fill, with Press Again to Restore on |
| `window.minimize` | none | `window-minimize` | |
| `window.place` | `preset` | `window-place-` and a preset | `window-place-left-half`, `window-place-top-right` |
| `window.resize` | `step` | `window-resize-grow`, `window-resize-shrink` | the default widths, as `step` uses them |
| `window.arrange` | `count` | `window-arrange`, and `window-arrange-` and a count from 2 to 8 or `all` | `window-arrange-3`, `window-arrange-all` |
| `window.focus` | `target` | `window-focus-` and a target: a word, or a number from 1 | `window-focus-left`, `window-focus-next`, `window-focus-recent`, `window-focus-2` |
| `window.swap` | `target` | `window-swap-` and a target | `window-swap-left`, `window-swap-first`, `window-swap-recent`, `window-swap-3` |
| `space.switch` | `target` | `space-switch-left`, `space-switch-right` (spec v17), and `space-switch-` and a number from 1 (spec v22) | `space-switch-1` |
| `space.send` | `target` | `space-send-left`, `space-send-right`, and `space-send-` and a number from 1 (spec v20) | `space-send-2`, which stays where you are |
| `space.follow` | `target` | `space-follow-left`, `space-follow-right`, and `space-follow-` and a number from 1 (spec v23) | `space-follow-2`, which goes with the window |
| `help` | none | `help` | Show Help |
| `noop` | none | `noop` | Do Nothing |

- `window-arrange` is `{ "type": "window.arrange" }`, and
  `window-place-left-half` is `{ "type": "window.place", "preset":
  "leftHalf" }`. `window-focus-2` is `{ "type": "window.focus",
  "target": 2 }`, on every display.
- Other types have no built-in names: Use Application, Open, Run Command
  and Send Shortcut have no defaults to use, Move Window's spots have no
  names in this spec, and Context Aware needs its variants.
  `window.maximize` is an old name, and has no built-in name of its own.
- **An action in `actions` with a built-in name overrides it** for the
  whole config. It can be any action, of any type, so a config can hand
  a built-in to another tool: an `exec` action called `window-focus-left`
  that runs a window manager's own command replaces Focus Window
  everywhere the config names it. An override isn't a warning. Like any
  action, it's `unused-action` when no binding uses it.
- **Which action a binding names:** the id in `actions`, when there is
  one, and otherwise the built-in action of that name. Neither is
  `undefined-action`, as for `window-place-left-hlf` or `window-swap-0`.
- An app never writes a built-in action into `actions` unless the user
  asks to change it, and then the change is an override.
- In the Bindings window, a built-in action shows its UI name and its
  value, such as "Place Window: Left Half".

### Window actions

Spec v2. Every window action works on the **focused window of the app in
front**, the same app the Guide leaves in front. With no focused window,
the action fails, and the failure shows as for any other action. Focus
Window is the exception (spec v5): see [`window.focus`](#windowfocus).

**No window there** (spec v10): when Focus Window or Swap Window finds no
window to go to or swap with, nothing moves and the action fails, as
any action does. The app also plays the platform's alert sound, when
`ui.errorBell` is on: the system alert on the Mac, which also flashes
the screen when that's turned on in Accessibility settings, and
`MessageBeep(MB_OK)` on Windows. Other failures, such as no focused
window, don't play it.

**The focus history** (spec v13): while it runs, the app keeps the
windows that had focus, the most recent first, each window once. A
window goes first each time it gets focus, however it got it: a click,
the platform's app switcher, or an action. The history isn't in the
config, and it starts empty when the app starts. An app may keep only
the last few windows, but at least 10. Focus Window's and Swap Window's
`recent` target reads it.

| `type` | UI name | Fields | Does |
| --- | --- | --- | --- |
| `window.fill` | Fill | `toggle` (default `true`) | Fills the usable area, less the gap. Not full screen. With `toggle`, a window that already fills it goes back to the frame it had before. |
| `window.maximize` | Fill | `toggle` (default `true`) | The old name of `window.fill`, from before spec v7. The same action. |
| `window.minimize` | Minimize | none | Minimizes the window, as the platform's own shortcut does (⌘M, Win+Down). Focus moves the same way. |
| `window.place` | Place Window | `preset` or `frame` | Moves and resizes the window to a part of the usable area. |
| `window.move` | Move Window | `to` or `by` | Moves the window. Its size stays the same. |
| `window.resize` | Resize Window | `to`, `by` or `step`; `anchor`; `widths` | Resizes the window. |
| `window.arrange` | Arrange Windows | `count`, `groups`, `layout` | Lays out the windows used last, on the focused window's display, in a grid (spec v8). |
| `window.focus` | Focus Window | `target` (required; `direction` before spec v11), `scope` | Focuses the nearest window in a direction, the next or previous one, the first or last one, one by number, or the window used before (spec v13). |
| `window.swap` | Swap Window | `target` (required), `scope` | Trades place and size with the window in a direction, the next or previous one, the first or last one, one by number (spec v10), or the window used before (spec v13). |

Window families use a dotted `type`, one family per module. Later
families follow the same shape, such as `space.switch` (spec v17),
`audio.volume` and `display.resolution`.

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
- **The gap** (spec v3, `windows.gap`, rounded to whole points) applies
  to Place Window and, from spec v7, to Fill, and to each cell of Arrange
  Windows (spec v8). Fill's frame is the whole usable area. After rounding a frame's edges, each edge on
  the usable area's edge moves in by the whole gap, and each edge inside
  it moves in by half: a left or top edge by the larger half, a right or
  bottom edge by the smaller, so neighbours end exactly one gap apart.
  With a gap of 8, the left half's right edge and the right half's left
  edge are 8 points apart, and each half is 8 points from the screen's
  edges. A window always keeps at least 1 point of width and height.
- **A window that won't take a size** (a minimum, a maximum, a fixed
  size) ends as close as it allows, and the action doesn't fail. It keeps
  the edge that's on the usable area's edge (with Place Window and Fill,
  one gap in from it), or its center for a centered frame.
- **A window never ends off screen.** After a move or a resize, at least
  40 points of its title bar stay on the screen it's on.

#### `window.fill`

Spec v7. Before, this type was `window.maximize`, with the UI name
Maximize, and it left no gap.

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `toggle` | boolean | `true` | When the window fills the usable area already (within 2 points of the frame Fill gives it, on every edge), put it back where it was before Fill. |

Fill leaves the gap from [`windows`](#windows) on every side of the
window, as Place Window does for a frame that covers the whole usable
area. With a gap of `0`, the window goes edge to edge.

The frame before Fill is kept per window, in memory only. A window
moved or resized since then forgets it, and Fill leaves it as it is.

**`window.maximize`** is the old name of this type and still loads. An
app reads it as `window.fill`: the same field, the same rules, the same
gap. When an app saves a file, it keeps the name the file uses, so an app
on an older spec can still read the file. A new action gets
`window.fill`. The old name stays until a later spec says otherwise.

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
1 within 0.001 counts as 1. Every placement leaves the gap from
[`windows`](#windows) around the window (spec v3).

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

#### `window.arrange`

Spec v8, with the focused window's cell from spec v9. Arrange Windows
lays out the windows used last, on one display, in a grid of **cells**. It has no toggle and keeps no memory. Focus
doesn't change.

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `count` | integer from 2 to 8, or `"all"` | `2` | How many cells the grid has: one for the focused window and one for each window used before it. `"all"` makes a cell for every window on the display, up to the limit below. |
| `groups` | integer from 1 to 8 | the app picks | How many columns, or rows, the grid has. With a number for `count`, at most that number. |
| `layout` | `auto`, `columns`, `rows` | `auto` | Whether the groups are columns or rows. `auto` is `columns` on a usable area at least as wide as it is tall, and `rows` on a taller one. |

- **The windows** are Focus Window's candidates on the display of the
  focused window: the display that holds the largest part of it. Windows
  on other displays are never counted or moved. They're taken from the
  front: the focused window, then the others, front to back, which is
  the order they were used in. With no focused window, the candidate in
  front of the others counts as the focused one. With no candidate, the
  action fails ("No window to arrange.").
- **Dialogs** (spec v21) are never counted or moved: see
  [Dialogs](#dialogs).
- **Left out** (spec v15): the windows of an app with `"arrange": false`
  in [`windows.apps`](#windows) are never counted or moved, but still
  cover the windows behind them. An app's `focus` and `swap` rules don't
  change what Arrange Windows does. When the focused window's app is
  left out, its display is still the one arranged, and the candidate in
  front of the others there counts as the focused one.
- **How many cells:** `count`. With `"all"`, the number of windows on the
  display, at most `windows.maxColumns` times `windows.maxRows`, and never
  more than 8. Windows past that stay as they are.
- **How many groups:** `groups`. Without it, as many groups as cells, at
  most `windows.maxColumns` for columns or `windows.maxRows` for rows.
  There are never more groups than cells. A `groups` in the action may be
  more than the setting, which only limits what the app picks.
- **The grid:** the groups split the usable area equally, by width for
  columns and by height for rows. Every group splits into the same
  number of equal parts: the cells divided by the groups, rounded up. A
  cell is one part, or several in a row for the focused window. Edges
  round as for every frame, and each cell gets the gap, as with Place
  Window.
- **Spare parts:** when the cells don't divide evenly, the grid has more
  parts than cells. The parts left over are the spare parts, and the
  focused window's cell gets them (spec v9; before, the focused window
  moved to a roomier cell on the right or at the top).
- **Order:** windows keep the order they're in. For columns, they sort
  by center from left to right and fill the columns in that order, and
  each column's windows sort from top to bottom. For rows, the same with
  top to bottom first. Two centers within 2 points on an axis count as
  the same there, and the other axis decides. Windows still tied go by
  when they were used: the one used more recently goes further right, or
  further up.
- **With no spare parts,** every window goes by that order, so with
  windows in the same spot the focused one ends on the right or at the
  top.
- **With spare parts,** the focused window stays in the group its center
  is in: the column that holds its center, or the row. A center on the
  line between two groups counts for the one further right, or further
  down, and a center outside the usable area for the nearest group. The
  focused window's cell there is one part and the spare parts, up to
  the whole group. The other windows fill the other parts in order, and
  the windows of the focused window's group sort across it with the
  focused one, so each stays on its side of it.
- **More spare parts than the focused window's group has:** the focused
  window takes its whole group. The other windows go in the other
  groups, in order, and each of those groups splits equally between its
  cells. When they don't divide evenly, the extra cells go one each to
  the leftmost of those columns, or to the bottom rows, so the groups
  with fewer cells are on the right and at the top.
- **Fewer windows than cells:** a missing window counts as furthest left
  and furthest down, so the empty cells end on the left or at the
  bottom.
- Windows that are in their cells already don't move, so the action can
  be repeated. A window that won't take its cell's size ends as with
  Place Window.

With the default settings, on a wide display: `{ "type": "window.arrange" }`
puts the two windows used last in halves. `"count": 3` gives the
focused window the half it's in, and the two others stack in the other
half. With `windows.maxColumns` at 3, `"count": 5` makes three columns:
the focused window fills the column it's in, and the others stand two
to a column. `"count": 8, "groups": 3` gives the focused window two
thirds of its column, with one window in the other third.
`"count": 3, "groups": 3` gives three equal columns. `"layout": "rows"`
stacks the two. `"count": "all"` arranges up to four windows.

#### `window.focus`

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `target` | `left`, `right`, `up`, `down`, `next`, `previous`, `first`, `last` (the last two from spec v6), `recent` (spec v13), or an integer from 1 (spec v11) | required | Which window to focus. |
| `scope` | `all`, `display` | `all` | Spec v10. `all`: windows on every display. `display`: only the windows on the focused window's display. |

- **`direction`** is the old name of `target`, from before spec v11. It
  still loads and takes the same values. An action with both is
  `conflicting-fields`. When an app saves a file, it keeps the name the
  file uses, as with `window.maximize`, and a new action gets `target`.
- **Candidates** are the standard windows and, from spec v21, the
  [dialogs](#dialogs), that are visible and aren't minimized,
  on the current Space or desktop of every display. Panels, the Guide and
  the app's own windows are skipped, and so are the windows of an app
  with `"focus": false` in [`windows.apps`](#windows) (spec v15). The
  focused window is still where a direction starts when its app is left
  out, but it isn't in the order,
  so `next` and `previous` from it go to the first and the last
  candidate.
- **Only windows you can see count** (spec v12). A candidate's visible
  part is its frame minus every candidate in front of it, on any display.
  A candidate counts only when its visible part holds a square
  `windows.minVisible` points on a side (20 by default). A window hidden
  behind others, or showing only a thin strip, is skipped by every
  `target` but `recent`. The focused window always counts.
- **With `"scope": "display"`** (spec v10), the candidates are only those
  on the focused window's display: the display that holds the largest
  part of it. With no focused window, the display of the candidate in
  front of the others. Every `target` then stays on that display, and
  the order below is that display's. Windows on other displays still
  hide the ones behind them.
- **A direction** (spec v12) looks at the part of each candidate's
  visible part that lies past the focused window's center in that
  direction. A candidate with no such part isn't there. Of the rest:
  - A candidate is **in line** when that part shares at least
    `windows.minVisible` points with the focused window across the
    direction: some of the same height for `left` and `right`, some of
    the same width for `up` and `down`. Candidates in line come first.
  - Of those in line, the nearest wins: the smallest gap from the
    focused window's edge on that side to the near edge of that part, or
    no gap when they overlap. Gaps within `windows.tieTolerance` points
    (2 by default) count as the same. Then the one that shares more,
    then the one whose center is nearest the focused window's center
    across the direction, then the one in front.
  - With none in line, the window whose center is nearest the focused
    window's center wins, among those within 45° of that direction,
    measured to the center of that part. A tie goes to the one in front.
  - With none, the action fails ("No window to the left.").
  - So a window inside a larger one goes to the larger one in every
    direction, and a window directly below a wide one is found even when
    its center is far to the side.
- **The order** of the candidates is fixed (spec v12; before, by center,
  left to right first): by display, left to right, then in rows, as text
  reads. Each window's place is the top left corner of the box around
  its visible part. The highest window starts a row, and every window
  whose top is within `windows.rowTolerance` points of that one's (20 by
  default) joins it. Rows go top to bottom, and each row goes left to
  right. The same windows in the same places always give the same
  order, whatever was focused last. A window that becomes hidden leaves
  the order, and the ones after it move up.
- **`next` and `previous`** go through the candidates in that order. They
  wrap at the ends.
- **`first` and `last`** (spec v6) focus the first and the last candidate
  in that order: the top left and the bottom right window. On a window
  that is already there, nothing happens, and that isn't a failure.
- **A number** (spec v11) focuses the candidate at that place in the
  order, counting from 1, so `1` is `first`. In a grid of three columns
  and two rows, `2` is the top middle window. It works as `first` and
  `last` do. Past the last candidate, the action fails ("No window 5.
  There are 3.").
- **`recent`** (spec v13) focuses the window used before: the first
  window in the [focus history](#window-actions) that is a candidate and
  isn't the focused window. Whether you can see it doesn't matter, so it
  can be behind other windows; `scope` still applies. The window that
  had focus becomes the one used before, so pressing again goes back.
  When the window used before was closed or minimized, it's the one used
  before that. When the history has no window that counts, as right
  after the app starts, it's the candidate in front of the others, after
  the focused window. With none, the action fails ("No window used
  before.").
- **On another Space** (spec v13): on the Mac, the window used before
  can also be on another Space. Focusing it moves to that Space, as the
  app switcher does. On Windows, a window on another virtual desktop
  counts when the app can bring it forward. Every other `target` only
  sees the current Space or desktop.
- **With no focused window** (spec v5), or no app in front, `first`,
  `last` and numbers work as always, `recent` focuses the first
  candidate in the history, and every other `target` focuses the
  candidate in front of the others. With no candidate, the action fails
  ("No window to focus.").

#### Dialogs

Spec v21. Many apps' Settings windows are dialogs, so they can be
reached as windows can, but never change places or size with them.

- **What a dialog is.** On the Mac: a window whose accessibility subrole
  is dialog. On Windows: a top-level window with an owner that is
  visible, enabled, has a caption and isn't a tool window. Tooltips,
  menus, palettes and owned windows with no caption aren't dialogs, and
  are never candidates.
- **Where a dialog counts:** as a candidate for every `target` of Focus
  Window, as an item of Recent Windows, as a window Use Application
  returns to, and in the focus history when it takes focus.
- **Where it doesn't:** a dialog is never a candidate for Swap Window, an
  item of a list that swaps, or a window Arrange Windows counts or
  moves. It still covers the windows behind it.
- **With a dialog focused,** Swap Window and choosing in a list that
  swaps move nothing and fail with the alert sound ("A dialog is left out
  of Swap Window."). Arrange Windows treats it as a left-out app's
  window: its display is still the one arranged, and the candidate in
  front of the others there counts as the focused one. Fill, Minimize,
  Place, Move and Resize Window work on it as on any window.
- **A window a dialog blocks** (Windows): a window that a modal dialog of
  its own disabled stands aside for that dialog in Focus Window, Recent
  Windows and Use Application, since it can't take input. It's still
  swapped and arranged.

#### `window.swap`

Spec v10. Swap Window trades places with another window: each takes the
other's frame. The focused window keeps focus, so pressed again it goes
on in the same direction.

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `target` | `left`, `right`, `up`, `down`, `next`, `previous`, `first`, `last`, `recent` (spec v13), or an integer from 1 | required | Which window to swap with. |
| `scope` | `all`, `display` | `all` | As for Focus Window: windows on every display, or only on the focused window's display. |

- **Candidates and their order** are Focus Window's, for the same
  `scope`, so a window you can't see is never swapped with (spec v12),
  except by `recent`. An app's `focus` rule doesn't apply here: its
  `swap` rule does (spec v15). A window of an app with `"swap": false`
  is never a candidate, and when the focused window is one, the action
  fails ("Calculator is left out of Swap Window."), with the app's name.
  A [dialog](#dialogs) is never a candidate either (spec v21).
  Its `target` takes the same values, and from spec v11 so does
  Focus Window's.
- **A direction** picks the window Focus Window would focus. With none,
  the action fails ("No window to the left.").
- **`next` and `previous`** are the windows after and before the focused
  one in the order. They wrap at the ends. With no other window, the
  action fails ("No other window to swap with.").
- **`first`, `last` and a number** count in the order. Numbers start at 1
  and count the focused window, so `1` is `first`, and after a swap with
  `2` the focused window is the second. Past the last window, the action
  fails ("No window 5. There are 3."). A target that is the focused
  window does nothing, and that isn't a failure.
- **`recent`** (spec v13) is the window Focus Window's `recent` would
  focus, which can be behind other windows, but only on the current
  Space or desktop, since trading frames doesn't bring a window over
  from another one. Since the focused window
  keeps focus, pressing again swaps them back. With none, the action
  fails ("No window used before.").
- **The swap:** each window takes the other's frame as it is, with no
  gap added, across displays too. A window that won't take the size ends
  as close as it allows, as with Place Window, and the action doesn't
  fail. Focus, and which window is in front, don't change.
- **With no focused window**, the action fails, as every window action
  does.
- A `left` swap and then a `right` one put two windows side by side back
  where they were.

### Space actions

Spec v17. A Space on the Mac, a virtual desktop on Windows. Each
platform's UI uses its own word, as the glossary says.

| `type` | UI name | Fields | Does |
| --- | --- | --- | --- |
| `space.switch` | Switch Space (Windows: Switch Desktop) | `target` (required) | Goes to the Space or desktop to the left or right of the current one, or to one by number (spec v22). |
| `space.send` | Send to Space (Windows: Send to Desktop) | `target` (required) | Puts the focused window on the Space or desktop to the left or right, or on one by number, and you stay (spec v20). |
| `space.follow` | Follow to Space (Windows: Follow to Desktop) | `target` (required) | Puts the focused window there as Send to Space does, and you go with it (spec v23). |

#### `space.switch`

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `target` | `left`, `right`, or an integer from 1 | required | Where to go: the Space or desktop to the left or the right of the current one, or the one with that number (spec v22), counting from 1 in the order Mission Control or Task View shows them. |

```json
"desktop-one": { "type": "space.switch", "target": 1 }
```

- **Which display:** the focused window's, the display that holds the
  largest part of it, since you're working from the keyboard there,
  wherever the pointer is. On Windows, and on the Mac with Displays have
  separate Spaces off, every display switches together.
- **With no focused window**, or no app in front, `left` and `right` do
  nothing. That isn't a failure, and no sound plays. A number still
  switches, on the display with the pointer, since an empty desktop is
  where you most often want to leave from (spec v22).
- **Full-screen apps** count as Spaces on the Mac for `left` and
  `right`, each in its place in the order, as they do for the
  platform's own shortcut. **Numbers count desktops only,** as for
  [Send to Space](#spacesend) and the Mac's own Switch to Desktop
  shortcuts: full-screen apps aren't counted.
- **At the first or last Space**, or with only one, `left` and `right`
  move nothing and don't wrap. That isn't a failure, and no sound plays,
  since the platform shows its own bounce, or nothing.
- **A number you're already on** does nothing, with no sound. **A number
  past the last desktop** fails, with the alert sound.
- **Quickly:** an app switches as quickly as the platform allows, and
  skips the sliding animation where it can. How it does that is each
  app's own. When an app can only switch with the animation, it still
  switches: the action never fails for that. To reach a number, an app
  may go one Space at a time, as the Mac app already does when Send to
  Space follows a window.
- **Focus** goes where the platform puts it after a switch, usually the
  window in front on that Space. It counts as a focus change for the
  [focus history](#window-actions).

#### `space.send`

Spec v20.

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `target` | `left`, `right`, or an integer from 1 | required | Where the window goes: the Space or desktop to the left or right of its own, or the one with that number, counting from 1 in the order Mission Control or Task View shows them. |

```json
"to-two": { "type": "space.send", "target": 2 }
```

- **Which window:** the focused one. **Which display:** the one the
  window is on, the display that holds the largest part of it. On
  Windows, and on the Mac with Displays have separate Spaces off,
  desktops span every display.
- **Numbers count desktops only.** On the Mac, full-screen apps aren't
  counted and can't take a window, as with the Mac's own Switch to
  Desktop shortcuts. `left` and `right` skip them too.
- **You stay:** the Space or desktop on screen stays, and focus goes to
  the [window used before](#window-actions) there, or to none. Typing
  never goes to a window you can't see. To go with the window, use
  [Follow to Space](#spacefollow).
- **The focus history** keeps the sent window, so Focus Window's
  `recent` target can go back to it on its new Space.
- **These fail, with the alert sound:** no focused window; a number past
  the last desktop; a window in full screen, or set to show on every
  Space or desktop; or the platform refusing the move.
- **Nothing happens, with no sound,** when the window is already there,
  or at the first or last Space for `left` or `right`. It doesn't wrap.
- **However the platform allows:** the platform has no public way to do
  this on the Mac, so how an app moves the window is its own. It may
  slide, or show the other Space for a moment; the action never fails
  for that.
- **`follow`** (spec v20 to v22) is gone. A `space.send` with it is
  `unknown-field`; use `space.follow` for `"follow": true`, and leave it
  out for `false`.

#### `space.follow`

Spec v23. Follow to Space does what Send to Space does, then goes to
that Space or desktop with the window. It has the same field, and the
same rules for which window, which display, numbers, failures and
nothing happening.

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `target` | `left`, `right`, or an integer from 1 | required | Where the window goes, and you with it, as for [Send to Space](#spacesend). |

```json
"with-two": { "type": "space.follow", "target": 2 }
```

- **You go with it:** the app goes to that Space or desktop too, and the
  sent window keeps focus. It counts as a focus change for the
  [focus history](#window-actions).
- **Quickly:** as for [Switch Space](#spaceswitch), an app goes there as
  quickly as the platform allows. It may go one Space at a time.
- **When nothing moves,** because the window is already there or at the
  first or last Space, you stay too, with no sound.

## Contexts

| Field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `id` | string | yes | Name used as a key in a Context Aware action's `variants`. Unique. |
| `apps` | array of strings | yes | App identifiers. Mac: bundle ids. Windows (spec v4): a string ending in `.exe` is an `exe` (file name or full path), and any other is an `appId`, matched as Use Application matches them. A blank string is `bad-app-id` on Windows. |

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
| `missing-field` | A required field is missing (`version`, `entries`, an entry's `keys` or `title`, a context's `id` or `apps`, an action's `type` or a field its type requires, a `list`'s `type` (spec v18), both fields of a pair where a window action needs one, or both `exe` and `appId` on Windows). |
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
| `step-is-repeat-key` | Spec v18. Under a leader with `hold`, a step key is the same as its menu's [repeat key](#held-menus): the leader chord's key for the leader's own keys, such as `CAG+D D`, or its menu's step key below that, such as `CAG+A B B`. Or a held [list](#lists)'s own key is `0` to `9`, such as `CAG+1`, since its items have those keys. |
| `hold-not-leader` | Spec v18. An entry with `"hold": true` isn't a leader: it's a direct binding, or a menu or binding under another menu. |
| `hold-no-modifiers` | Spec v18. A leader with `"hold": true` has a chord with no modifiers, such as `F13`. |
| `list-has-children` | Spec v18. An entry's parent is a [list](#lists). |
| `unknown-list-type` | Spec v18. A `list`'s `type` isn't one of the types in [Lists](#lists). |
| `bad-list-value` | Spec v18. A `list`'s `count` isn't from 1 to 99, or its `spaces`, `scope` or `choose` (spec v19) isn't one of its values. |
| `bad-runtime-key` | `cancelKey` or `backKey` isn't a key name. |
| `same-cancel-back` | The cancel and back keys are the same. |
| `negative-delay` | `hudDelayMs`, `timeoutMs` or `windows.quitOnCloseDelayMs` (spec v16) is negative. |
| `negative-gap` | `windows.gap` is negative. |
| `bad-window-limit` | `windows.maxColumns` or `windows.maxRows` isn't from 1 to 8. |
| `negative-window-distance` | `windows.minVisible`, `windows.rowTolerance` or `windows.tieTolerance` is negative (spec v12). |
| `bad-ui-value` | `theme` or `position` isn't one of its values. |
| `reserved-chord` | A leader or direct binding uses a chord this platform reserves. |
| `undefined-action` | A binding names an action that isn't in `actions` and isn't a [built-in action](#built-in-actions). |
| `unknown-action-type` | An action's `type` isn't one of the types above. |
| `bad-app-mode` | An `application` action's `mode` isn't one of its values. |
| `undefined-context` | A Context Aware variant names a context that isn't in `contexts`. |
| `nested-contextual` | A Context Aware variant or fallback is itself `contextual`. |
| `duplicate-context` | Two contexts share an `id`. |
| `conflicting-fields` | An action has fields that don't go together: both `preset` and `frame`; two of `to`, `by` and `step`; `anchor` with `step`; `widths` without `step`; both `target` and `direction` (spec v11); an entry with both `action` and `list` (spec v18); a `list` with `"choose": "swap"` and `minimized`, `hidden` or `spaces` (spec v19); or, on Windows, both `exe` and `appId`. (Neither of a required pair is `missing-field`.) |
| `other-platform-field` | Windows only: an `application` action has `bundleId`, which only the Mac uses, or `windows` has `quitOnClose` or `quitOnCloseDelayMs`, or an app in `windows.apps` has `quit` (spec v16), or a `windows.recent` list has `hidden` (spec v18). (On the Mac, `exe` and `appId` are `unknown-field`.) |
| `bad-app-id` | Windows only: an `exe` doesn't end in `.exe`, an `appId` is blank, or a context's `apps` has a blank string, or a key in `windows.apps` (spec v15) is blank. |
| `bad-icon` | Windows only: an entry's `icon` is neither a `U+` code point nor a path. |
| `bad-fraction` | A fraction is out of its range, a `frame` runs past the usable area, or `widths` is empty. |
| `bad-window-value` | A window action's `preset`, `direction`, `step`, `anchor`, `layout` or `scope` isn't one of its values, or a `target` (or `direction`) is neither one of its words nor an integer from 1, such as `0` or the string `"2"`. Or Arrange Windows' `count` isn't `"all"` or from 2 to 8, or its `groups` isn't from 1 to 8, or is more than a `count` that's a number. Or Switch Space's `target` is neither `left`, `right` (spec v17) nor an integer from 1 (spec v22), such as `0`, `"up"` or `"2"`. Or Send to Space's or Follow to Space's `target` is neither `left`, `right` nor an integer from 1 (spec v20, v23), such as `0`, `"up"` or `"2"`. |
| `unsupported-action` | An action's `type` is in the spec, but this platform's app can't do it yet. |

### Warnings

| Rule id | Reported when |
| --- | --- |
| `unused-action` | No binding uses an action. |
| `unused-context` | No action uses a context. |
| `key-not-on-platform` | An entry uses a key this platform's keyboards don't have. |
| `office-shortcut` | Windows only: a leader or direct binding uses one of the [Office shortcuts](#reserved-chords), such as `Hyper+W`. |

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

`version` stays `1` in spec v2 to v23. They only add: new action types, new fields,
the optional `windows` object and its settings, new rules, new values for `direction`, what Focus Window does with no focused window, when the alert sound plays, built-in action names, a new name for a type or a field whose old name still loads, which windows Focus Window and Swap Window see and their order (spec v6 and v12), a history of focused windows that the app keeps (spec v13), a family for Spaces (spec v17), held menus and lists, two new entry fields (spec v18), a list field (spec v19), Send to Space (spec v20), numbers for Switch Space (spec v22), and Follow to Space (spec v23). Spec v15 drops `windows.exclude`, from spec v13 and v14, which no released app read; a file with it is `unknown-field`. Spec v23 drops Send to Space's `follow` the same way, while the format is still young: a file with it is `unknown-field`. Spec v7 also makes `window.maximize` leave the gap: no field changed meaning, and an old app still reads the file. An app that doesn't know a type rejects the config with
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
- **Inline actions:** a binding's `action` could also be an action
  object, written as Context Aware's variants are, for an action only
  that binding uses: `"action": { "type": "window.swap", "target": 3,
  "scope": "display" }`. It would cover what built-in names can't, such
  as a scope or a custom frame, with no id. Not decided: it needs an
  action form inside the binding in each app's editor.
- **Switch Space to the Space of the window used before.** Spec v22 has
  numbers; this would be one more target word.
- **Built-in names for Move Window**, once its spots (Center, Top Left
  and the others) have names in this spec.
- **⌘Tab as a held leader on the Mac.** The Dock takes ⌘Tab and ⌘⇧Tab
  before other apps' shortcuts. The only known way to stop it is a
  private call whose change outlives the app, even a crash. Not decided:
  test whether the event tap sees ⌘Tab first, then decide between
  `reserved-chord` and support.
- **Keys for the highlighted item in a held menu,** as the app switchers
  have: close or minimize a window, quit or hide an app, without letting
  go.
- **More `choose` values for a list,** such as `close`, `minimize` or
  `quit`, run on the chosen window or its app (spec v19 has `focus` and
  `swap`).
- **Search in a held list,** by typing part of a title.
- **More list types,** such as the running apps, or the windows of the
  app in front.
- **A list of Spaces,** now that Switch Space can go to one by number
  (spec v22).
- **Modifiers on step keys,** so `T` and `Shift+T` can be two entries in
  one menu. See below.
- **Place on Grid,** a list that places the focused window in a grid you
  pick as you go: how many columns, which one, how many rows, which one.
  See below, with three alternatives: **Grid Moves** (one fixed grid, a
  size in cells, moves one cell at a time), **Arrange with Limits**
  (arrange without shrinking past each app's smallest size) and
  **Columns** (windows sent to columns, with shelves for what doesn't fit). The
  section **Window Layout: What We Need** says what they're for.

### Modifiers on step keys

A proposal for spec v24. Today only the first chord has modifiers, and a
step key with modifiers is `step-modifiers`. This would let a step key
be a chord too: `CAG+Space T` focuses Terminal and `CAG+Space Shift+T`
opens a new one, with no extra level for the second.

```json
{ "keys": "CAG+Space", "title": "Leader" },
{ "keys": "CAG+Space T", "title": "Terminal", "action": "terminal" },
{ "keys": "CAG+Space Shift+T", "title": "New Terminal", "action": "terminal-new" }
```

**Matching.** A step key matches when its key is pressed and the extra
modifiers held are exactly its modifiers. The extra modifiers are the
ones held, less the leader's modifiers that have stayed down since the
leader. Once a leader modifier is let go, it counts again, as the rule
that held leader modifiers are ignored already says. With `CAG+Space`:

| You do | Matches |
| --- | --- |
| Keep CAG down, press T | `T` |
| Keep CAG down, add Shift, press T | `Shift+T` |
| Let go, press T | `T` |
| Let go, press Shift+T | `Shift+T` |
| Let go, press Ctrl+T | `Ctrl+T` |

So one entry works whether you hold the leader's modifiers or let go of
them, and no leader has to choose a style.

**What would change:**

- **Keys:** the rule "Only the first chord has modifiers" goes, and
  `step-modifiers` is removed. A step key is a key name or a chord, with
  the same names and groups as the first chord. Apps write it the same
  way: `Ctrl+Alt+Shift+T` as `Meh+T`.
- **Duplicates:** paths are compared as chords, so `T` and `Shift+T` are
  different paths, and `Shift+Alt+T` and `Alt+Shift+T` are the same one
  (`duplicate-path`).
- **A leader's own modifier:** a step key with a modifier its leader has
  can only match after you let go of it. That's a warning,
  `step-uses-leader-modifier`, not an error. Under a `Hyper` leader every
  modified step key gets it.
- **The reverse modifier** (`runtime.reverseModifier`): the modifier
  that, with the repeat key, moves a held menu's highlight back. Today
  that's always Shift. It becomes a setting, one modifier name (`Shift`,
  `Ctrl`, `Alt` or `Cmd`, or one of their other names), default
  `Shift`, so a config can use `Alt` (`Opt` on the Mac) for it. A group
  or anything else is `bad-runtime-key`. When the leader has the reverse
  modifier, it works as Shift does today in a leader that has Shift:
  only the repeat key and Up and Down move the highlight.
- **Held menus:** a step key in a held menu that is the reverse modifier
  with its menu's repeat key is an error, `step-is-reverse-key`, unless
  the leader has that modifier. Any other modified step key moves the
  highlight to its item, as a bare one does.
- **Cancel and Back** (`runtime.cancelKey`, `runtime.backKey`) work
  whatever modifiers are held, so `Shift+Escape` still cancels. A step
  key on either key, with or without modifiers, is still
  `step-is-runtime-key`.
- **Lists:** their items keep the bare keys `1` to `9` and More on `0`.
  Modified digits do nothing in a list.
- **Home:** works as any menu. Its entries are leaders and direct
  bindings, which already have modifiers.
- **Send Shortcut:** no change. Modifiers held for the step key are let
  go before the shortcut is sent, as held leader modifiers already are.
- **The Guide** shows each row's modifiers with its key, and puts `T` and
  `Shift+T` next to each other.
- **The apps' key recorders** take a chord, not a key, for a step.

**Compatibility.** Every config that's valid today stays valid and works
the same. A config that uses a modified step key is `step-modifiers` in
an app on spec v23 or older.

**Decided:**

1. ~~Should a modified step key with no match fall back to the bare
   key?~~ No. `Shift+X` with only `X` in the menu is an error flash, so
   a stray modifier never runs the wrong action.
2. ~~Should the reverse modifier be a setting?~~ Yes,
   `runtime.reverseModifier`, default `Shift`. Configs are likely to use
   `Alt` or `Opt` for it.

**Not decided:**

1. Does the Mac app's event tap see every modified step key? Option with
   a letter types a character on the Mac, but the tap consumes the key
   before any app sees it, so it should. Needs a test on the Mac.

### Window Layout: What We Need

A record of what the window layout proposals below are for, kept while
we compare them. Old options get cleaned out once a plan is chosen.

- **Displays:** a 57-inch 32:9 display (7680 by 2160 pixels, 3840 by
  1080 points, about 140 pixels per inch) and a laptop. One setup has to
  work on both.
- **Both platforms.** Whatever we choose has to work on the Mac and on
  Windows. Nothing here is for the Mac only.
- **Don't fight the OS.** Many Mac apps and dialogs handle being resized
  badly. Paneru, a scrolling tiler for the Mac, failed mostly for that
  reason. A window that refuses a size keeps the size it takes. The
  layout never tries again and again.
- **Tiling breaks past 2 to 4 windows.** A tiler divides the screen by
  the number of windows, so each window shrinks as more open. But an
  app's smallest useful size is fixed in points, whatever the display
  or the number of windows. Most apps aren't useful below half of a
  laptop screen. A few are.
- **How the work goes:**
  - Windows are set up in workspaces, one working group each, and the
    work switches between workspaces. Spaces (virtual desktops on
    Windows) should be the workspaces.
  - Now and then a short-lived window, such as a terminal, opens, gets
    used and closes. Focus should then go back to what was in use before.
  - Floating windows and dialogs have to work with focus keys. In
    Paneru, floating windows were left out of directional focus. Focus
    Window already counts every window you can see.

### Place on Grid

A proposal for a later spec version. Presets and frames cover a few
fixed parts of the screen. On a wide display you want many more: halves,
thirds, quarters and sixths across, and halves or thirds down, which is
close to a hundred frames. A binding for each is too many to learn.
Place on Grid is a [list](#lists) of type `window.grid` that builds the
frame from a few number keys instead.

```json
{ "keys": "CAG+W", "title": "Place on Grid", "list": { "type": "window.grid" } }
```

| Field | Type | Default | Meaning |
| --- | --- | --- | --- |
| `columns` | array of integers from 1 to 9, at least one | `[1, 2, 3, 4, 6]` | The numbers of columns you can pick. Order and duplicates don't matter. |
| `rows` | array of integers from 1 to 9, at least one | `[1, 2, 3]` | The numbers of rows you can pick. |

**Steps.** The list opens on the first of four steps. Each is a menu in
the Guide, and Back goes to the step before.

1. **How many columns:** one item for each number in `columns`. Its key
   is the number itself, so `6` is Sixths, not the fifth item.
2. **Which column:** keys `1` to the number of columns, left to right.
3. **How many rows:** one item for each number in `rows`, keyed by the
   number.
4. **Which row:** keys `1` to the number of rows, top to bottom. This
   key places the window.

**Skipped steps.** A step with only one choice is skipped, so the key
before it moves on to the next step, or places the window. With the
defaults:

| You press | The window goes to |
| --- | --- |
| `CAG+W 1 1` | The whole usable area |
| `CAG+W 2 1 1` | The left half, full height |
| `CAG+W 6 4 1` | The 4th sixth, full height |
| `CAG+W 3 2 2 1` | The middle third, top half |
| `CAG+W 4 1 3 3` | The first quarter, bottom third |

A config can fix a count by giving one number. With `"rows": [1]` every
placement is full height, and `CAG+W 6 4` places the 4th sixth. With
`"columns": [3]` the first key picks a third.

**Spans.** In a "which" step, `Shift` with a digit marks that column or
row as one end of a span and stays on the step. The next digit without
`Shift` is the other end, in either order, and ends the step. In Sixths,
`Shift+2` then `4` covers columns 2 to 4, half the width. Another
`Shift` digit moves the mark. Back clears the mark before it leaves the
step.

**The frame.** For `N` columns with the span from column `a` to column
`b`, `x` is `(a - 1) / N` and `width` is `(b - a + 1) / N`. Rows do the
same for `y` and `height`. The window is then placed as
[Place Window](#windowplace) places a `frame`: on the display that holds
most of it, with the gap, edges rounded, and the same rules for a window
that won't take the size. With no focused window, the list doesn't
open: it fails, as every window action does.

**In the Guide,** each step draws the usable area in its shape, with the
grid on it and the columns, or the cells, picked so far filled with the
accent. Items are titled by their count (Full Width, Halves, Thirds,
Quarters, Sixths, and "5 Columns" for counts without a name) or by
their place (Column 2, Row 1).

**How it differs from other lists:** its items come from the config,
not from what's on screen; it has several steps; count steps key items
by number, not in order; it never has More, since a step has at most 9
items; and `Shift` with a digit is a key in it.

**Compatibility.** A config that uses it is `unknown-list-type` in an
app on an older spec.

**Not decided:**

1. **Held.** Should `"hold": true` work with Place on Grid, choosing the
   highlighted cell when you let go? Each step would need its own
   highlight. Until then it would be an error, with a new rule id.
2. **Keep one axis.** Should `0` in a count step keep the window's
   current columns or rows (its `x` and `width`, or `y` and `height`)?
   `CAG+W 6 4 0` would change the width only, and `CAG+W 0 2 1` the
   height only. That would place each axis on its own without a second
   action type.
3. **Rows first.** Should a `"first": "rows"` field let a config pick
   rows before columns, for tall displays?
4. **Spans and step-key modifiers.** The proposal for
   [modifiers on step keys](#modifiers-on-step-keys) says modified
   digits do nothing in a list. Place on Grid would be the exception, so
   the two proposals have to agree.
5. **An overlay instead.** A grid drawn over the display, moved with the
   arrows, with `Shift` and the arrows to stretch it and Return to
   place, handles spans better, but it's a new kind of interaction that
   both apps would have to build. Not proposed for now.

**Tried on 2026-11-27** on the Mac, with the steps written as plain
menus and `window.place` frames (96 actions, no spans, no grid picture
in the Guide). The keys are easy to type, since they are all digits.
The cost is thinking: you see where the window should go, then have to
turn that into a count (halves, thirds) and a place in it.

### Grid Moves

An alternative to Place on Grid, recorded to compare the two. Nothing
is decided. Instead of picking a grid each time, a config sets one grid
per display, and windows are sized and moved in its cells. Size and
place are two separate steps: "make it 3 by 1, then push it right
twice".

**Prior art.** This model is well tried:

| Tool | Platform | What it does |
| --- | --- | --- |
| [Hammerspoon `hs.grid`](https://hammerspoon.org/docs/hs.grid.html) | Mac | A grid per screen or resolution (`setGrid('12x3')`). Push a window one cell left, right, up or down, or onto the next screen at the edge. Make it one cell wider, thinner, taller or shorter. Snap a window onto the grid. |
| [yabai `--grid`](https://github.com/koekeishiya/yabai/wiki/Commands) | Mac | One command with the grid, start cell and size in cells: `rows:cols:x:y:width:height`. |
| [Moom](https://manytricks.com/moom/help/keyboard.html) | Mac | A keyboard grid: arrows pick the start cell, Shift and the arrows size it, Return places. |
| [gTile](https://github.com/gTile/gTile) | Linux, GNOME | A grid over the screen: arrows pick one corner, Shift and the arrows the other. Space changes between grids. |
| [FancyZones](https://learn.microsoft.com/windows/powertoys/fancyzones) | Windows | Zones you draw. Win and an arrow moves a window to the next zone. Win+Ctrl+Alt and an arrow stretches it across zones. |

**Sketch.** Names and fields are placeholders.

- **The grid:** a setting, `windows.grid`, such as
  `{ "columns": 12, "rows": 3 }`. Cells split the usable area equally.
  Edges round and the gap applies as for
  [Place Window](#windowplace), so a window in cells looks the same as
  one placed with a frame.
- **Size:** set the window to a number of cells, such as 3 wide and 1
  high. It keeps its top left cell, and moves left or up only if it
  would pass the edge. Or grow or shrink by one cell on one side.
- **Move:** push the window one cell in a direction. A second form moves
  by its own width or height, so a quarter-wide window steps a quarter
  at a time.
- **Snap:** a window that isn't on the grid moves to the nearest cells
  before it's sized or moved.
- **Keys:** a size menu, `CAG+W <width> <height>`, and direct bindings
  for the moves, a modifier with J, K, L and ;, as Focus Window and Swap
  Window use. A held menu doesn't suit the moves: it chooses once, when
  you let go, so it can't push a window several times.

**Which grid.** The grid decides which sizes you can have:

| Columns | Gives | Lacks |
| --- | --- | --- |
| 8 | halves, quarters, eighths, 3/8, 5/8 | thirds, sixths |
| 12 | halves, thirds, quarters, sixths | eighths |
| 24 | all of these | short moves: a quarter-wide window takes 6 single-cell pushes to move a quarter |

A grid that holds several fractions needs the least common multiple of
their parts. Across, 1/2, 1/3, 1/4, 1/6 and 1/8 need 24 columns. Down,
halves and thirds need only 6 rows; 12 rows adds quarters. On a 57-inch
32:9 display at 3840 by 1080 points, a 24 by 6 grid has cells of about
160 by 175 points.

**Smallest size.** A fine grid is easier with a smallest size per app,
in cells, such as 3 by 2 (an eighth wide, a third high), perhaps in
[`windows.apps`](#windows). Most apps aren't useful smaller than a
half on a laptop, so the smallest size could depend on the display.
Moves by the window's own size matter more on a fine grid.

**Compared with Place on Grid:**

| | Place on Grid | Grid Moves |
| --- | --- | --- |
| Thinking | Where, as a count and a place in it | How big, then which way |
| Keys to a given spot | 2 to 5 digits, always | A size, then a move for each cell or window width |
| Grids | Any, picked each time | One per display, set in the config |
| Small changes | Place it again | One push or one grow |
| Windows not on a grid | Placed exactly | Snapped first |
| Possible today | Yes, as plain menus and frames | Size only, with `window.resize` and `to`, without the gap and without snapping. A move by points fits one display only. |

**Not decided:**

1. One grid for every display, or one for each display or resolution,
   as `hs.grid` does.
2. Widths above 9 cells: a 12-column grid needs keys past `9` in the
   size menu.
3. What a push does at the edge: nothing, the alert sound, or the next
   display.
4. Whether both models could live together: Place on Grid for getting
   to a spot, Grid Moves for small changes after.

**Arranging automatically.** What the tools above do, as of 2026-11-27:

| Tool | Arranges on its own |
| --- | --- |
| yabai | Yes. A tiling manager: the `bsp` layout tiles each new window, with `stack` and `float` as other layouts, set per Space. Rules per app can float an app (`manage=off`). Directional focus doesn't appear to cover floating windows. Some features need System Integrity Protection partly off. |
| Hammerspoon | With scripts. `hs.window.layout` keeps rules in force, and its `tile` and `fit` rules spread windows over an area. `hs.layout.apply` places apps once. Not tied to `hs.grid` cells. |
| FancyZones | Partly. Puts an app back in its zone when it opens, and keeps zone numbers when the layout changes. Doesn't fill zones. |
| Moom | Saved snapshots of a whole arrangement, applied by click or shortcut. |
| AeroSpace, komorebi, GlazeWM, i3 | Fully tiling. You never pick sizes. |

Arrange Windows already lays out the windows used last in equal columns
or rows. Grid Moves could extend it: arrange in cells, and keep each
app's smallest size. No tool above was found to combine a fixed grid,
a smallest size per app in cells, and arranging on its own.

**Next:** try `hs.grid` with a 24x6 grid on a 57-inch 32:9 display before
choosing between the two.

**yabai with Unnamed Manager** was considered: yabai tiles, and Unnamed
Manager does focus (which counts floating windows), Spaces and menus,
calling yabai with Run Command. yabai's Space commands need System
Integrity Protection partly off; its tiling doesn't. Set aside, since
tiling shrinks windows past 2 to 4 (see What We Need), and yabai is Mac
only.

### Arrange with Limits

Recorded to compare. An arranger that moves windows and resizes them
only within limits, instead of dividing the screen among them.

- **Sizes:** each window has a preferred size (its size now, or a
  default for its app), and each app can have a smallest and a largest
  size, perhaps in [`windows.apps`](#windows). Limits are in points, not
  fractions, so one rule works on every display.
- **Which windows get the screen:** the ones used last, from the focus
  history, as for Arrange Windows today.
- **Move as little as possible:** a window that already fits stays.
- **When windows don't fit,** the choices are: shrink them (what tilers
  do, which we're avoiding), scroll them off the edge (niri, PaperWM,
  Paneru), stack them in one place, or send the oldest to another Space
  or minimize them. Scrolling off the edge fights the OS: macOS keeps
  windows on screen, and the spec says a window never ends off screen.
- **When it runs:** when you press a key, at first. Arranging every
  time a window opens is where tilers start to fight you.
- **Where it lives:** Unnamed Manager already has the focus history,
  Arrange Windows, rules per app and the Space actions, so this could
  be a mode of Arrange Windows rather than a new app.

### Columns

The current favorite, still being thought out. Tidied on 2026-11-27:
this is the design as it stands, with what it replaced at the end.

**The parts.**

- **A Space is a workspace.** It holds a row of **columns** across the
  usable area of its display.
- **A column** has a width and holds one or more windows, top to
  bottom.
- **A shelf** is a strip at an edge that holds what doesn't fit: columns
  at the left or right of the screen, windows at the top or bottom of a
  column. What's on a shelf stays a normal window, with at least 40
  points of its title bar on screen, so nothing is hidden and the spec's
  rule for title bars holds.
- **The grid** sets the steps for sizing with keys, such as 24 by 6 on
  the wide display and 6 by 6 on the laptop.

**Decided:**

1. **New windows float.** A window opens where its app puts it and
   isn't arranged. A short-lived window (a terminal for one command) is
   used and closed, and focus goes back to the window used before. A
   window that stays gets sent to a column with a key. Dialogs always
   float. This is the reverse of a tiler, where every new window is
   arranged, and it keeps the layout from fighting the OS: it only
   touches windows you gave it.
2. **A column lays out its windows.** Once a window is in a column, the
   column decides its frame:
   - **Width:** the column's, unless that's past the window's largest
     width; then the window just doesn't grow.
   - **Height:** from the column's settings.
   - **Place:** from what the column shows now.
   - A window that refuses a size keeps the size it takes. The layout
     doesn't try again.
3. **Widths.** A new column takes the width of the first window sent to
   it, rounded up to the grid. After that, the column is resized as a
   whole, and its windows with it. A column is never narrower than the
   grid steps that hold the largest smallest width among its windows.
   Widths are kept as fractions of the usable width, so a column keeps
   its share when the display changes, and snaps to that display's grid
   the next time it's sized with a key.
4. **The grid is set for each display.** It's only for sizing with keys.
5. **Shown, then shelved.** Windows in a column are shown top to bottom
   while they fit; one window gets the column's whole height. When all
   fit, there's no shelf. When they don't, the rest go to the column's
   shelf, each at the column's width. Columns in a Space work the same
   way, across.
6. **Three kinds of shelf,** for columns and for windows alike:
   - **No shelf.** Everything must fit on the display. Sending a window
     or a column that won't fit is an error, and so is resizing a
     column so that it, or another one, would go past the display's
     edge. Errors flash or sound as other actions' errors do. With
     several displays next to each other, this keeps every window on
     its own display.
   - **One shelf** (left or right for columns, top or bottom for
     windows). Clicking the front one on the shelf brings it in at the
     shelf's edge. What's shown shifts away from the shelf, and the
     farthest goes to the back of the shelf. Clicking on and on turns
     through all of them, like a wheel, back to the layout you started
     with.
   - **Two shelves** (left and right, or top and bottom). The same, but
     the shelf you click decides the way, and what leaves goes to the
     front of the other shelf. It works as scrolling does.
7. **One rule for shifting.** What's already shown stays, pushed away
   from the shelf, while there's room. What no longer fits, the
   farthest first, goes onto the shelf in order. There's no "current"
   window or column to keep, only what is shown. Stepping moves windows
   and doesn't resize them, since a window keeps its height.
8. **The kind of shelf is set for each display,** so two displays next
   to each other can have their shelves on the far sides. A top shelf
   keeps title bars at the top of the column, with the windows behind
   the ones shown, since a title bar can't go above the screen.
9. **What's on a shelf overlaps,** as a window behind another does
   today. Only the front one's title bar shows, and clicking it shifts
   as in 6.
10. **Nothing rearranges on its own.** When a column's last window
    closes, the column stays, empty. Removing a column is something you
    do, and other columns don't spread into its space unless you ask.
11. **No tabs. A bar** along one edge of a column with more than one
    window, drawn by the app in a panel of its own, like a scroll bar:
    one part for each window, with the shown ones marked. A column with
    one window has no bar. Keys step forward and back through the
    column's windows, as clicking the shelf does.

**Not decided:**

1. **Keys:** send a window to a column (by number, left, right, a new
   one), take it out so it floats again, shift as a shelf click does,
   and size a column on the grid.
2. **Heights:** a window's own height, a share of the column from the
   grid's rows, or the column's whole height less the shelf.
3. **A window that fits only in part** above or below the shelf: it goes
   to the shelf, or it shows in part, with the shelf over it.
4. **Where a sent window goes:** brought in at the shelf's edge, as a
   click does, or at the far end.
5. **A window closing in a column:** the windows after it move up, or
   the space stays until you ask (as in 10).
6. **Where a new column goes:** next to the focused column, at the
   first free place, or where the window already is.
7. **A window dragged out of its column** with the mouse: it floats
   again, or it goes back. Going back is fighting you.
8. **Memory:** whether a Space keeps its columns after the app quits.
   The config is the only file the app keeps.
9. **Both platforms:** Spaces on the Mac and virtual desktops on Windows
   have different limits, and apps report smallest sizes differently.
   The bar's panel has to follow its Space on both.
10. **The laptop's widths:** fractions of a narrow display are narrow,
    so the smallest widths of its windows mostly decide the columns
    there.

**Ideas to weigh:**

- **The bar in the gap.** With a gap of 8 points, a thin bar fits
  between columns without covering any window.
- **The column's windows as a list.** A held [list](#lists) of the
  windows in the focused column could stand in for a dock, with the bar
  as the sign that's always there.

**Walkthrough** with the rules above. The wide display is 3840 points
across, so a 24th is 160 points, with one shelf on the right for
columns and one at the bottom for windows. The laptop is taken as 1512
points across (a 14-inch MacBook Pro's default).

1. On an empty Space, Zed opens 1200 points wide and floats. Sent to a
   new column, the column is 8/24 (1280 points), and Zed widens to fit.
2. Chrome opens 1600 points wide. Sent to a new column, that column is
   10/24. Together, 18/24 is used.
3. A terminal opens, floating, runs a command and closes. Focus goes
   back to Chrome. Nothing moved.
4. Another terminal is sent to Zed's column and takes its width. If both
   fit in the column's height, both show; if not, one goes to the
   column's shelf at the bottom and the bar appears.
5. Zed's column grows to 10/24. Chrome's column moves right; it isn't
   resized. Now 20/24 is used.
6. Zed's column grows to 16/24, which with Chrome's 10/24 is 26/24.
   Chrome's column goes to the right shelf. With no shelf, the resize is
   an error instead.
7. Clicking Chrome's title bar on the shelf brings its column in at the
   right edge. Zed's column no longer fits and goes to the back of the
   shelf. Clicking again brings Zed's back.
8. The wide display is unplugged. The laptop's own shelf settings
   apply. Zed's 10/24 there is 630 points, so Zed's smallest width
   decides it, and fewer columns are shown, the rest on the shelf.
9. Where macOS puts the windows of an unplugged display's Spaces needs a
   test on the Mac. Windows does the same with virtual desktops in its
   own way.
10. A window that refuses the column's width: wider, it makes the column
    grow (3); narrower, it keeps its size, at the column's left edge or
    in the middle.
11. Zed goes full screen on the Mac. It gets a Space of its own and
    leaves the column; on its way back it floats, or it goes back to
    its column.

**Considered and replaced:**

- **Rows or one at a time, with the others behind the front window.**
  Replaced by shown-then-shelved (5), which needs no second mode.
- **Tabs.** No OS draws tabs on other apps' windows. Replaced by the
  bar (11).
- **Columns that scroll up and down, hiding what's outside.** A title
  bar can't go above the screen, and no OS hides one window of an app
  (the Mac hides whole apps; Windows hides the window and its taskbar
  button; minimizing animates and fills the Dock or taskbar). Replaced
  by shelves, which hide nothing. AeroSpace's way, leaving windows in a
  corner of the screen, is the same idea as a shelf.
- **The clicked window goes to the top of its column.** Replaced by one
  rule for shifting (7).
- **A layout for each display shape.** Not needed: shelves let one
  layout work on the laptop and the wide display.

**Prior art:** niri (Linux) puts windows in columns on a strip that
scrolls, and new windows never resize the others; columns can show
their windows as tabs. PaperWM (GNOME) and Paneru (Mac) scroll the same
way. i3 and sway have tabbed and stacked containers. yabai can stack
windows in one tile. Floating new windows, and shelves instead of going
off screen, are what set this apart.

### Decided in spec v2

1. ~~How does a config name an app on both platforms?~~ One config file
   per platform (see [The file](#the-file)).
2. ~~One `version` or two?~~ One. See [Versions](#versions).
3. ~~What does an app do with a type it can't do?~~ It's an error on that
   platform, `unsupported-action`. With one config per platform, a file
   never needs the other platform's actions.

### Decided in spec v4

1. ~~Windows app identity?~~ `exe` (a file name or full path) or `appId`
   (an AppUserModelID), exactly one. See [Actions](#actions).
2. ~~Windows reserved chords?~~ `Cmd+L` and Ctrl+Alt+Delete are reserved.
   The Office shortcuts are a warning, `office-shortcut`, since the app
   catches them first while it runs. See [Reserved chords](#reserved-chords).
3. ~~Windows icons?~~ A Segoe Fluent Icons code point (`U+E756`) or a file
   path. Names would need a table both apps ship; code points don't.

### Open questions

None.

# Spec changelog

Tags are `spec-v<N>`. Each app repo pins its `spec/` submodule to one.

## spec-v16

- **Quit on close** (Mac only): `windows.quitOnClose`, `false` by
  default, quits an app when its last window closes. It never quits
  Finder or the Mac app itself, an app that hasn't shown a window, a menu
  bar or background app, or an app with a minimized window, one on
  another Space, or one that's hidden.
- **`windows.quitOnCloseDelayMs`** (Mac only), `500` by default: how long
  after a window closes the app's windows are counted. Negative is
  `negative-delay`.
- **`quit`**, a fourth rule in `windows.apps`, `true` by default.
  `false` keeps the app open with Quit on close on. It also keeps apps
  inside that app's bundle. The Mac app writes all four rules for each
  listed app.
- On Windows, `quitOnClose`, `quitOnCloseDelayMs` and `quit` are
  `other-platform-field`.
- `version` stays `1`.
- `config.schema.json` has `windows.quitOnClose`,
  `windows.quitOnCloseDelayMs` and the `quit` rule. The
  title says spec v16.
- 126 fixtures (23 valid, 103 invalid). `valid/quit-on-close.json`,
  `invalid/quit-on-close-type.json`,
  `invalid/quit-on-close-negative-delay.json` and
  `invalid/windows-app-quit-type.json` (Mac only),
  `invalid/win-quit-on-close.json`, `invalid/win-quit-on-close-delay.json`
  and `invalid/win-app-quit.json` (Windows only) are new.
- Not yet supported by the Windows app, which pins spec-v4.

## spec-v15

- **`windows.apps`**, rules for each app's windows, keyed by app id:
  `focus`, `swap` and `arrange`, each `true` by default. `false` leaves
  the app's windows out of Focus Window, Swap Window or Arrange Windows.
  The rules are separate, so Calculator can take part in Focus Window
  and stay out of Arrange Windows.
- **Swap Window** reads `swap`, not `focus`, and never moves a window of
  an app with `"swap": false`, as the window swapped with or as the
  focused one. From such a window it fails ("Calculator is left out of
  Swap Window.").
- **`windows.exclude` is gone.** No released app read it, so nothing
  loads it in its place: a file with it is `unknown-field`.
- A rule that isn't one of the three is `unknown-field`, one that isn't
  a boolean is `wrong-type`, and on Windows a blank app id is
  `bad-app-id`.
- `version` stays `1`.
- `config.schema.json` has `windows.apps` in place of `windows.exclude`.
  The title says spec v15.
- 119 fixtures (22 valid, 97 invalid). `valid/window-apps.json`,
  `invalid/windows-apps-type.json`, `invalid/windows-app-rule-type.json`
  and `invalid/windows-app-unknown-rule.json` replace the
  `windows-exclude` ones. `invalid/windows-exclude.json` and
  `invalid/win-blank-window-app.json` (Windows only) are new.
- Not yet supported by the Windows app, which pins spec-v4.

## spec-v14

- **`windows.exclude` is an object with two lists.** `focus` holds the
  apps Focus Window and Swap Window never pick, as the v13 list did.
  `arrange`, new, holds the apps whose windows Arrange Windows never
  counts or moves, so an app such as Calculator can still be focused
  without being arranged. The lists are separate. An array, the v13
  form, still loads as `focus`, and an app writes it back that way.
  `exclude` with another field is `unknown-field`.
- **Arrange Windows** leaves out only `exclude.arrange`. Under v13 its
  windows were Focus Window's candidates, which left out `exclude`. When
  the focused window's app is left out, its display is still the one
  arranged, and the window in front there counts as focused.
- **Clearer wording, same behavior:** `recent` falls back to the window
  in front whenever the history has no window that counts, not only when
  it's empty. With focus on an excluded app's window, `next` and
  `previous` go to the first and the last candidate.
- No field changed meaning for a file that loads under v13, so `version`
  stays `1`.
- `config.schema.json` takes either form for `exclude`. The title says
  spec v14.
- 117 fixtures (22 valid, 95 invalid): `valid/window-exclude.json`,
  `invalid/windows-exclude-arrange-type.json` and
  `invalid/windows-exclude-unknown-field.json` are new.
- Not yet supported by the Windows app, which pins spec-v4.

## spec-v13

- **`recent`**, a new `target` for Focus Window and Swap Window: the
  window used before the focused one, so two presses go back and forth
  between two windows. It reads a focus history the app keeps while it
  runs, so when the window used before was closed or minimized, it's the
  one before that. Unlike the other targets, it can pick a window you
  can't see. `scope` still applies. With none, the action fails ("No
  window used before.").
- **Across Spaces:** on the Mac, Focus Window's `recent` can go to a
  window on another Space, and the screen moves there. On Windows, to a
  window on another virtual desktop when the app can bring it forward.
  Swap Window's `recent` stays on the current Space or desktop.
- **Built-in actions** `window-focus-recent` and `window-swap-recent`,
  from the naming rule, with no new rule.
- **`windows.exclude`:** app ids whose windows Focus Window and Swap
  Window never pick, for any `target`. Empty by default, so nothing
  changes for a config without it. A value that isn't an array of
  strings is `wrong-type`; on Windows, a blank one is `bad-app-id`.
- No field changed meaning, so `version` stays `1`.
- `config.schema.json` lists `recent` for `target` and `direction`, and
  the setting. The title says spec v13.
- 114 fixtures (21 valid, 93 invalid): `valid/window-recent.json` and
  `invalid/windows-exclude-type.json` are new.
- Not yet supported by the Windows app, which pins spec-v4.

## spec-v12

- **Only windows you can see:** Focus Window and Swap Window skip a
  window whose visible part, its frame minus the windows in front of it,
  doesn't hold a square `windows.minVisible` points on a side. That goes
  for every `target`: directions, `next`, `previous`, `first`, `last`
  and numbers.
- **Directions use edges:** a direction looks at the visible part past
  the focused window's center. Windows in line with the focused window
  come first, nearest edge first, with gaps within
  `windows.tieTolerance` points counting as the same. The 45° rule on
  centers is now the fallback for windows not in line. A window inside a
  larger one goes to the larger one in every direction.
- **Order changed:** by display, then in rows as text reads, by the top
  left of each visible part, with tops within `windows.rowTolerance`
  points in one row. Before, by center, left to right and then top to
  bottom, so in a grid `2` was the window below the first one and is now
  the one beside it.
- **Three settings** in `windows`: `minVisible` (default 20),
  `rowTolerance` (default 20) and `tieTolerance` (default 2), each a
  number from 0. A negative one is `negative-window-distance`.
- No field changed meaning, so `version` stays `1`.
- `config.schema.json` lists the three settings. The title says spec v12.
- 112 fixtures (20 valid, 92 invalid): `valid/window-focus-distances.json`
  and `invalid/negative-window-distance.json` are new.

Support: the Mac app from the commit that pins `spec/` to `spec-v12`. Not
yet in the Windows app: it pins `spec-v4`.

## spec-v11

- **Built-in actions:** a binding can name an action that isn't in
  `actions`, such as `window-place-left-half`, `window-focus-left`,
  `window-swap-2`, `window-fill` or `help`. The names follow one rule:
  the type with hyphens for its dots, then a value of its main field in
  lowercase words joined by hyphens, with every other field at its
  default.
- **Overrides:** an action in `actions` with a built-in name replaces the
  built-in everywhere the config names it, with any type, such as a
  window manager's own command. It isn't a warning.
- **Focus Window's `direction` is now `target`,** which also takes a
  number from 1, as Swap Window's does. `direction` still loads, and an
  app keeps the name the file uses when it saves. Both is
  `conflicting-fields`.
- `undefined-action` now means neither an id in `actions` nor a built-in
  name. `bad-window-value` also covers Focus Window's `target`.
- **Planned:** inline actions in a binding, and built-in names for Move
  Window once its spots have names.
- `config.schema.json`: Focus Window has `target`, with `direction` as its
  old name, and an entry's `action` mentions built-in actions. The title
  says spec v11.
- 110 fixtures (19 valid, 91 invalid): `valid/builtin-actions.json`,
  `valid/builtin-override.json`, `valid/window-focus-target.json`, seven
  `invalid/builtin-*.json` and two `invalid/window-focus-target-*.json`
  are new. `valid/window-focus-ends.json` uses `target`, and
  `valid/window-actions.json` keeps `direction`, the old name.

Support: the Mac app from the commit that pins `spec/` to `spec-v11`. Not
yet in the Windows app: it pins `spec-v4`.

## spec-v10

- **Swap Window:** a new action, `window.swap`, trades place and size
  with another window. The focused window keeps focus. Its `target` is
  `left`, `right`, `up`, `down`, `next`, `previous`, `first`, `last`, or a
  number from 1 in Focus Window's order.
- **`scope` for Focus Window and Swap Window:** `all` (the default, as
  before) or `display`, which keeps them to the focused window's display.
- **The alert sound:** when Focus Window or Swap Window finds no window,
  the app plays the platform's alert sound, when `ui.errorBell` is on.
  Before, `errorBell` was only for a key that isn't bound.
- `bad-window-value` now also covers `scope` and `target`. A `target`
  that's neither a string nor an integer is `wrong-type`.
- `config.schema.json` lists the action and the field, and says spec v10
  in its title.
- 98 fixtures (16 valid, 82 invalid): `valid/window-swap.json`, six
  `invalid/window-swap-*.json` and `invalid/window-focus-scope.json` are
  new.

Support: the Mac app from the commit that pins `spec/` to `spec-v10`. Not
yet in the Windows app: it pins `spec-v4`.

## spec-v9

- **Arrange Windows keeps the focused window where it is.** When the
  cells don't divide evenly over the groups, the focused window stays in
  the column or row its center is in and takes the spare room there, up
  to the whole group. Before, it moved to a roomier cell on the right or
  at the top. With five windows in three columns, it now fills its own
  column.
- **The other windows** keep their order in the rest of the grid. With
  more spare room than the focused window's group has, other groups
  with a single window are on the right or at the top.
- **A group with the focused window** may hold cells of two sizes: with
  eight windows in three columns, the focused window has two thirds of
  its column.
- No change to the file format: the fields, the schema's rules and the
  90 fixtures are the same. `config.schema.json` says spec v9 in its
  title.

Support: the Mac app from the commit that pins `spec/` to `spec-v9`. Not
yet in the Windows app: it pins `spec-v4`.

## spec-v8

- **Arrange Windows:** a new action, `window.arrange`, lays out the
  windows used last, on the focused window's display, in a grid of
  columns or rows, with the gap. Its fields are `count` (2 to 8, or
  `"all"`), `groups` and `layout` (`auto`, `columns`, `rows`).
- **Two settings:** `windows.maxColumns` and `windows.maxRows`, from 1 to
  8, both 2 by default. They limit the grid the action picks by itself.
- **A new rule,** `bad-window-limit`, for a setting out of that range.
  `bad-window-value` now also covers `layout`, `count` and `groups`.
- `config.schema.json` lists the action and the settings.
- 90 fixtures (15 valid, 75 invalid): `valid/window-arrange.json`, five
  `invalid/window-arrange-*.json` and `invalid/window-limit.json` are new.

Support: the Mac app from the commit that pins `spec/` to `spec-v8`. Not
yet in the Windows app: it pins `spec-v4`.

## spec-v7

- **Fill:** `window.maximize` is now `window.fill`, with the UI name
  Fill. Its `toggle` field is the same.
- **Fill leaves the gap:** the window ends one `windows.gap` in from
  every edge of the usable area, as with Place Window. Before, it went
  edge to edge. With a gap of `0` it still does.
- **The old name still loads:** `window.maximize` is read as
  `window.fill`, gap included. An app keeps the name a file uses when it
  saves, and writes `window.fill` for a new action.
- `config.schema.json` lists `window.fill`, and `window.maximize` as its
  old name.
- 83 fixtures (14 valid, 69 invalid): `valid/window-fill.json` and
  `invalid/window-fill-toggle-type.json` are new.

Support: the Mac app from the commit that pins `spec/` to `spec-v7`. Not
yet in the Windows app: it pins `spec-v4`.

## spec-v6

- **First and Last:** `window.focus` takes two new values for
  `direction`, `first` and `last`: the leftmost and the rightmost window.
  They don't need a focused window, and on a window that's already there
  nothing happens.
- **Order changed:** the order `next`, `previous`, `first` and `last`
  share is now by display, left to right, then by window center, left to
  right and then top to bottom. Before, top to bottom came before left to
  right, so stacked windows now come in a different order.
- `config.schema.json` lists `first` and `last`.
- 81 fixtures (13 valid, 68 invalid): `valid/window-focus-ends.json` is
  new.

Support: the Mac app from the commit that pins `spec/` to `spec-v6`. Not
yet in the Windows app: it pins `spec-v4`.

## spec-v5

- **Focus Window with no focused window:** `window.focus` no longer fails
  when the app in front has no focused window, or no app is in front.
  Every `direction` then focuses the candidate window in front of the
  others. With no candidate it fails with "No window to focus."
- No change to the format, the schema or the fixtures.

Support: the Mac app from the commit that pins `spec/` to `spec-v5`. Not
yet in the Windows app: it pins `spec-v4`.

## spec-v4

- **Windows app names:** Use Application and contexts name an app on
  Windows by `exe` (a file name such as `Code.exe`, or a full path) or
  `appId` (an AppUserModelID), exactly one per action.
- **Windows reserved chords:** `Cmd+L` (Win+L), and `Delete` with Ctrl
  and Alt held. The Office shortcuts (Hyper with W, X, P, O, T, D, N, Y,
  L) are a warning, not reserved.
- **Windows icons:** a Segoe Fluent Icons code point (`U+E756`) or a
  file path.
- **New rules, Windows only:** `other-platform-field` (`bundleId` on
  Windows), `bad-app-id`, `bad-icon`, and the warning `office-shortcut`.
  `conflicting-fields` and `missing-field` now cover `exe` and `appId`.
- `config.schema.json` has an `exe` and an `appId` variant of
  `application`.
- **Shared fixtures are now platform-neutral:** the fixtures that used
  `bundleId` only to have an action (35 of them) use `exec` instead, and
  break the same rule as before. `minimal.json` and `warnings.json` bind
  `Meh+T` and `Meh+W` instead of `Hyper+T` and `Hyper+W`, which are Office
  shortcuts on Windows. `valid/spec-example.json`,
  `valid/all-actions.json` and `invalid/bad-app-mode.json` are Mac only;
  `invalid/win-bad-app-mode.json` covers `bad-app-mode` on Windows.
- 80 fixtures (12 valid, 68 invalid), 11 new and Windows only
  (`win-*` and `reserved-chord-win-*`).

Support: the Windows app from the commit that pins `spec/` to `spec-v4`,
and the Mac app from 25a0017, which moves its `spec/` to `spec-v4`. Its
generated schema has the `exe` and `appId` variants (the Mac still reads
only `bundleId`), and its rule-id and writer tests skip the Windows-only
rules and fixtures. Its `swift test` run on a Mac is still to do.

## spec-v3

- **Gap:** a new optional top-level object, `windows`, with `gap` (points,
  default 8). Place Window leaves it between a window and the usable
  area's edges, and between windows placed side by side. Move, Resize and
  Maximize don't use it.
- **New rule:** `negative-gap`.
- `config.schema.json` has the `windows` object.
- 69 fixtures (10 valid, 59 invalid): two valid ones with a gap of 12 and
  of 0, and invalid ones for a negative gap, an unknown field in
  `windows` and a gap that isn't a number.

Support: the Mac app from the commit that moves its `spec/` to
`spec-v3`. The Windows app doesn't exist yet.

## spec-v2

- **Window actions:** `window.maximize`, `window.minimize`, `window.place`,
  `window.move`, `window.resize` and `window.focus`, with every field, its
  range and its default, and the geometry both apps share (coordinates,
  the usable area, fractions, rounding edges, windows that won't resize).
- **New rules:** `conflicting-fields`, `bad-fraction`, `bad-window-value`
  and `unsupported-action`.
- **Decided:** one config file per platform; `version` stays `1`, since v2
  only adds; an action a platform can't do is `unsupported-action`.
- **Location:** `~/.config/unnamed-manager/config.json`, moved up from the
  proposals.
- `config.schema.json` has a variant per window action.
- 64 fixtures (8 valid, 56 invalid): `valid/window-actions.json` uses
  every window action and field, and 18 invalid ones cover the new rules
  and the window fields' missing, unknown and wrong-type cases.
  `unsupported-action` has no fixture until a type exists on one platform
  only.

Support: the Mac app marks the window fixtures pending until S-062 to
S-069 build the actions. The Windows app doesn't exist yet.

## spec-v1

- The Unnamed Launcher 0.2.0 config format, written down for both
  platforms: fields, key syntax, the modifier table for Mac and Windows,
  actions, contexts.
- Every validation rule has an id. Windows reserved chords are still open.
- `config.schema.json` generated from the launcher's `ConfigSchema.swift`.
- 45 fixtures (7 valid, 38 invalid) with `fixtures/manifest.json`.
- Window actions and the other v2 changes are proposals in
  `CONFIG-SPEC.md`, Planned for v2.

# Spec changelog

Tags are `spec-v<N>`. Each app repo pins its `spec/` submodule to one.

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

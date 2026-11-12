# Spec changelog

Tags are `spec-v<N>`. Each app repo pins its `spec/` submodule to one.

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

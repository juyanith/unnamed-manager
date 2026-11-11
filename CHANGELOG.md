# Spec changelog

Tags are `spec-v<N>`. Each app repo pins its `spec/` submodule to one.

## spec-v1 (draft, not tagged)

- The Unnamed Launcher 0.2.0 config format, written down for both
  platforms: fields, key syntax, the modifier table for Mac and Windows,
  actions, contexts.
- Every validation rule has an id. Windows reserved chords are still open.
- `config.schema.json` generated from the launcher's `ConfigSchema.swift`.
- 45 fixtures (7 valid, 38 invalid) with `fixtures/manifest.json`.
- Window actions and the other v2 changes are proposals in
  `CONFIG-SPEC.md`, Planned for v2.

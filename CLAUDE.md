# Unnamed Manager spec

The shared contract for the Mac app (`unnamed-manager-mac`, BAMA) and the
Windows app (`unnamed-manager-windows`, BAWA). No app code lives here.
README.md says what each file is.

## Rules

- `CONFIG-SPEC.md` is the source of truth. The schema and fixtures follow
  it, never the other way round.
- Every rule in Validation has an id. A new rule gets an id, a row in the
  table, and at least one fixture in `fixtures/manifest.json`.
- An invalid fixture breaks exactly one rule. A rule that only one
  platform has gets `"platforms"` in the manifest.
- Write subjects the way the apps write key paths: `Hyper+Space A`, group
  names for exact groups, canonical key names.
- Keep `fixtures/manifest.json` valid JSON and its `file` paths real:
  `python3 -c "import json; json.load(open('fixtures/manifest.json'))"`.
- Proposals go under "Planned for v2" until decided. Only settled rules
  move up into the spec, with fixtures.
- A spec change is done when both apps' fixture tests are run against it,
  or the change is noted as not yet supported in CHANGELOG.md.
- Use the words in GLOSSARY.md. No em dashes. Plain words, short sentences.

## Releasing a spec version

1. Update CHANGELOG.md.
2. Commit, then `git tag spec-vN`.
3. In each app repo: `git -C spec checkout spec-vN`, run the tests, commit.

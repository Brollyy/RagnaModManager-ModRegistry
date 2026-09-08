# RagnaModManager official mod registry

The registry is a reviewed allow-list: a mod is trusted because a maintainer
has reviewed and merged its entry, not because somebody mentioned it in an
Issue.

The manager consumes the published `index.json` from:

https://github.com/Brollyy/RagnaModManager-ModRegistry

## Request a mod

Open the **Mod request** Issue template and include:

- mod name, author, and public source repository;
- what the mod does and whether it is maintained;
- the Ragnarock game version(s) tested;
- the mod type: Lua, DLL, pak, config, or loose file;
- a link to the exact GitHub Release containing the `.rmod` asset;
- known dependencies, conflicts, limitations, and permissions.

Do not attach arbitrary binaries to an Issue. The maintainer needs a public
source or release that can be inspected and a reproducible package asset.

## Mod author submission process

1. Develop and test the mod against a clean Ragnarock installation.
2. Create a valid `.rmod` package containing a root `manifest.json` and only
   the files declared by that manifest.
3. Use a stable lowercase ID, semantic version, clear author information, and
   accurate `requires`, `dependencies`, `conflicts`, `affects`, and `hooks`.
4. Publish the package as an immutable GitHub Release asset named with its
   version, for example `better-hit-feedback-1.0.0.rmod`.
5. Open a registry Issue or pull request with the release URL and test details.

Maintainers inspect the archive, validate all paths and manifest declarations,
test installation and removal, calculate the SHA-256 checksum, and add the
release to `index.json`. A package is not trusted until that catalog change is
reviewed and merged.

## Package rules

- `schemaVersion` must be `1` and `game` must be `ragnarock`.
- IDs must be lowercase letters, numbers, dots, underscores, or hyphens.
- Every file must be declared in `manifest.json`; paths must be relative and
  must not escape the archive or installation root.
- Supported file types are `ue4ss-lua`, `ue4ss-dll`, `pak`, `config`, and
  `loose-file`.
- DLLs require extra review because they execute native code in the game
  process. Explain their purpose and provide source when possible.
- Do not replace or modify the game executable, Steam files, manager files, or
  unrelated user files.
- Do not include telemetry, credential collection, miners, DRM bypasses, or
  unrelated bundled software.
- Declare runtime requirements, dependencies, conflicts, affected assets, and
  UE4SS hooks honestly.
- Releases must be reproducible enough for the maintainer to verify the
  published checksum. Never silently replace an asset at an existing version.
- Respect the Ragnarock developers' terms, the licenses of included assets, and
  the licenses of dependencies.

## Package manifest format

Every `.rmod` archive must contain a `manifest.json` at its root. The manifest
must include `schemaVersion` (`1`), `id`, `name`, `version`, `game`
(`"ragnarock"`), and a non-empty `files` array. `id` and `version` must match
the catalog entry and the package filename metadata.

The remaining manifest fields are:

- `author`: optional author or maintainer name.
- `description`: optional user-facing description.
- `requires`: optional object mapping manager/runtime requirements to version
  requirements, for example `{ "ragnamodmanager": ">=1.1.0" }`.
- `dependencies`: optional object mapping mod IDs to version requirements. These
  are runtime mod dependencies and should agree with the catalog entry.
- `conflicts`: optional array of mod IDs that cannot be enabled together with
  this mod. A declared conflict blocks deployment.
- `affects`: optional array of stable target identifiers, such as an asset,
  feature, or shared game area. Two enabled mods declaring the same target are
  treated as conflicting unless the deployment rules allow it.
- `hooks`: optional array of UE4SS hook names used by the mod.
- `files`: array of file declarations. Each declaration requires `type` and
  `source`; `target`, `modFolder`, and `loadOrder` are optional and depend on
  the file type.

For example:

```json
{
  "schemaVersion": 1,
  "id": "better-hit-feedback",
  "name": "Better Hit Feedback",
  "version": "1.0.0",
  "author": "Author",
  "game": "ragnarock",
  "description": "Improves hit feedback.",
  "requires": {
    "ragnamodmanager": ">=1.1.0"
  },
  "dependencies": {
    "example-library": ">=1.2.0"
  },
  "conflicts": ["other-hit-feedback"],
  "affects": ["results-screen"],
  "hooks": ["ExampleHook"],
  "files": [
    {
      "type": "ue4ss-lua",
      "source": "scripts/better_hit_feedback.lua",
      "modFolder": "BetterHitFeedback",
      "loadOrder": 100
    }
  ]
}
```

## Catalog entry format

`index.json` is schema version `1`. The top-level object must contain:

- `schemaVersion`: required string, currently `"1"`.
- `repository`: required string, exactly `"official"`.
- `mods`: required array of catalog entries.

Each `mods` entry must contain:

- `id`: required lowercase package ID; it must match `manifest.json`.
- `name`: required display name.
- `author`: recommended author or maintainer name.
- `description`: recommended short description.
- `sourceUrl`: recommended public source repository URL.
- `license`: recommended SPDX identifier or an explicit statement that no
  license is declared.
- `dependencies`: optional object mapping another catalog `id` to a version
  requirement such as `">=0.2.1"`. Every dependency must have its own catalog
  entry and at least one release satisfying the requirement.
- `conflicts`: optional array of catalog `id` values that must not be enabled
  together with this mod. A catalog conflict is an installation/deployment
  constraint and should also be declared in the package manifest. Do not list
  arbitrary prose here; use catalog IDs so the manager can identify the
  conflicting installed mod.
- `releases`: required non-empty array of immutable package releases.

Each `releases` entry must contain:

- `version`: required semantic version; it must match the package manifest.
- `packageUrl`: required HTTPS URL for the immutable `.rmod` asset.
- `sha256`: required 64-character SHA-256 checksum of that exact asset.
- `publishedAt`: recommended ISO-8601 UTC publication timestamp.
- `changelog`: recommended release-note summary. Use an explicit statement if
  the upstream release has no notes.
- `sizeBytes`: optional package size in bytes.

The manager uses `dependencies` to automatically select and download missing
catalog packages before installing the requested mod. It selects the newest
release satisfying each requirement. Versions are compared using SemVer,
including prerelease identifiers. A catalog dependency is not a replacement
for the same dependency declaration in the package manifest; both must agree.

Versions are selected semantically by the manager; the package manifest ID and
version must match the catalog entry.

```json
{
  "schemaVersion": "1",
  "repository": "official",
  "mods": [
    {
      "id": "better-hit-feedback",
      "name": "Better Hit Feedback",
      "author": "Author",
      "description": "Improves hit feedback.",
      "sourceUrl": "https://github.com/example/better-hit-feedback",
      "license": "MIT",
      "dependencies": {
        "example-library": ">=1.2.0"
      },
      "conflicts": [
        "other-hit-feedback"
      ],
      "releases": [
        {
          "version": "1.0.0",
          "packageUrl": "https://github.com/example/mod/releases/download/v1.0.0/better-hit-feedback-1.0.0.rmod",
          "sha256": "64 hexadecimal characters",
          "publishedAt": "2026-09-05T00:00:00Z",
          "changelog": "Initial public release.",
          "sizeBytes": 123456
        }
      ]
    }
  ]
}
```

## Maintainer checklist

- Verify the source repository and author identity.
- Inspect the manifest and archive for unsafe paths and undeclared content.
- Review native DLL behavior and licenses especially carefully.
- Install, enable, disable, update, rollback, and remove the package.
- Test with the supported Ragnarock and RE-UE4SS versions.
- Confirm dependencies and conflicts are declared.
- Confirm the GitHub Release asset is immutable and calculate its SHA-256.
- Merge only the catalog entry for the reviewed release.
- Remove or mark releases that become incompatible or unsafe; document the
  reason in the associated Issue.

## Development resources

- [RagnaModManager](https://github.com/Brollyy/RagnaModManager)
- [RE-UE4SS documentation](https://docs.ue4ss.com/)
- [RE-UE4SS installation guide](https://docs.ue4ss.com/dev/installation-guide.html)
- [Creating a Lua mod with RE-UE4SS](https://docs.ue4ss.com/dev/guides/creating-a-lua-mod.html)
- [Creating a C++ mod with RE-UE4SS](https://docs.ue4ss.com/dev/guides/creating-a-c%2B%2B-mod.html)
- [RE-UE4SS source and examples](https://github.com/UE4SS-RE/RE-UE4SS)
- [Ragnarock modding notes](https://gist.github.com/UnforeseenOcean/1d3ea5a04f5952e8815c707d8943cb5b)
- [FModel Unreal archive explorer](https://github.com/4sval/FModel)
- [GitHub Releases documentation](https://docs.github.com/en/repositories/releasing-projects-on-github)

Follow the licenses and terms of each tool and source project. Asset
extraction tools are useful for understanding Unreal assets, but do not
redistribute copyrighted game assets without permission.

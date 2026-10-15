# Luna Base Patcher

File patch source for the Luna mobile client.

A patch replaces one or more files the app reads — **any file, in any folder**. Each patch is a ZIP of
replacement files plus a description of what each file replaces and what it becomes. Every step is
checked by SHA-256, so the app never writes something it is not sure about.

## Release

| Item | Value |
|---|---|
| **Tag** | `patches` |
| **Assets** | `<patch-id>.zip` |

## What a patch can replace

| Target | What it means | Example path |
|---|---|---|
| `apk` | A file packaged inside the APK (UI data, tables, images, script data) | `ui/pc-item-tooltip.json` |
| `client` | A file in the restored base client (the PC data tree) | `Data/BuffManagerConfig.dat`, `System/ServerList.bin` |

`path` keeps its full folder structure, so a file nested any number of folders deep is addressed with
the same relative path the app already uses. Nothing is inferred: a patch records the hash of the file
it replaces (`from_sha256`) and the hash of the new content (`to_sha256`).

How the app decides, per file:

| Current content | Action |
|---|---|
| hash equals `to_sha256` | already applied — skipped (running twice is harmless) |
| hash equals `from_sha256` | patch applied |
| anything else | left untouched, a warning is logged (the file is not what the patch was built against) |

## How a patch is published

From the project:

```
node Mobile/tools/publish-patch.mjs patches.example.json
```

The tool reads the spec, records the `from`/`to` hashes, builds the ZIP, rewrites
`Mobile/assets/patch.json` with the real link and hashes, uploads the ZIP, and verifies GitHub's
digest. Use `--dry-run` to build and inspect without uploading.

## Spec format

```json
{
  "id": "item-tooltips-2026-10",
  "version": 2,
  "entries": [
    { "path": "ui/pc-item-tooltip.json", "target": "apk", "source": "patches/item-tooltip.json" },
    { "path": "Data/BuffManagerConfig.dat", "target": "client", "source": "patches/BuffManagerConfig.dat" }
  ]
}
```

## Where patched files land on the device

Patched copies are kept in the app's own override folder, never written over the signed APK:

```
overrides/apk/<asset path>
overrides/client/<client path>
```

Every read the app makes is routed through the override first, so a patched file is used everywhere
the original would have been — Java UI, data tables, native renderer and audio alike.

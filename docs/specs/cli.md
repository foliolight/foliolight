# Specification: Command-Line Interface

Status: Draft v0.1.

## 1. Overview

`foliolight-cli.exe` is a console program that ships in the same zip as the GUI and uses the same `foliolight-core` library. It exists so administrators and reviewers can script library setup, schema work, pack management, and comment export.

Design rules:

- **Non-interactive by default.** Commands never prompt. Prompts appear only with `--interactive`. Confirmations required by the GUI are satisfied with `--yes` in the CLI.
- **Machine-readable output** with `--json`, using a stable envelope (Section 3).
- **Documented exit codes** (Section 4).
- **Same rules as the GUI.** The CLI cannot bypass policy, launch files, or write to package files.
- **No network**, except `update check`, which follows the same settings and policy as the GUI.

## 2. Global options

| Option | Meaning |
| --- | --- |
| `--config <path>` | Use a specific config file (same resolution as the GUI) |
| `--json` | Print the JSON envelope to stdout; human messages go to stderr |
| `--quiet` | Errors only |
| `--verbose` | Extra detail to stderr (never file contents) |
| `--no-color` | Disable color (also honors the `NO_COLOR` environment variable) |
| `--interactive` | Allow prompts |
| `--yes` | Assume yes for confirmations that are permitted by policy |
| `--version` | Print version, commit, and build date |
| `--help` | Help for any command |

## 3. JSON envelope

```json
{
  "schema": "foliolight/cli-output/v1",
  "command": "library verify",
  "ok": false,
  "data": { },
  "warnings": [
    { "code": "LIB_UNMAPPED_FOLDER", "message": "Folder is not mapped to a document type", "path": "Cobalt Example Systems/Misc 2" }
  ],
  "errors": [
    { "code": "LIB_FILE_TOO_LARGE", "message": "File exceeds the 512 MB limit", "path": "Cobalt Example Systems/SAR/sar-full.xml" }
  ]
}
```

- Paths are relative to the library root unless the command works on an arbitrary file.
- Error and warning `code` values are stable and documented; `message` text may change.

## 4. Exit codes

| Code | Meaning |
| --- | --- |
| 0 | Success (warnings allowed) |
| 1 | General error |
| 2 | Usage error (bad arguments) |
| 3 | Validation failed (schema, pack, library) |
| 4 | Integrity or signature verification failed |
| 5 | Not found (file, library, pack) |
| 6 | Denied by policy |
| 7 | Not writable (for commands that write) |
| 10 | Update available (only with `update check --exit-code`) |

## 5. Commands

### library

```
foliolight-cli library add <path> [--name <name>] [--id <id>]
foliolight-cli library list
foliolight-cli library remove <id>
foliolight-cli library init <id> [--force]
foliolight-cli library verify [<id>]
```

- `add` registers a folder in the config. It does not write to the folder.
- `init` writes a starter `foliolight-library.yaml` to the library root. It refuses if one exists unless `--force` is given, and `--force` keeps a `.bak` copy.
- `verify` runs the checks in [library-convention](library-convention.md#12-validation).

```
> foliolight-cli library add "D:\Packages" --name "Agency packages" --id main
Registered library "Agency packages" (main) at D:\Packages

> foliolight-cli library verify main --json
{ "schema": "foliolight/cli-output/v1", "command": "library verify", "ok": true,
  "data": { "csps": 3, "documentTypes": 14, "files": 2214, "unmapped": 1, "sidecars": 2, "orphanedComments": 3 },
  "warnings": [ ... ], "errors": [] }
```

### csp

```
foliolight-cli csp scaffold <library-id> <csp-name> [--types SSP,SAP,SAR,POAM,Other]
foliolight-cli csp list <library-id>
```

`scaffold` creates the CSP folder and the document-type folders. It never overwrites or removes anything and reports which folders already existed.

```
> foliolight-cli csp scaffold main "Delta Example Services"
Created: Delta Example Services\SSP, SAP, SAR, POAM, Other
```

### detect

```
foliolight-cli detect <file>
```

Reports format, confidence, OSCAL model and version, matching view schemas, and the one that would be chosen.

```
> foliolight-cli detect "D:\Packages\Aster Example Cloud\SSP\ssp-2026-10.json"
Format:   JSON (high confidence)
OSCAL:    system-security-plan 1.1.2
Schemas:  builtin:oscal/ssp (selected), example-publisher/oscal-ssp-compact
```

### schema

```
foliolight-cli schema infer <sample-file> --out <schema.view.yaml> [--id <publisher/name>] [--install]
foliolight-cli schema validate <schema-file>... [--strict]
foliolight-cli schema list
foliolight-cli schema show <schema-id>
```

- `infer` writes a draft view schema using the [inference algorithm](view-schema.md#11-inference-algorithm-schema-builder-and-schema-infer). `--install` also copies it to the user schemas folder.
- `validate` exits with 3 when any file fails.

```
> foliolight-cli schema infer samples\vendor-export.json --out vendor.view.yaml --id my-agency/vendor-export
Inferred 4 sections, 2 tables (7 and 5 columns). Wrote vendor.view.yaml

> foliolight-cli schema validate vendor.view.yaml
vendor.view.yaml: valid (1 warning)
  line 42, column 9: unknown field "colour" (ignored)
```

### pack

```
foliolight-cli pack validate <pack-folder>
foliolight-cli pack install <pack-folder> [--trust-key <key.pub>] [--allow-unsigned]
foliolight-cli pack list
foliolight-cli pack verify [<pack-id>]
foliolight-cli pack remove <pack-id> [--version <version>]
foliolight-cli trust add-key <key.pub> --publisher <publisher>
foliolight-cli trust revoke-key <key-id>
foliolight-cli trust pin <pack-folder>
```

- `install` validates, verifies hashes and signature, and copies the pack. Unsigned packs require `--allow-unsigned` and may be blocked by policy (exit 6).
- `verify` rechecks installed packs and drop-folder packs against their hashes and the trust store.

```
> foliolight-cli pack install packs\example-publisher.oscal-ssp-compact
Validated 2 schemas. Signature verified (key 0123456789ABCDEF, Example Publisher).
Installed example-publisher/oscal-ssp-compact 2.1.0 (trust: signed)
```

### comments

```
foliolight-cli comments list <library-id> [--csp <name>] [--type <doc-type>] [--state attached|changed|orphaned] [--status open|resolved|...]
foliolight-cli comments export <library-id> [--csp <name>] [--type <doc-type>] [--status ...] --format md|csv|json --out <file>
```

The CLI reads comments but does not create or edit them in v1 (comment authoring is a GUI activity).

```
> foliolight-cli comments export main --csp "Aster Example Cloud" --status open --format csv --out aster-open.csv
Exported 17 comments to aster-open.csv
```

### config

```
foliolight-cli config path
foliolight-cli config get <key>
foliolight-cli config set <key> <value>
foliolight-cli config validate
```

`set` on a policy-locked key exits with 6.

### update

```
foliolight-cli update check [--exit-code]
foliolight-cli update verify <downloaded-zip> [--sums <SHA256SUMS>] [--sig <SHA256SUMS.minisig>]
```

- `check` follows the same settings and policy as the GUI.
- `verify` works fully offline. It checks the zip against `SHA256SUMS` and the signature against the release public keys embedded in the binary. This is the supported way for air-gapped agencies to validate a release transferred on approved media.

```
> foliolight-cli update verify E:\transfer\foliolight-1.2.0-windows-x64.zip
SHA-256 matches SHA256SUMS. Signature valid (release key 2026-A). Version 1.2.0.
```

### doctor

```
foliolight-cli doctor
```

Reports app version, portable or per-user mode, config and data paths, policy file and locked keys, WebView2 runtime version (or missing), effective network state (update check on or off), and library reachability.

## 6. Scripting example

```powershell
# Register a library, scaffold a new CSP, install an approved pack, and verify everything.
$cli = "C:\Tools\Foliolight\foliolight-cli.exe"
& $cli library add "\\fileserver\ato\Packages" --id ato --name "ATO packages"
& $cli csp scaffold ato "Delta Example Services"
& $cli pack install "\\fileserver\ato\packs\example-publisher.oscal-ssp-compact"
$result = & $cli library verify ato --json | ConvertFrom-Json
if (-not $result.ok) { exit 3 }
```

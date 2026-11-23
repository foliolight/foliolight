# Specification: Application Configuration and Policy

Status: Draft v0.1. Spec ID: `foliolight/config/v1`.

## 1. Principles

- Plain YAML (safe subset) or JSON, human-readable and diffable.
- No registry use (ASSUMPTION A-19).
- An administrator can lock settings with a read-only policy file, without Group Policy templates or registry keys.
- There is no telemetry setting because there is no telemetry.

## 2. Config location

Resolved in this order; the first match wins:

| Order | Source | Data directory |
| --- | --- | --- |
| 1 | `--config <path>` command-line flag (GUI and CLI) | Folder containing that file |
| 2 | Portable mode: a file named `foliolight.portable` next to the executable | `<exe folder>\data\` |
| 3 | Per-user default | `%APPDATA%\Foliolight\` for config, packs, and schemas; `%LOCALAPPDATA%\Foliolight\` for the WebView2 user data folder |

In portable mode, the WebView2 user data folder is redirected to `<exe folder>\data\webview\`, so nothing is written to the user profile. If the portable folder is not writable (for example, read-only media), Foliolight runs with in-memory settings and says so in the status bar.

Contents of the data directory:

```
config.yaml           User settings
trust.yaml            Pack trust store
pins.yaml             Pinned "current" files per library
schemas/              User-created view schemas
packs/                Installed schema packs
logs/                 Only if logging is enabled (ASSUMPTION A-22)
webview/              WebView2 user data (portable mode)
```

## 3. Settings file

```yaml
apiVersion: foliolight/config/v1
kind: Config
libraries:
  - id: main
    name: Agency packages
    path: 'D:\Packages'
    rules: {}                  # optional mapping rules, same shape as foliolight-library.yaml
    useLibraryFile: true       # read foliolight-library.yaml at the root if present
    followLinks: none          # none | within-root
updates:
  check: false                 # ASSUMPTION A-15: off until the user opts in
  intervalHours: 24
  timeoutSeconds: 5
  channel: stable              # stable | prerelease
  downloadFolder: ""           # empty means ask each time
network:
  proxy: system                # system | none | "http://proxy.example.com:8080"
links:
  remoteUrls: copy-only        # copy-only | confirm-open (ASSUMPTION A-01)
  launch:
    allowExtensions: [.pdf, .docx, .xlsx, .pptx, .vsdx, .txt, .csv, .md, .png, .jpg, .jpeg, .gif, .svg, .drawio]
    unknownTypes: confirm      # confirm | block
limits:
  maxFileSizeMB: 512
  maxYamlSizeMB: 64
  maxDepth: 256
  maxAttachmentMB: 25
comments:
  author: ""                   # asked on first comment (ASSUMPTION A-09)
  storeExcerpts: true
ui:
  theme: system                # system | light | dark
  fontScale: 1.0
  reduceMotion: system
  density: comfortable         # comfortable | compact
packs:
  dropFolders: []
schemaBindings:                # "Use this view for this folder" (see opening-files spec)
  - match: { library: main, csp: "Aster Example Cloud", documentType: SSP }
    schema: builtin:oscal/ssp
  - match: { library: main, csp: "Cobalt Example Systems", documentType: VulnReports, fileGlob: "*-detail-*.json" }
    schema: user/cobalt-vuln-detail
bundledPacks:                  # packs that ship inside the app; each can be turned off
  foliolight/fedramp-json:
    enabled: true              # policy can lock this; off means no suggestions or checks for these files
openChecks:
  fitNotices: true             # show "new fields" and "missing fields" notices
  structureCheck: true         # informational structure check in the background
  dismissedFields: []          # managed by "Don't show this again"
export:
  enabled: true                # policy can lock this to false
  defaultFormat: xlsx          # xlsx | csv
  defaultValues: as-shown      # as-shown | raw
  aboutFile: true              # CSV: also write <name>.about.txt
  startFolder: downloads       # downloads | last-used; the Save As dialog always appears
logging:
  enabled: false
  level: info                  # info | debug
```

Validation: `schemas/spec/config-v1.schema.json`. Invalid settings are reported and replaced with defaults for that key only; the app still starts.

### Fixed security rules (not configurable)

These are compiled in and cannot be relaxed by config or policy:

- Executable and script file types are never launched (see [threat model](../security/threat-model.md#safe-external-launch)).
- The web view never has network access.
- XML DTDs are never processed.
- Downloads are never executed.
- Exports never contain images, attachments, formulas, macros, or clickable links (see [export](export.md#4-safety-rules)).

## 4. Precedence

`Policy (locked keys)` > `command-line flags` > `config.yaml` > `built-in defaults`.

## 5. Admin policy

An administrator may place `foliolight-policy.yaml` in either:

- `%ProgramData%\Foliolight\foliolight-policy.yaml` (writable only by administrators on standard images), or
- next to the executable (for portable deployments where the folder itself is admin-controlled).

```yaml
apiVersion: foliolight/policy/v1
kind: Policy
locked:
  updates.check: false               # air-gapped: disable update checks entirely
  links.remoteUrls: copy-only
  links.launch.unknownTypes: block
  limits.maxFileSizeMB: 256
  export.enabled: true               # set false to remove export where data handling rules require it
  bundledPacks.foliolight/fedramp-json.enabled: true   # set false to ship nothing program-specific
trust:
  preseedFile: trust.yaml            # relative to the policy file
  allowUserKeys: false               # users cannot add their own pack keys
  allowUnsignedPacks: false
```

The Settings screen shows locked settings as read-only with "Managed by your organization."

## 6. Air-gapped profile

Setting `updates.check: false` (by user or policy) guarantees zero network activity. `foliolight-cli doctor` reports the effective network state so administrators can verify it.

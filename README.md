# paket-storage-packages

## Probe metadata

| Field | Value |
|---|---|
| Pattern | storage-packages |
| Target framework | net8.0 |
| NuGet source | https://api.nuget.org/v3/index.json |
| Storage mode | packages (traditional, creates packages/ directory) |
| Resolution strategy | default (max) |
| Dependency groups | Main only (no named groups) |
| Generated | 2026-04-22 |

## Feature exercised

This probe exercises Paket's traditional `storage: packages` mode combined with
per-dependency build options (`redirects: force`, `copy_local: false`,
`import_targets: false`). These are pure build-time settings that Paket records
in the `paket.dependencies` file but which have no effect on the shape of
`paket.lock`. The probe validates that Mend detects all dependencies correctly
regardless of storage mode and that per-dep build annotations do not confuse or
suppress dependency detection.

## Expected dependency tree

### Direct dependencies (Main group)

| Package | Resolved version | Per-dep option | Has transitives |
|---|---|---|---|
| Newtonsoft.Json | 13.0.3 | `redirects: force` | No |
| Serilog | 3.1.1 | `copy_local: false` | No |
| Microsoft.Extensions.Logging | 8.0.0 | `import_targets: false` | Yes |
| AutoMapper | 12.0.1 | (none) | Yes |

### Transitive dependencies (resolved by lockfile)

| Package | Resolved version | Required by |
|---|---|---|
| Microsoft.Extensions.DependencyInjection.Abstractions | 8.0.1 | Microsoft.Extensions.Logging, Microsoft.Extensions.Logging.Abstractions, Microsoft.Extensions.Options, AutoMapper |
| Microsoft.Extensions.Logging.Abstractions | 8.0.0 | Microsoft.Extensions.Logging |
| Microsoft.Extensions.Options | 8.0.0 | Microsoft.Extensions.Logging |
| Microsoft.Extensions.Primitives | 8.0.0 | Microsoft.Extensions.Options |

### Detection expectations

- Mend should detect **8 unique packages** in total from `paket.lock`.
- All packages should be attributed to the `Main` (default) group.
- Source should be recorded as `https://api.nuget.org/v3/index.json`.
- `storage: packages` must not prevent detection — Mend reads `paket.lock`,
  not the presence or absence of a local `packages/` directory.
- Per-dep options (`redirects`, `copy_local`, `import_targets`) are build
  annotations only; they must not cause any package to be dropped, duplicated,
  or mis-versioned in the detected dependency tree.
- Transitive packages must appear as children of their direct-dependency
  parents in the dependency tree, not as top-level entries.

## File structure

```
paket-storage-packages/
├── paket.dependencies       # storage: packages + per-dep build options
├── paket.lock               # Fully resolved lockfile (8 packages)
├── expected-tree.json       # Expected Mend dependency tree (probe format)
├── README.md                # This file
└── src/
    └── MyProject/
        ├── MyProject.csproj # SDK-style net8.0 project
        └── paket.references # 4 direct package references
```

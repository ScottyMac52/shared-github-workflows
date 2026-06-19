# Shared Workflows for all repos

## Reusable Workflow Flowchart

```mermaid
flowchart TD
    A[Trigger: Push Tag / PR / Dispatch] --> B{ci_mode?}
    B -->|Yes| C[Run Tests + Coverage Only]
    B -->|No| D{run_tests?}
    D -->|No| E[Skip Tests]
    D -->|Yes| F[Discover Solution/Projects]
    F --> G[Restore + Build Solution]
    G --> H{run_tests?}
    H -->|Yes| I[Run All Tests]
    I --> J{run_coverage?}
    J -->|Yes| K[Generate Coverage Report]
    J -->|No| L[Skip Coverage]
    H -->|No| L
    C --> M[End - Quality Gate]
    K --> N{run_build_release?}
    L --> N
    E --> N
    N -->|Yes| O[Determine Version]
    N -->|No| Y[End]
    O --> P{build_mode?}
    P -->|Executable| Q[dotnet publish Single EXE]
    P -->|Library| R[Build + Copy All Non-Test Projects]
    Q --> S{enable_installer?}
    S -->|Yes| T[Build Inno Setup Installer\n+ Auto-generate .iss if missing]
    S -->|No| U[Prepare ZIP Artifact]
    R --> U
    T --> U
    U --> V[Upload Artifacts]
    V --> W{Should Create Release?}
    W -->|Yes| X[Create GitHub Release\n+ Attach ZIP + Setup.exe]
    W -->|No| Y[End]
    X --> Y
    style C fill:#ffcc00
    style M fill:#ffcc00
    style Y fill:#90EE90
```

## Stub 

```yaml
name: Build and Release

on:
  push:
    tags:
      - 'v*.*.*.*'
    paths-ignore:
      - '.github/workflows/**'
  workflow_dispatch:
    inputs:
      dotnet_version:
        description: '.NET version'
        required: false
        default: '10.0.x'
        type: string
      enable_installer:
        description: 'Build Setup.exe installer?'
        required: false
        default: false
        type: boolean
      installer_iss_path:
        description: 'Path to installer.iss'
        required: false
        default: 'installer/installer.iss'
        type: string
      app_publisher:
        description: 'Publisher name'
        required: false
        default: 'Vyper Industries'
        type: string
      skip_projects:
        description: 'Projects to skip (comma-separated)'
        required: false
        default: 'LuaParser'
        type: string
      create_release:
        description: 'Create GitHub Release?'
        required: false
        default: false
        type: boolean

jobs:
  call-reusable:
    name: Build via Shared Workflow
    uses: ScottyMac52/shared-github-workflows/.github/workflows/reusable-build-and-release.yml@main
    with:
      dotnet_version: ${{ inputs.dotnet_version }}
      enable_installer: ${{ inputs.enable_installer }}
      installer_iss_path: ${{ inputs.installer_iss_path }}
      app_publisher: ${{ inputs.app_publisher }}
      skip_projects: ${{ inputs.skip_projects }}
      create_release: ${{ inputs.create_release }}
    secrets:
      inherit: true
```

## Samples


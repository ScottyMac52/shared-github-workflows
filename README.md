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
    branches: [ main ]
    tags:
      - 'v*.*.*.*'
    paths-ignore:
      - '.github/workflows/**'
  pull_request:          # Optional: for PR quality gates
    branches: [ main ]
  workflow_dispatch:
    inputs:
      dotnet_version:
        description: '.NET version'
        required: false
        default: '10.0.x'
        type: string
      enable_installer:
        description: 'Build Setup.exe?'
        required: false
        default: false
        type: boolean
      run_tests:
        description: 'Run unit tests?'
        required: false
        default: true
        type: boolean
      run_coverage:
        description: 'Run code coverage?'
        required: false
        default: true
        type: boolean
      run_build_release:
        description: 'Build + create release artifacts?'
        required: false
        default: true
        type: boolean
      ci_mode:
        description: 'CI-only mode (tests + coverage only, no build/release)'
        required: false
        default: false
        type: boolean
      # other inputs...

jobs:
  call-reusable:
    uses: ScottyMac52/shared-github-workflows/.github/workflows/reusable-build-and-release.yml@main
    with:
      dotnet_version: ${{ inputs.dotnet_version || '10.0.x' }}
      enable_installer: ${{ inputs.enable_installer || false }}
      run_tests: ${{ inputs.run_tests || true }}
      run_coverage: ${{ inputs.run_coverage || true }}
      run_build_release: ${{ inputs.run_build_release || true }}
      ci_mode: ${{ inputs.ci_mode || false }}
      # ... other inputs
    secrets:
      inherit: true
```

## Samples


# Nixomatic GitHub Action

A GitHub Action for setting up reproducible Nix environments with packages from specific nixpkgs revisions.

This action uses [nixomatic.com](https://nixomatic.com) to generate Nix flakes on-demand, allowing you to mix and match packages from different nixpkgs revisions in your CI/CD workflows.

## Usage

```yaml
- uses: curriedsoftware/nixomatic-action@main
  with:
    packages: python3@3.13.1 nodejs@24.13.0 jq
    run: python --version
```

## Inputs

| Input         | Description                                                                                                                                    | Required | Default |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------------|----------|---------|
| `packages`    | Packages to install (space or comma-separated)                                                                                                 | Yes      | -       |
| `install-nix` | Whether to install Nix. Set to `false` if Nix is already available.                                                                            | No       | `true`  |
| `run`         | Script to run inside the Nix environment. If provided, runs the script and exits. If not provided, adds packages to PATH for subsequent steps. | Yes      | -       |

## Package Formats

- `package` - Latest from nixos-unstable (e.g., `jq`, `ripgrep`)
- `package@version` - Specific version resolved via [nxv](https://nxv.urandom.io) (e.g., `python3@3.13.1`)
- `package:revision` - Exact nixpkgs commit hash (e.g., `python3:3b93cf5abc123`)

## Examples

### Basic usage

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: curriedsoftware/nixomatic-action@main
    with:
      packages: cowsay jq ripgrep
      run: |
        cowsay "Hello from nixomatic!"
        jq --version
        rg --version
```

### Specific package versions

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: curriedsoftware/nixomatic-action@main
    with:
      packages: python3@3.13.1 nodejs@24.13.0
      run: |
        python3 --version
        node --version
```

### Multi-line package list

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: curriedsoftware/nixomatic-action@main
    with:
      packages: |
        cowsay
        jq
        ripgrep
      run: |
        cowsay "Hello from nixomatic!"
        jq --version
        rg --version
```

### With Nix already installed

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: cachix/install-nix-action@v31
  - uses: curriedsoftware/nixomatic-action@main
    with:
      packages: hello
      install-nix: false
      run: hello
```

## How It Works

1. Installs Nix using [cachix/install-nix-action](https://github.com/cachix/install-nix-action) (unless `install-nix: false`)
2. Builds a URL with the specified packages pointing to `nixomatic.com`
3. Runs the provided script inside `nix develop` (if `run` is specified)

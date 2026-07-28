# Kubermatic EE Downloader

A CLI tool to download Kubermatic Enterprise Edition binaries from OCI registries.

## Overview

`kubermatic-ee-downloader` pulls enterprise tool binaries (such as the conformance tester) from their OCI registries and saves them locally.

### Authentication

Credentials are resolved in the following order:

1. **CLI flags** — `--username` and `--password`
2. **Docker config** — `~/.docker/config.json` (e.g. after `docker login`)
3. **Interactive prompt** — if credentials are still missing, the tool asks on stdin

## Installation

### From Source

```bash
make build
# Binary is written to bin/kubermatic-ee-downloader
```

### Go Install

```bash
go install k8c.io/kubermatic-ee-downloader/cmd@latest
```

## Usage

### List Available Tools

```bash
kubermatic-ee-downloader list
```

Output shows one row per tool with all supported versions, operating systems, and architectures:

```text
TOOL                 VERSIONS   OS                       ARCH           DESCRIPTION
conformance-tester   latest     linux, darwin, windows   amd64, arm64   Kubermatic conformance cli
```

### Download a Tool

```bash
# Uses current machine's OS and arch, latest version
kubermatic-ee-downloader get conformance-tester

# Explicit version, OS, and arch
kubermatic-ee-downloader get conformance-tester \
  --version v1.2.0 \
  --os linux \
  --arch amd64 \
  --output /usr/local/bin

# With explicit credentials
kubermatic-ee-downloader get conformance-tester \
  --username user \
  --password pass
```

The downloaded binary is named `{tool}_{version}-{os}_{arch}`, for example:
`conformance-tester_latest-linux_amd64`

### Flags

#### Global

| Flag | Short | Description |
|------|-------|-------------|
| `--username` | `-u` | Registry username |
| `--password` | `-p` | Registry password |
| `--verbose` | `-v` | Enable verbose logging |
| `--catalog-url` | | URL of the tools catalog YAML (defaults to the catalog on this repository's `main` branch) |
| `--catalog-file` | | Path to a local tools catalog YAML, instead of fetching `--catalog-url` |

#### `get` command

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--version` | `-V` | tool-specific or `latest` | Tool version to download |
| `--os` | | current machine OS | Target operating system (`linux`, `darwin`, `windows`) |
| `--arch` | | current machine arch | Target architecture (`amd64`, `arm64`) |
| `--registry` | `-r` | tool-specific | Override OCI registry |
| `--output` | `-o` | `.` | Output directory |

## Adding Tools

Tools are defined in [`internal/tools/tools.yaml`](internal/tools/tools.yaml), which is fetched at runtime from the `main` branch of this repository. To add a new tool, append an entry and open a pull request:

```yaml
my-tool:
  description: "Short description of the tool"
  registry: "quay.io/kubermatic/my-tool"
  binary_name: "my-tool"
  tags:
    - latest-cli
    - v1.0.0-cli
  architectures:
    - amd64
    - arm64
  os:
    - linux
    - darwin
    - windows
```

| Field | Description |
| ----- | ----------- |
| `description` | Short human-readable description shown in `list` |
| `registry` | Full OCI registry reference (without tag) |
| `binary_name` | Filename used when saving the downloaded binary |
| `tags` | Available version tags; the first entry is used as the default |
| `architectures` | Supported CPU architectures |
| `os` | Supported operating systems |

The tag pulled from the registry is constructed as `{tag}-{os}_{arch}`, e.g. `latest-cli-linux_amd64`.

### Using a Custom Catalog

Both commands accept an alternative catalog, which is useful for testing an entry before it is merged, or for running fully offline:

```bash
# Local file
kubermatic-ee-downloader list --catalog-file internal/tools/tools.yaml
kubermatic-ee-downloader get my-tool --catalog-file ./my-catalog.yaml

# Alternative URL (e.g. a branch or fork)
kubermatic-ee-downloader list \
  --catalog-url https://raw.githubusercontent.com/my-org/kubermatic-ee-downloader/my-branch/internal/tools/tools.yaml
```

`--catalog-file` and `--catalog-url` are mutually exclusive.

## Development

```bash
make fmt          # Format code
make vet          # Run go vet
make lint         # Run golangci-lint
make test         # Run tests
make verify-all   # Run all verification checks
```

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.

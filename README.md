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

#### `get` command

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--version` | `-V` | tool-specific or `latest` | Tool version to download |
| `--os` | | current machine OS | Target operating system (`linux`, `darwin`, `windows`) |
| `--arch` | | current machine arch | Target architecture (`amd64`, `arm64`) |
| `--registry` | `-r` | tool-specific | Override OCI registry |
| `--output` | `-o` | `.` | Output directory |

## Adding Tools

Tools are defined in [`internal/tools/tools.yaml`](internal/tools/tools.yaml) and embedded into the binary at build time. To add a new tool, append an entry and rebuild:

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

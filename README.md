# OpenCode Sandbox Kit

A Docker Sandbox Kit workload for running [OpenCode](https://opencode.ai/) as a sandboxed coding agent while exposing its web interface to the host.

The workload is based on Docker's published OpenCode sandbox template and repins OpenCode to the version declared by the Kit.

## What this provides

* **OpenCode web UI** running inside the sandbox on port `4096`
* **Sandboxed agent execution** with the Docker Sandbox Kit agent workload
* Persistent OpenCode application state for each sandbox
* Shared global OpenCode configuration and skills
* Global agent definitions for a general coding workflow
* Optional credential injection for:

  * GitHub
  * OpenRouter
  * Exa
  * Context7
* Network policy restricting sandbox egress to the services OpenCode needs
* Local Ollama access through the Docker host
* Automatic registration of the Sandbox Kit MCP gateway when one is provided

The project deliberately keeps the **coding-agent workflow global** rather than tying the workload to a particular application repository.

## Repository contents

| File                           | Purpose                                                            |
| ------------------------------ | ------------------------------------------------------------------ |
| `opencode-workload.yaml`       | Sandbox Kit workload definition                                    |
| `opencode-workload.dockerfile` | Image definition and pinned OpenCode installation                  |
| `opencode.json`                | Global OpenCode configuration, including the coding-agent workflow |
| `opencode-context.md`          | Global `AGENTS.md` content supplied to OpenCode                    |
| `agents/`                      | Global agent prompt definitions                                    |

The application repository being worked on is separate from this repository. It becomes the sandbox's workspace when the sandbox is launched.

## Requirements

You need:

* Docker with Docker Sandbox / the `sbx` CLI and Sandbox Kit v3 support
* An Ollama installation on the same machine that runs the sandbox
* The Ollama model you want OpenCode to use

The workload is designed to use Ollama through Docker's host-service networking. It does **not** require the sandbox to know the host's LAN IP address.

## Running the workload

The simplest way to use the workload from this repository is to let `sbx` build the Kit from the local directory:

```bash
sbx run . /path/to/your/project
```

The project directory supplied to `sbx` becomes the OpenCode workspace.

You can also use the current directory as the workspace:

```bash
sbx run .
```

The workload publishes OpenCode's web interface on container port `4096`. The Sandbox Kit runtime assigns the corresponding host port.

## OpenCode version

The workload currently defaults to OpenCode **2.0.22**.

The version is a Kit argument and is installed explicitly during the image build:

```yaml
args:
  version:
    default: "2.0.22"
```

This keeps the published Kit capability version and the actual OpenCode binary in sync.

## Agent workflow

The global OpenCode configuration defines a general coding workflow with four agents:

* **Architect** — plans implementation work without changing files
* **Test writer** — writes tests without changing production code
* **Implementer** — implements production code without changing tests
* **Reviewer** — reviews code without changing it

The repository's `AGENTS.md` rules provide the general behavioral constraints shared by these agents, including inspecting the existing project before changing it, preserving working-tree changes, treating tests as part of the specification, keeping scope controlled, and reporting meaningful verification results.

Project-specific instructions remain with the project being worked on rather than being built into this workload.

## Credentials

Credentials are optional and are handled through Docker Sandbox Kit's credential capabilities rather than being stored in the repository or sandbox filesystem.

The workload supports:

| Service    | Environment variable |
| ---------- | -------------------- |
| GitHub     | `GITHUB_TOKEN`       |
| OpenRouter | `OPENROUTER_API_KEY` |
| Exa        | `EXA_API_KEY`        |
| Context7   | `CONTEXT7_API_KEY`   |

A credential only needs to be configured when the corresponding service is needed.

How credentials are stored and bound is handled by Docker Sandbox Kit rather than by this repository. Do not put API keys into `opencode.json`, the workload YAML, or a committed `.env` file.

## Local Ollama

The workload is designed to use an Ollama server running on the **Docker host**.

OpenCode inside the sandbox connects to:

```text
http://host.docker.internal:11434
```

The workload's network policy permits the host Ollama service on port `11434`.

### Default Ollama setup

If Ollama is installed normally and listening on its default local endpoint, no special `OLLAMA_HOST` configuration is required.

Make sure Ollama is running before starting the sandbox.

For example:

```bash
ollama serve
```

If Ollama is already running as a system service, simply make sure the service is running.

### Ollama listening on a LAN address

If Ollama needs to listen on a specific LAN address rather than only its default local interface, configure `OLLAMA_HOST` on the **Docker host**.

For example:

```bash
export OLLAMA_HOST=http://192.168.2.86:11434
```

Put that setting in the appropriate shell/service environment for your system if you want it to persist.

The important point is that the IP address belongs to **Ollama's host-side configuration**. You do not put that IP into `opencode-workload.yaml` or `opencode.json`.

The sandbox continues to reach the host service through:

```text
http://host.docker.internal:11434
```

This keeps the workload independent of the particular LAN address used by the host.

## Adding your own host network access

The public workload deliberately does not contain access to arbitrary machines on the author's LAN.

If you need OpenCode to reach another service on your own network — for example, a local ComfyUI server, a NAS, a development server, or an internal website — create a **local mixin** containing your additional network permissions.

Docker Sandbox Kit v3 mixins are overlays that compose onto a workload. Multiple mixins can be used, and their network policies are combined during composition.

### Example personal network mixin

Create a file such as:

```text
liz-opencode-network.yaml
```

with:

```yaml
# syntax=docker/sandbox-kit:3

schemaVersion: "3"

kind: mixin

displayName: My OpenCode Network

description: Personal network access for local OpenCode services

capabilities:
  - type: com.docker.sandbox/network-policy@1
    config:
      runtime:
        allow:
          - 192.168.1.100:8188
          - "*.example.com:443"
```

Replace the example entries with the services you actually want OpenCode to reach.

For example, a local service listening on port `8080` at `192.168.1.100:8080` could be added as:

```yaml
          - 192.168.1.100:8080
```

A specific HTTPS host could be added as:

```yaml
          - internal.example.com:443
```

Keep the permissions as narrow as practical: specify the actual host and port needed by the service rather than allowing an entire network when that isn't necessary.

### Keep your mixin local

If you keep your personal mixin in this repository but do not want it published, add its filename to `.gitignore`:

```gitignore
liz-opencode-network.yaml
```

This lets you maintain one working checkout while keeping your personal network permissions out of the public repository.

### Run with the mixin

Compose the mixin onto the workload when launching the sandbox:

```bash
sbx run . --kit ./liz-opencode-network.yaml /path/to/your/project
```

The workload remains the main Kit; the mixin simply adds your additional network permissions.

You can create additional mixins for other personal requirements instead of modifying the public workload itself. A mixin can be declaration-only, so a network-only mixin does not need its own Dockerfile.

## MCP

When Docker Sandbox Kit provides an MCP gateway, the workload's startup hook registers that gateway in OpenCode's configuration.

The configuration is **merged** with the existing `opencode.json` rather than replacing it. This preserves the workload's OpenCode settings and any other MCP configuration already present.

If no MCP gateway is provided, the gateway-specific configuration is simply not used.

## Network policy

The workload uses a restrictive runtime network policy rather than unrestricted sandbox egress.

The public policy includes the external services required by the workload, including:

* GitHub
* OpenRouter
* Exa
* Context7
* OpenCode
* package and schema services needed by OpenCode and its tooling
* the host Ollama service

Personal LAN services do not belong in the public workload. Add those through a local mixin instead.

This keeps the published workload useful to other users without giving every sandbox access to services that only exist on the author's network.

## Why the application repository is separate

This repository defines the **coding environment**, not a particular software project.

The workload therefore contains:

* the OpenCode installation
* global OpenCode configuration
* global agent definitions
* general agent instructions
* credentials and network capability declarations

The project being worked on is supplied separately when the sandbox is launched.

This makes the same workload reusable across different repositories without embedding project-specific paths or development instructions into the Kit.

## Development

To modify the workload itself, edit the Kit files in this repository and run it directly from the checkout.

Docker Sandbox Kit v3 supports building and running Kits directly from their source directories, so a registry is not required for local development.

For descriptor validation:

```bash
docker buildx build . -f opencode-workload.yaml --output type=cacheonly
```

To run the local workload against a project:

```bash
sbx run . /path/to/your/project
```

To compose a local mixin:

```bash
sbx run . --kit ./liz-opencode-network.yaml /path/to/your/project
```

A mixin cannot run by itself; it is composed onto a workload.

# pi-vanilla image builder

Builds the **pi-vanilla** container image — the plain, unmodified
[Pi](https://pi.dev/) coding agent on top of the `deven` toolchain image —
and publishes it to Alibaba Cloud ACR via the ACR builder service.

Part of the per-image split of the archived
[pi-agent-image](https://github.com/lulin/pi-agent-image) project:

```
deven  ->  pi-vanilla  ->  pi-agent
          (this repo)
```

## What is in the image

| Layer | Contents |
|-|-|
| Vanilla Pi | `@earendil-works/pi-coding-agent` installed globally via npm (`--ignore-scripts`, per official install instructions), version pinned by `PI_VERSION` |
| inherited | everything from [`deven`](../deven): Ubuntu 24.04, uv + Python 3.14, Node.js 24, Rust |

- `ENTRYPOINT ["pi"]`, `WORKDIR /workspace`
- Provider API keys are passed at runtime, e.g. `-e ANTHROPIC_API_KEY`

## Repository layout

```
Dockerfile    # FROM deven (by ACR tag) + npm install pi (from pi-agent-image/aliyuncs/pi-vanilla)
```

## Building

ACR builder service (云端构建), one rule for this repo:

| Setting | Value |
|-|-|
| Code source | this GitHub repository |
| Dockerfile path | `Dockerfile` |
| Context directory | repo root (no context files are used) |
| Namespace/repo | `***/pi-vanilla` |
| Tag rules | `latest` + the pi version baked in (e.g. `0.87.1`) |
| Build args | `PI_VERSION=0.87.1` (or set the default in the Dockerfile) |

Other optional args: `BASE_IMAGE` (defaults to
`registry.cn-hangzhou.aliyuncs.com/***/deven:latest`; same-region builders
can use the `registry-vpc` endpoint), `NPM_REGISTRY` (empty = upstream).

**Ordering:** trigger the `deven` builder first when the toolchain changed —
this image consumes `deven:latest` from ACR, it does not rebuild it.

Manual build:

```bash
docker build --build-arg PI_VERSION=0.87.1 -t pi-vanilla:latest .
```

## Using

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/***/pi-vanilla:latest
docker run -it --rm -v "$PWD:/workspace" \
  -e ANTHROPIC_API_KEY \
  registry.cn-hangzhou.aliyuncs.com/***/pi-vanilla:latest
```

## Releasing a new pi version

1. Bump `PI_VERSION` (Dockerfile default and/or builder build arg).
2. Run the builder; it tags `latest` and the version tag.
3. Rebuild `pi-agent` afterwards (it inherits this image).

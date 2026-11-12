# langflow-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Langflow](https://github.com/langflow-ai/langflow).

## What this is

A four-file TOS Docker application archive (`langflowdocker.tar.gz`) that installs
Langflow on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `langflowdocker`)
- `langflowdocker.lang` — 23-language superset (TOS requires 14)
- `langflowdocker.svg` — application icon (1157 bytes, 3 SVG elements)
- `docker-compose.yml` — one service, image `langflowai/langflow:1.9.3`

## How it works

| | |
|---|---|
| Container image | `langflowai/langflow:1.9.3` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8114/` (container port 7860) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/data` → `/Volume1/DockerAppData/langflowdocker/data` |
| Logs | `docker logs langflowdocker` (stdout/stderr) |



## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 7860.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/langflowdocker/data` | Langflow application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/langflowdocker.tar.gz + .sha256
scripts/build.sh 1.9.3-2   # packaging iteration bump
scripts/build.sh 1.9.3-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/langflowdocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v1.9.3-1`**, assets `langflowdocker.tar.gz` +
   `langflowdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `langflowdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `1.9.3-1` → automated validation → review.

## License / branding

Langflow is distributed by its upstream authors under **MIT**. The container image is the official image from Docker Hub (`langflowai/langflow:1.9.3`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

- The application runs inside a container; data lives in the application data directory (`/Volume1/DockerAppData/langflowdocker/`) as described above.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.

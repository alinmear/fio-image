# fio-image

Minimal Docker image (Alpine-based) with [fio](https://github.com/axboe/fio), the flexible I/O
tester, and [jq](https://jqlang.org/) for processing fio's JSON output.
Built automatically via GitHub Actions and published to the GitHub Container Registry.

## Supported platforms

Published images are multi-arch (`linux/amd64` and `linux/arm64`), so `docker pull`/`docker run`
automatically get the right one for your machine - including Apple Silicon (M-series) MacBooks
running Docker Desktop.

## Usage

```bash
docker pull ghcr.io/alinmear/fio-image:latest

# random read/write test against a mounted directory
docker run --rm -v "$PWD/data:/data" ghcr.io/alinmear/fio-image:latest \
  --name=randrw --directory=/data --rw=randrw --bs=4k --size=1G --runtime=60 --time_based

# JSON output piped through jq (read IOPS)
docker run --rm -v "$PWD/data:/data" --entrypoint sh ghcr.io/alinmear/fio-image:latest -c \
  'fio --name=randread --directory=/data --rw=randread --bs=4k --size=1G --output-format=json \
   | jq ".jobs[0].read.iops"'
```

## Available tags

- `latest` — most recent build from the `main` branch
- `vX.Y.Z`, `vX.Y` — released versions (pushed via git tag, e.g. `v1.0.0`)
- `<git-sha>` — build for a specific commit

## Build

The image is built and published automatically by
[`.github/workflows/docker-publish.yml`](.github/workflows/docker-publish.yml) on every push to
`main` and on version tags (`vX.Y.Z`), for both `linux/amd64` and `linux/arm64`. Pull requests
only build the image (amd64 only, to verify it still works) without pushing.

To build locally for your own machine (including an Apple Silicon Mac):

```bash
docker build -t fio-image .
```

To build a multi-arch image locally (e.g. to reproduce the CI build), use Buildx:

```bash
docker buildx create --use  # once, if you don't already have a buildx builder
docker buildx build --platform linux/amd64,linux/arm64 -t fio-image .
```

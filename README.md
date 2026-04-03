<div align="center">

<img src="web/public/logo.svg" alt="Octopus Logo" width="120" height="120">

### Octopus Fork

**A fork of Octopus maintained by zbsdsb, focused on scored routing, health visibility, and Docker/GHCR deployment ergonomics**

English | [简体中文](README_zh.md)

</div>

## Upstream Project

This repository is maintained as a fork of `bestruirui/octopus`. It does not try to duplicate the full upstream documentation.

- Upstream repository: https://github.com/bestruirui/octopus
- Upstream English README: https://github.com/bestruirui/octopus/blob/dev/README.md
- Upstream Chinese README: https://github.com/bestruirui/octopus/blob/dev/README_zh.md

For the original product overview, base configuration, API details, and full usage guide, refer to the upstream README directly.

## What Is Different In This Fork

This fork keeps Octopus as an LLM API aggregation service, but adds a stronger focus on routing operations and self-hosted deployment:

- richer group routing behavior beyond fixed priority and static weights
- more operational visibility in the console
- fork-friendly Docker, GHCR, and workflow guidance

## Added Features

### 1. Scored Group Routing

This fork adds a `Scored` group mode that selects channel/model/key combinations using multiple signals.

- combines health score, priority, and weight in candidate ordering
- key selection considers recent usage, 429 throttling, and accumulated cost
- score weights are configurable instead of hard-coded

### 2. Channel Health Dashboard

The health page provides:

- per-channel health score
- per-model health details inside each channel
- search, filtering, and sorting
- expand / collapse for model details

### 3. Enhanced Logs

The log page adds:

- search
- status filtering
- sorting
- realtime toggle
- API format display and filtering

### 4. Better Group Editor

The right-side “selected models” area now supports:

- search
- sorting
- same-source duplicate detection
- `Base URL` display

This is especially useful when the same upstream base URL is backed by multiple keys and repeated models.

### 5. Group Default Values

The settings page now provides global defaults for:

- first-token timeout
- session stickiness

Behavior:

- custom group value wins when set
- empty or `0` inherits the global default

### 6. Stability Fixes

This fork also includes several practical fixes:

- locale tag issues that broke the settings page
- lint / build issues in shared animation primitives
- missing group member metadata that broke production builds

## Deployment

### Recommended Image

If you want to deploy this fork directly, use the GHCR images:

- `ghcr.io/zbsdsb/octopus:dev`
- `ghcr.io/zbsdsb/octopus:dev-alpine`

Tag meaning:

- `dev`: published automatically from the fork `dev` branch, suitable for active deployment and testing
- `latest`: reserved for the release / `master` flow

### Run With Docker

```bash
docker run -d \
  --name octopus \
  -v /path/to/data:/app/data \
  -p 8080:8080 \
  ghcr.io/zbsdsb/octopus:dev
```

### Run With Docker Compose

The repository includes a ready-to-use compose example:

```bash
cp docker-compose.ghcr.yml docker-compose.yml
docker compose up -d
```

By default, it pulls:

```bash
ghcr.io/zbsdsb/octopus:dev
```

To override the image, port, or data directory:

```bash
export OCTOPUS_IMAGE=ghcr.io/zbsdsb/octopus:dev
export OCTOPUS_DATA_DIR=./data
export OCTOPUS_PORT=8080
docker compose -f docker-compose.ghcr.yml up -d
```

### Migrate From The Official Image To This Fork

If your current production deployment still uses the official image, for example:

```yaml
image: bestrui/octopus
```

the safest migration path is:

1. keep the existing data mount unchanged
2. back up the database and config first
3. replace only the image reference
4. recreate the container

Using the already validated deployment shape as an example:

```yaml
services:
  octopus:
    image: ghcr.io/zbsdsb/octopus:dev
    ports:
      - "8080:8080"
    volumes:
      - "/path/to/data:/app/data"
    container_name: octopus
    restart: unless-stopped
```

Migration steps:

```bash
# 1. Back up data
sudo cp -a /path/to/data/data.db /path/to/data/data.db.bak-$(date +%Y%m%d-%H%M%S)
sudo cp -a /path/to/data/config.json /path/to/data/config.json.bak-$(date +%Y%m%d-%H%M%S)

# 2. Update the image in docker-compose.yml
# image: bestrui/octopus
# -> image: ghcr.io/zbsdsb/octopus:dev

# 3. Pull and recreate
docker compose pull octopus
docker compose up -d octopus
```

Do not change the bind-mounted data directory to a new empty path during migration, or the service will start with a fresh empty dataset.

If you are using SQLite, avoid running the old and new containers against the same `data.db` file at the same time. The safer approach is to create a copied database for precheck on another port, then switch the production container after validation.

### GHCR Publishing Workflows

This fork now has two image publishing paths:

1. `dev` branch
   - workflow: `.github/workflows/docker-dev.yaml`
   - trigger: `push` to `dev` or manual dispatch
   - tags:
     - `ghcr.io/zbsdsb/octopus:dev`
     - `ghcr.io/zbsdsb/octopus:dev-alpine`

2. `master` branch
   - workflow: `.github/workflows/release.yaml`
   - trigger: `push` to `master`
   - tags:
     - `ghcr.io/zbsdsb/octopus:latest`
     - `ghcr.io/zbsdsb/octopus:latest-alpine`

### Deployment Notes

- Keep the existing `/app/data` mount during upgrades to preserve data
- New setting rows are created automatically at startup; you do not need to extend `data/config.json` manually
- If other users need to pull your GHCR image directly, make sure the `octopus` container package is set to `public` in GitHub Packages

## Current Branch State

The default branch of this fork is `dev`, and it already contains the features described above.  
If you just want to deploy this fork, pulling `ghcr.io/zbsdsb/octopus:dev` is enough.

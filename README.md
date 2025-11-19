# Spilo Images

This repository builds and publishes [Spilo](https://github.com/zalando/spilo) PostgreSQL container images with custom build configurations.

## Build Configuration

All images are built from Spilo source with:
- `TIMESCALEDB_APACHE_ONLY=false` - Includes full TimescaleDB (not just Apache-licensed version)
- PostGIS version updated to 3.6 (patched from upstream Spilo's 3.5)
- Explicit PostgreSQL versions
- Multi-platform support (amd64, arm64)
- Built on 8-core runners for faster compilation

## Available Images

Images are published to `ghcr.io/gfx-labs/spilo-images/spilo` with explicit version tags only.

### PostgreSQL 16 (Spilo 3.x)
- `ghcr.io/gfx-labs/spilo-images/spilo:16-3.3-p3`

### PostgreSQL 17 (Spilo 4.x)
- `ghcr.io/gfx-labs/spilo-images/spilo:17-4.0-p3`

All tags follow the format `<pg-version>-<spilo-version>`. No `-latest` tags are provided to ensure explicit version tracking.

## Building Locally

To build locally, clone the Spilo repository and build from `postgres-appliance`:

```bash
# Clone Spilo at specific version
git clone --branch 3.3-p3 --depth 1 https://github.com/zalando/spilo.git
cd spilo/postgres-appliance

# Build for PostgreSQL 16
sed -i 's/POSTGIS_VERSION=3\.5/POSTGIS_VERSION=3.6/g' Dockerfile
docker build \
  --build-arg TIMESCALEDB_APACHE_ONLY=false \
  -t spilo:16-3.3-p3 .

# Build for PostgreSQL 17
git checkout 4.0-p3
sed -i 's/POSTGIS_VERSION=3\.5/POSTGIS_VERSION=3.6/g' Dockerfile
docker build \
  --build-arg TIMESCALEDB_APACHE_ONLY=false \
  -t spilo:17-4.0-p3 .
```

## GitHub Actions

The repository uses a matrix strategy for building multiple PostgreSQL versions in parallel. The workflow:

- Clones the upstream Spilo repository at the specified tag
- Builds from `postgres-appliance/Dockerfile` with custom build arguments
- Pushes to GitHub Container Registry (ghcr.io)
- Supports multi-platform builds (amd64, arm64)
- Uses build caching for faster builds
- Can be triggered manually via workflow dispatch

### Matrix Configuration

All version combinations are defined in the workflow matrix in `.github/workflows/build.yml`:

```yaml
strategy:
  matrix:
    include:
      - pg_version: "16"
        spilo_version: "3.3-p3"
        postgis_version: "3.6"
      - pg_version: "17"
        spilo_version: "4.0-p3"
        postgis_version: "3.6"
```

### Build Process

1. Checks out Spilo repository at specified tag
2. Patches Dockerfile to update PostGIS version (3.5 → 3.6)
3. Builds with custom arguments

### Build Arguments

Each build uses:
- `TIMESCALEDB_APACHE_ONLY=false`: Include full TimescaleDB with proprietary features
- PostGIS version is patched in the Dockerfile before build

## Adding New PostgreSQL Versions

To add support for a new PostgreSQL version:

1. Add the version to the matrix in `.github/workflows/build.yml`:
   ```yaml
   - pg_version: "18"
     spilo_version: "5.0-p1"
     postgis_version: "3.6"
   ```
2. Commit and push to trigger the build

## Updating Spilo Versions

To update to a new Spilo version:

1. Update the `spilo_version` in the matrix in `.github/workflows/build.yml`
2. Commit and push to trigger the build

## Version Matrix

| PostgreSQL Version | Spilo Version | PostGIS Version | Image Tag |
|-------------------|---------------|-----------------|-----------|
| 16 | 3.3-p3 | 3.6 | 16-3.3-p3 |
| 17 | 4.0-p3 | 3.6 | 17-4.0-p3 |

All builds use `TIMESCALEDB_APACHE_ONLY=false`.

## Upstream Resources

- [Zalando Spilo](https://github.com/zalando/spilo)
- [Spilo Releases](https://github.com/zalando/spilo/releases)
- [Patroni](https://github.com/zalando/patroni)
- [TimescaleDB](https://github.com/timescale/timescaledb)

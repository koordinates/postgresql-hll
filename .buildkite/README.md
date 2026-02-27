# Buildkite CI Pipeline for PostgreSQL HLL

This directory contains the Buildkite CI pipeline configuration for building and publishing Debian packages of the PostgreSQL HLL extension.

## Pipeline Overview

The pipeline consists of two main steps:

1. **Build**: Builds the PostgreSQL HLL extension for PostgreSQL 16 and creates a Debian package
2. **Publish**: Uploads the built package to the Koordinates APT repository

## Files

- `pipeline.yml` - Buildkite pipeline configuration
- `build.sh` - Build script that creates the Debian package

## What it does

1. **Build Step**:
   - Uses the `jammybuild` Docker image
   - Installs PostgreSQL 16 development headers
   - Builds the HLL extension
   - Creates staging directory with installed files
   - Uses `ci-tools` container with pre-installed fpm to create Debian package
   - Artifacts the `.deb` file

2. **Publish Step**:
   - Downloads the built `.deb` file
   - Uses `aptly-upload` to publish to the APT repository
   - Targets the `kx-builds-jammy` repository

## Versioning

The package version is constructed as:
- Base version from `hll.control` (e.g., `2.19`)
- Plus CI build number: `+kx-ci${BUILDKITE_BUILD_NUMBER}`
- Final format: `postgresql-16-hll_2.19+kx-ci123_amd64.deb`

## Local Testing

To test the build locally:

```bash
export ECR="276514628126.dkr.ecr.ap-southeast-2.amazonaws.com"
./.buildkite/build.sh
```

## Environment Variables

- `ECR` - AWS ECR registry URL
- `BUILDKITE_BUILD_NUMBER` - Build number (provided by Buildkite)
- `BUILDKITE_JOB_ID` - Job ID for unique container naming
- `APT_GPG_KEY` - GPG key for signing packages (optional)
- `APTLY_UNAME` - Aptly username for publishing
- `APTLY_PASSWD` - Aptly password for publishing

## Docker Images Used

- `${ECR}/jammybuild:master.latest` - Ubuntu Jammy build environment
- `${ECR}/ci-tools:cds-ci-tools-upgrade.latest` - CI tools with fpm for packaging and signing

## Package Details

The built package:
- Name: `postgresql-16-hll`
- Depends on: `postgresql-16`
- Installs to: `/usr/lib/postgresql/16/` and `/usr/share/postgresql/16/extension/`
- Description: PostgreSQL HLL extension for HyperLogLog cardinality estimation

## Adding Support for Other PostgreSQL Versions

To add support for other PostgreSQL versions, modify the `build.sh` script:
1. Change or parameterize the `PG_VERSION` variable
2. Update the package installation step to include the appropriate `postgresql-server-dev-XX` package
3. Consider creating separate build steps in `pipeline.yml` for each version
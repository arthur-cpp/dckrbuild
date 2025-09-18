# dckrbuild

Build Debian packages in clean environment using [Docker](https://docker.com). Process is split into
three stages:

 - initialization: create base images that are used for all builds for same distribution;
 - prepare: install dependencies of source package;
 - build: build package using prepared image.

Prepare and build stages are separated for two reasons:

 - build can be run several times without wasting time on installing dependencies
 - build is running without network access (however there is `--with-network` flag to enable it)

## Usage

Prepare base images for distribution you want to use:

```
dckrbuild-pyinit ubuntu24.04
```

If you want to include custom mirrors use `--repo` and `--key` flags or create configuration file
and pass it with `--config` option. Keys from configuration file are treated as command line options
to initialization script:

```
prefix: build
key:
  - local.repo.gpg
repo:
  - deb http://local.repo/debian {codename} contrib backports
```

Install build dependencies for your package

```
dckrbuild-prepare ubuntu24.04
```

Run build

```
dckrbuild-pybuild ubuntu24.04
```

## Buildhost Integration

There are two scripts that can be used inside buildhost: `dckrbuild-print-deps` in conjuction with
`apt satisfy` and optional `dckrbuild-version-bump` to set temporary version from git tags:

```
# Install dependencies
apt update
apt satisfy `dckrbuild-print-deps`

# Bump version
dckrbuild-version-bump

# Build binary package
dpkg-buildpackage --no-sign -b -j4
```

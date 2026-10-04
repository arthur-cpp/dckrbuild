# dckrbuild

A lightweight Docker-based build system for Debian/Ubuntu packages. It provides a clean, isolated environment for building packages and caches dependencies using Docker images.

## Prerequisites

1.  **Docker Desktop Settings**: 
    If you are using Docker Desktop, ensure that `/tmp` (or your project workspace) is added to `Settings -> Resources -> File Sharing`.
   
2.  **Local Python Environment**:
    Install required libraries on your host machine:
    ```bash
    pip install dulwich python-debian PyYAML
    ```

3.  **Local Infrastructure**:
    Prepare folder for caching:
    ```bash
    mkdir -p ~/.ccache
    ```

4.  **Installation**:
    Clone this repository and add its root folder to your `$PATH` in `.bashrc` or `.zshrc`:
    ```bash
    git clone https://github.com/arthur-cpp/dckrbuild.git
    export PATH="path/to/dckrbuild:$PATH"
    ```

---

## Configuration

Before starting, check the `config/` directory:
* **dists.conf**: List the target distributions (e.g., `debian12`, `debian13`, `ubuntu22.04`) you want to support. Every top-level `dckrbuild-*` script iterates over this list.
* **repositories.yaml** (+ matching `.gpg` key(s)): Declares extra APT repositories that get baked into every `build:<dist>` base image, so build-dependencies that don't live in the official Debian/Ubuntu archives are still available. Fields:
  * `prefix` — tag prefix for the generated images (default `build`).
  * `key` — list of GPG keyring files, **binary/dearmored** (not ASCII-armored), sitting next to this YAML file. Each one is copied into `/etc/apt/trusted.gpg.d/` inside the image and trusted globally for all repositories.
  * `repo` — list of `deb ...` source lines. `{dist}` and `{codename}` placeholders are substituted with the current target's name (`debian12`) and codename (`bookworm`) respectively.

  Both `key` and `repo` are lists, so several third-party repositories can be declared in the same file — see [Adding another repository](#adding-another-repository).

### Adding another repository

To make another APT repository available inside the build images:

1. Make sure you have its signing key as a **binary (dearmored)** keyring file, and drop it into `config/`. If the upstream only ships an ASCII-armored key, convert it first:
   ```bash
   gpg --dearmor < repo-key.asc > config/repo-key.gpg
   ```

2. Add a key and repo entry to `config/repositories.yaml`:
   ```yaml
   key:
     - repo-key.gpg
   repo:
     - deb https://example.com/debian {codename} main
   ```
   Only use `{codename}`/`{dist}` if the third-party repo actually publishes packages for every distribution listed in `dists.conf`. If it only serves a single suite, hardcode that suite name instead — `apt-get update` fails hard if a configured suite doesn't exist upstream.

   There's no `[signed-by=...]` on the line — dckrbuild drops every configured key into `/etc/apt/trusted.gpg.d/`, so it's trusted for all repositories already.

3. Rebuild the base images so the change takes effect:
   ```bash
   dckrbuild-host-init
   ```

---

## Usage Workflow

### 1. One-time Host Setup
Initialize the base OS images for all distributions listed in `config/dists.conf`:
```bash
dckrbuild-host-init

```

*This command creates `build:dist` images containing basic build tools (build-essential, ccache) and your custom repositories.*

### 2. Project Setup

Go to your project directory (containing the `debian/` folder) and prepare the dependency layers:

```bash
dckrbuild-proj-prepare

```

*This script parses `debian/control` and creates a `build-tmp:project-dist` image for each target OS with all `Build-Depends` pre-installed.*

### 3. Build Project

Execute the multi-distro build from your project root:

```bash
dckrbuild-proj-build

```

* **Requirements**: Your project must have at least one Git tag (e.g., `v1.0.0`). Debian versioning requires the version string to start with a digit; the script automatically handles the `v` prefix.
* **Output**: Ready-to-install `.deb` packages and `.changes` files will be generated in `./result/<codename>/` (one subfolder per distribution, e.g. `./result/bookworm/`, `./result/trixie/`).

---

## Maintenance

To clean up dangling Docker images and stopped containers:

```bash
./core/dckrbuild-clean
```

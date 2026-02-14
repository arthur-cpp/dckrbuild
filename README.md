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
* **dists.conf**: List the target distributions (e.g., `debian12`, `ubuntu24.04`) you want to support.
* **psha.org.ru.yaml**: Custom repository settings, prefixes, and GPG keys.

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
* **Output**: Ready-to-install `.deb` packages and `.changes` files will be generated in the `./result` folder.

---

## Maintenance

To clean up dangling Docker images and stopped containers:

```bash
./core/dckrbuild-clean
```

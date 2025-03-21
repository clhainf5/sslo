# SSLO Via AppFramework POC

## Overview
This is a POC repo for providing SSLO analysis on an external box with components
that are compatible with AST.

System Diagram
![SSLO Overview](./img/sslo.png)

Result Dashboard
![SSLO Dashboard](./img/sslo-dashboard.png)

Pipeline Perf Overview
![SSLO Data Pipeline](./img/sslo-pipeline-overview.png)

## Getting Started

### Prerequisites

#### [Git Client](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

#### OpenSSL
Used for test certificate generation, can skip if using real certs.

#### Docker (or compatible) container environment with compose.
Installation Instructions:
  * [General (docker engine)](https://docs.docker.com/engine/install/)
  * [Ubuntu (docker engine)](https://docs.docker.com/engine/install/ubuntu/)
  * [RHEL (docker engine)](https://docs.docker.com/engine/install/rhel/)
  * [Podman](https://podman.io/docs/installation)

#### GNU Make (Strongly recommended)
Installation Instructions:
<details>
  <summary>Linux</summary>

  Ubuntu/Debian:
  ```
  sudo apt update
  sudo apt-get install build-essential
  ```

  Redhat:
  ```
  yum install make
  ```

</details>


<details>
  <summary>Mac</summary>

  ```
  brew install make
  ```

</details>

<details>
  <summary>Windows</summary>

  [Installation Instructions](https://gnuwin32.sourceforge.net/install.html)

</details>

### Installation

Clone the repo or download source tarball from the [release](https://github.com/f5devcentral/application-study-tool/releases) section.

```shell
# Clone the repo
git clone https://github.com/clhainf5/sslo 
cd sslo
# If AppFramework is still private:
# Generate a PAT with Read permission in f5observabilityhub/AppFramework on "Contents" 
git config submodule.AppFramework.url https://USERNAME:PAT@github.com:/F5ObservabilityHub/AppFramework # Only required if AppFramework is still private:
# This must be run before anything else to fully initialize the project layout
# The files listed below will not exist at the specified paths until this is run.
make init
# Create testing certs to use on the syslog receiver listener
make test-certs
# Depending on your docker environment you may need to allow the user in the otel collector (10001) access to it
sudo chown 10001 services/otel_collector/ssl/key.pem
# Start the tool
make start
# Load generator
python ./load_test.py
```

<details>
<summary>Instructions Without Make</summary>

```shell
# Clone the repo

git clone https://github.com/clhainf5/sslo 
cd sslo
# Initialize the core AppFramework components
git submodule init
git submodule update
# Copy the any template config files
find ./.init/project -type f | while read src; do \
    dest=$${src/.init\/project\//}; \
    dest_dir=$$(dirname $$dest); \
    mkdir -p $$dest_dir; \
    echo "📄 Copying $$src to $$dest..."; \
    if [ ! -f $$dest ]; then \
      cp -rn $$src $$dest; \
      echo "  ✅ $$dest file created."; \
    else \
      echo "  ⚠️  $$dest already exists. Skipping copy."; \
    fi \
  done
# Optional, run the openssl commands in Makefile to generate certificates for collector to use, or provide your own in services/otel_collector/ssl
# Edit env files and config files as required by the project #TODO - list these out
vi .env
# Start the tool
docker compose up -d
```

</details>

## Configuration

### Configure Grafana
The Grafana instance can be configured via environment variables using their standard
[options](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/#override-configuration-with-environment-variables).

The included .env (present after running `make init`) can be modified to set the initial admin
password to a value you select:

```
# Grafana Environment Variables
# These should be updated to more secure values.
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=admin
```

#### View The Dashboards
The default Grafana user/pass is `admin/admin`, and can be accessed at
`http://<hostname>:3000`.


## Updating Project Versions
Updating to a new release of this repo can be done with the following general process.

1. Review the release notes for all intermediate versions and check for warnings about
special instructions / breaking changes.
2. Recommended: backup this directory and all files.
3. Stash changes, update the repo state, and unstash changes as follows:

```shell
make stop
# This assumes you're upgrading from one v1.0.0+ version to another.
git stash
make upgrade-ast RELEASE_VERSION=v1.0.1 # (Desired release version, e.g. v1.0.1)
git stash pop
# <check for warning messages and merge any conflicts with your local changes>
make start
```

## Support

For support, please open a GitHub issue.  Note, the code in this repository is community supported and is not supported by F5 Networks.  For a complete list of supported projects please reference [SUPPORT.md](SUPPORT.md).

## Community Code of Conduct

Please refer to the [F5 DevCentral Community Code of Conduct](code_of_conduct.md).

## License

[Apache License 2.0](LICENSE)

## Copyright

Copyright 2014-2024 F5 Networks Inc.

### F5 Networks Contributor License Agreement

Before you start contributing to any project sponsored by F5 Networks, Inc. (F5) on GitHub, you will need to sign a Contributor License Agreement (CLA).

If you are signing as an individual, we recommend that you talk to your employer (if applicable) before signing the CLA since some employment agreements may have restrictions on your contributions to other projects.
Otherwise by submitting a CLA you represent that you are legally entitled to grant the licenses recited therein.

If your employer has rights to intellectual property that you create, such as your contributions, you represent that you have received permission to make contributions on behalf of that employer, that your employer has waived such rights for your contributions, or that your employer has executed a separate CLA with F5.

If you are signing on behalf of a company, you represent that you are legally entitled to grant the license recited therein.
You represent further that each employee of the entity that submits contributions is authorized to submit such contributions on behalf of the entity pursuant to the CLA.

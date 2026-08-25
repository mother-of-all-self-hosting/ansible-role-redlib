<!--
SPDX-FileCopyrightText: 2018-2026 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently there is one testing scenario available.

### `default`

Tests a standard Redlib installation.

#### Why the scenario stands a stub in Reddit's place

Redlib does not serve a single route until it has obtained an OAuth token from `https://www.reddit.com`, and it exits with status 1 after ten failed attempts. Reddit turns away the datacenter addresses that CI runners have, so an unaided Redlib on a runner never binds a port at all — it just crash-loops under the unit's `Restart=always`, and `systemctl is-active` says `active` the whole time.

So the scenario answers that one request itself. `prepare.yml` creates a throwaway certificate authority and a certificate for `www.reddit.com`, and `converge.yml` starts an nginx holding that certificate on the role's own container network under the `www.reddit.com` network alias, which Docker's embedded resolver answers before it forwards anything upstream. Redlib is pointed at the authority through the role's own `redlib_container_additional_volumes_custom` and `redlib_environment_variables_additional_variables` variables, so the wiring doubles as a test of those two knobs.

Nothing in the verification depends on Reddit answering, or on the stub returning anything resembling real Reddit data. Its only job is to let Redlib finish starting, so that the role's configuration can be read back out of a running process rather than off the disk.

#### What is verified

Verification goes past "the container is up":

- an unconfigured Redlib, started from the same image with none of the role's settings, is asked the same questions first, and every assertion is on an answer it gives differently
- every setting the role writes into its env file is read back from the running process on `/info.json`, including one that arrives through the free-form `redlib_environment_variables_additional_variables` escape hatch
- two further surfaces are checked, because they render the configuration through different code paths: `/robots.txt` and the preselected options on `/settings`
- the container is checked to run the image tag `redlib_version` names, to have been created from the image that tag points at right now, and to run a process reporting the version that image's own binary reports
- the container is checked to be read-only, capability-less, running as the configured user, on the role's network, with the role's additional volume mounted read-only
- the restart counter is watched for 45 seconds with everything already answering, because `Restart=always` makes a crash loop read as `active`

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```

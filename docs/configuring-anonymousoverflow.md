<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Redlib

This is an [Ansible](https://www.ansible.com/) role which installs [Redlib](https://github.com/httpjamesm/Redlib) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Redlib allows you to view StackOverflow threads without exposing your IP address, browsing habits, and other browser fingerprinting data to the website.

See the project's [documentation](https://github.com/httpjamesm/Redlib/blob/main/README.md) to learn what Redlib does and why it might be useful to you.

[<img src="assets/home_dark.webp" title="Home screen in dark mode" width="600">](assets/home_dark.webp) [<img src="assets/question_dark.webp" title="Question in dark mode" width="600">](assets/question_dark.webp) [<img src="assets/answers_light.webp" title="Answer in light mode" width="600">](assets/answers_light.webp)

## Adjusting the playbook configuration

To enable Redlib with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# redlib                                                               #
#                                                                      #
########################################################################

redlib_enabled: true

########################################################################
#                                                                      #
# /redlib                                                              #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Redlib you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
redlib_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Redlib under a subpath (by configuring the `redlib_path_prefix` variable) does not seem to be possible due to Redlib's technical limitations.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `redlib_environment_variables_additional_variables` variable

For a complete list of Redlib's config options that you could put in `redlib_environment_variables_additional_variables`, see its [`docker-compose.example.yml`](https://github.com/httpjamesm/Redlib/blob/main/docker-compose.example.yml).

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Redlib becomes available at the specified hostname like `https://example.com`.

[Libredirect](https://libredirect.github.io/), an extension for Firefox and Chromium-based desktop browsers, has support for redirections to Redlib. See [this section](https://github.com/httpjamesm/Redlib/blob/main/README.md#how-to-make-stack-overflow-links-take-you-to-redlib-automatically) on the official documentation for more information.

If you would like to publish your instance so that it can be used by anyone including Libredirect, please consider to send a PR to the [upstream project](https://github.com/httpjamesm/Redlib) to add yours to [`instances.json`](https://github.com/httpjamesm/Redlib/blob/main/instances.json), which Libredirect automatically fetches using a script (see [this FAQ entry](https://libredirect.github.io/faq.html#where_the_hell_are_those_instances_coming_from)).

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu redlib` (or how you/your playbook named the service, e.g. `mash-redlib`).

<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Tautulli

This is an [Ansible](https://www.ansible.com/) role which installs [Tautulli](https://tautulli.com/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Tautulli monitors the activity of a [Plex Media Server](https://www.plex.tv/), and keeps statistics and a history of what has been watched, by whom and on which device. It can also send notifications and newsletters.

See the project's [documentation](https://github.com/Tautulli/Tautulli/wiki) to learn what Tautulli does and why it might be useful to you.

## Adjusting the playbook configuration

To enable Tautulli with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# tautulli                                                             #
#                                                                      #
########################################################################

tautulli_enabled: true

########################################################################
#                                                                      #
# /tautulli                                                            #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Tautulli you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
tautulli_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

>[!NOTE]
> The `tautulli_path_prefix` variable can be adjusted to host under a subpath (e.g. `tautulli_path_prefix: /tautulli`).

### Configuring HTTP Basic authentication (optional)

To put HTTP Basic authentication on Traefik in front of Tautulli, add the following configuration to your `vars.yml` file. Refer to [this page](https://doc.traefik.io/traefik/reference/routing-configuration/http/middlewares/basicauth/) on the Traefik's documentation for details.

```yaml
tautulli_container_labels_traefik_middleware_basic_auth_enabled: true

tautulli_container_labels_traefik_middleware_basic_auth_users: 'USERNAME_HERE:HASHED_PASSWORD_HERE'
```

Make sure to replace `USERNAME_HERE` and `HASHED_PASSWORD_HERE` with your own values. You can create users by running `htpasswd -nb USERNAME_HERE PASSWORD_HERE`.

### Connecting to Plex Media Server

If Plex Media Server is installed on the same server with [its role](https://github.com/mother-of-all-self-hosting/ansible-role-plex), you can let Tautulli talk to it directly over the container network, instead of going through its public hostname. To do so, add the following configuration to your `vars.yml` file.

```yaml
# Lets Tautulli reach Plex Media Server by its container name
tautulli_container_additional_networks_custom:
  - "{{ plex_container_network }}"

# Starts Plex Media Server whenever Tautulli is started
tautulli_systemd_wanted_services_list_custom:
  - "{{ plex_identifier }}.service"
```

**Note**: if you use the MASH playbook, this is done for you.

Then use `plex` (or how you/your playbook named the service, e.g. `mash-plex`) as the Plex IP address or hostname, and `32400` as the port, when Tautulli asks for them.

### Mounting Plex Media Server logs (optional)

Tautulli's log viewer can show the logs of Plex Media Server. To use it on the same server, mount the data directory of Plex Media Server to the container read-only, by adding the following configuration to your `vars.yml` file:

```yaml
tautulli_container_additional_volumes_custom:
  - type: bind
    src: "{{ plex_data_path }}"
    dst: /plex
    options: readonly
```

Then set the **Logs Folder** under *Settings → Plex Media Server* (with **Show Advanced** turned on) to `/plex/Library/Application Support/Plex Media Server/Logs`.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `tautulli_environment_variables_additional_variables` variable

Tautulli reads any of its settings from a `TAUTULLI_<SETTING>` environment variable, named after the setting's key in its `config.ini` (e.g. `TAUTULLI_API_KEY`). A setting configured this way takes precedence over `config.ini`, and cannot be changed on the web UI.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Tautulli becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and follow the setup wizard to create an account and connect Tautulli to Plex Media Server.

**Note**: until the setup wizard is completed, anyone who can reach the URL can complete it. Make sure to do so right after installing, or to [enable HTTP Basic authentication](#configuring-http-basic-authentication-optional) on Traefik beforehand.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu tautulli` (or how you/your playbook named the service, e.g. `mash-tautulli`).

# Ansible role: `foundata.podman.host`

The `foundata.podman.host` Ansible role (part of the `foundata.podman` Ansible collection).



## Table of contents<a id="toc"></a>

- [Example playbooks, using this role](#examples)
- [Centralizing rootless storage](#rootless-storage)
- [Supported tags](#tags)<!-- ANSIBLE DOCSMITH TOC START -->
- [Role variables](#variables)
  - [`host_podman_state`](#variable-host_podman_state)
  - [`host_podman_autoupgrade`](#variable-host_podman_autoupgrade)
  - [`host_podman_service_state`](#variable-host_podman_service_state)
  - [`host_podman_containers`](#variable-host_podman_containers)
  - [`host_podman_storage`](#variable-host_podman_storage)
  - [`host_podman_registries`](#variable-host_podman_registries)
  - [`host_podman_policy`](#variable-host_podman_policy)
  - [`host_podman_auto_update`](#variable-host_podman_auto_update)
  - [`host_podman_auto_update_timer_settings`](#variable-host_podman_auto_update_timer_settings)
  - [`host_podman_docker_compat`](#variable-host_podman_docker_compat)
  - [`host_podman_rootless_users`](#variable-host_podman_rootless_users)
    - [`host_podman_rootless_users['name']`](#variable-host_podman_rootless_users-sub-name)
    - [`host_podman_rootless_users['state']`](#variable-host_podman_rootless_users-sub-state)
    - [`host_podman_rootless_users['linger']`](#variable-host_podman_rootless_users-sub-linger)
    - [`host_podman_rootless_users['subuid']`](#variable-host_podman_rootless_users-sub-subuid)
    - [`host_podman_rootless_users['subgid']`](#variable-host_podman_rootless_users-sub-subgid)
    - [`host_podman_rootless_users['auto_update']`](#variable-host_podman_rootless_users-sub-auto_update)
    - [`host_podman_rootless_users['containers']`](#variable-host_podman_rootless_users-sub-containers)
    - [`host_podman_rootless_users['storage']`](#variable-host_podman_rootless_users-sub-storage)
    - [`host_podman_rootless_users['registries']`](#variable-host_podman_rootless_users-sub-registries)
  - [`host_podman_selinux_manage`](#variable-host_podman_selinux_manage)
  - [`host_podman_selinux_label_paths`](#variable-host_podman_selinux_label_paths)
<!-- ANSIBLE DOCSMITH TOC END -->
- [Dependencies](#dependencies)
- [Compatibility](#compatibility)
- [External requirements](#requirements)



## Example playbooks, using this role<a id="examples"></a>

Installation with automatic upgrade:

```yaml
---

- name: "Initialize the foundata.podman.host role"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Trigger invocation of the foundata.podman.host role"
      ansible.builtin.include_role:
        name: "foundata.podman.host"
      vars:
        host_podman_autoupgrade: true
```

Rootless enablement for a service account (create the account first, e.g. with the `foundata.linux.user` role; deploy applications afterwards with the `foundata.podman.quadlet` role):

```yaml
---

- name: "Initialize the foundata.podman.host role"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Trigger invocation of the foundata.podman.host role"
      ansible.builtin.include_role:
        name: "foundata.podman.host"
      vars:
        host_podman_rootless_users:
          - name: "exampleapp"
        host_podman_selinux_label_paths:
          - "/var/lib/podman" # the accounts' home directories live below this path
```

Uninstall:

```yaml
---

- name: "Initialize the foundata.podman.host role"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Trigger invocation of the foundata.podman.host role"
      ansible.builtin.include_role:
        name: "foundata.podman.host"
      vars:
        host_podman_state: "absent"
```



## Centralizing rootless storage<a id="rootless-storage"></a>

By default, this collection does not influence where Podman stores its data at all: rootful storage lives below `/var/lib/containers`, and each rootless user's data lives below their home directory (`~/.local/share/containers` for images, container layers, **named volumes** — the precious application data —, secrets and networks; `~/.config/containers` for Quadlet units and configuration). If you want the persistent data of rootless applications in a central, predictable place instead of scattered across home directories, there are three independent, optional levers (ordered by recommendation):

**1. Central home directories for the service accounts (recommended, zero configuration).** Both trees hang off `$HOME`, so an account whose home is e.g. `/var/lib/podman/<application>` keeps units, configuration *and* all storage in one self-contained directory — nothing else needs to be configured. Do NOT place such homes below `/var/lib/containers` (that path carries the SELinux label of the *rootful* storage and relabeling would clobber the rootless labels); use a sibling tree like `/var/lib/podman` and list it in `host_podman_selinux_label_paths`:

```yaml
---

- name: "Rootless Podman with central, self-contained application trees"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Create the service account with a central home (example, see foundata.linux.user)"
      ansible.builtin.include_role:
        name: "foundata.linux.user"
      vars:
        user_linux_accounts:
          - name: "exampleapp"
            system: true
            home:
              path: "/var/lib/podman/exampleapp"
              create: true
            shell: "/usr/sbin/nologin"

    - name: "Trigger invocation of the foundata.podman.host role"
      ansible.builtin.include_role:
        name: "foundata.podman.host"
      vars:
        host_podman_rootless_users:
          - name: "exampleapp"
        host_podman_selinux_label_paths:
          - "/var/lib/podman"
```

**2. Relocate only the storage tree via `rootless_storage_path` (optional flatten).** This system-wide `storage.conf` setting moves every rootless user's *storage* (images, volumes, secrets — not units or configuration, which stay in `~/.config`) to a templated path. Combined with lever 1 it merely flattens the `~/.local/share/containers/storage` nesting; on its own it centralizes storage while homes stay wherever they are:

```yaml
    - name: "Trigger invocation of the foundata.podman.host role"
      ansible.builtin.include_role:
        name: "foundata.podman.host"
      vars:
        host_podman_storage:
          storage:
            rootless_storage_path: "$HOME/storage" # $HOME and $USER get expanded by Podman
```

**3. Relocate the configuration tree via `XDG_CONFIG_HOME` (possible, but discouraged).** Moving Quadlet units and configuration out of `~/.config` requires the environment variable to be visible to the user's *systemd instance* as well (e.g. via `~/.config/environment.d/`), otherwise the Quadlet generator keeps looking in the default location and the units are silently ignored — and every interactive `podman`/`systemctl --user` invocation needs it too. The `.config` dot-directory is already central when using lever 1; leave it alone unless you have a hard requirement.



## Supported tags<a id="tags"></a>

It might be useful and faster to only call parts of the role by using tags:

- `host_podman_setup`: Manage basic resources, such as packages or service users.
- `host_podman_config`: Manage settings, such as adapting or creating configuration files.
- `host_podman_service`: Manage services and daemons, such as running states and service boot configurations.

There are also tags usually not meant to be called directly but listed for the sake of completeness** and edge cases:

- `host_podman_always`, `always`: Tasks needed by the role itself for internal role setup and the Ansible environment.


<!-- ANSIBLE DOCSMITH MAIN START -->

## Role variables<a id="variables"></a>

Main entry point for the foundata.podman.host role

The following variables can be configured for this role:

| Variable | Type | Required | Default | Description (abstract) |
|----------|------|----------|---------|------------------------|
| `host_podman_state` | `str` | No | `"present"` | Determines whether the managed resources should be `present` or `absent`.<br><br>`present` ensures that required components, such as software packages, are installed and configured.<br><br>`absent` reverts changes as much as possible, such as […](#variable-host_podman_state) |
| `host_podman_autoupgrade` | `bool` | No | `false` | If set to `true`, all managed packages will be upgraded during each Ansible run (e.g., when the package provider detects a newer version than the currently installed one).<br><br>Note: This only affects the packages that provide Podman and its […](#variable-host_podman_autoupgrade) |
| `host_podman_service_state` | `str` | No | `"enabled"` | Defines the status of the service(s).<br><br>`enabled`: Service is running and will start automatically at boot.<br><br>`disabled`: Service is stopped and will not start automatically at boot.<br><br>`running`: Service is running but will not start […](#variable-host_podman_service_state) |
| `host_podman_containers` | `dict` | No | `{}` | Additional settings for the system-wide Podman container engine configuration at `/etc/containers/containers.conf`.<br><br>Dictionary structure:<br><br>- Keys: TOML section names of `containers.conf` (e.g. `containers`, `engine`, `network`, […](#variable-host_podman_containers) |
| `host_podman_storage` | `dict` | No | `{}` | Additional settings for the system-wide Podman storage configuration at `/etc/containers/storage.conf`.<br><br>Dictionary structure:<br><br>- Keys: TOML section names of `storage.conf` (e.g. `storage`, `storage.options`, `storage.options.overlay`). - […](#variable-host_podman_storage) |
| `host_podman_registries` | `dict` | No | `{}` | Additional settings for the system-wide container image registry configuration at `/etc/containers/registries.conf`.<br><br>Dictionary structure:<br><br>- Keys: Top-level TOML settings of `registries.conf` (e.g. `unqualified-search-registries`, […](#variable-host_podman_registries) |
| `host_podman_policy` | `dict` | No | `{}` | Container image trust / signature-verification policy, written to `/etc/containers/policy.json`.<br><br>If empty (the default), the distribution's stock `policy.json` is left untouched. If a policy is supplied, it is rendered verbatim as JSON and […](#variable-host_podman_policy) |
| `host_podman_auto_update` | `bool` | No | `true` | Whether the role manages the system-wide `podman-auto-update.timer` which periodically pulls newer images for containers labeled with `AutoUpdate=registry` or `AutoUpdate=local` and restarts their systemd units.<br><br>If `true`, the timer is managed […](#variable-host_podman_auto_update) |
| `host_podman_auto_update_timer_settings` | `dict` | No | `{}` | Configuration for the systemd timer that triggers the automatic container image update run (`podman-auto-update.timer`). This dictionary controls when and how often updates are checked and applied.<br><br>These settings map to systemd timer unit […](#variable-host_podman_auto_update_timer_settings) |
| `host_podman_docker_compat` | `bool` | No | `false` | If set to `true`, Docker compatibility gets installed and enabled: the `podman-docker` package (providing a `docker` command alias) and the Podman API socket (`podman.socket`, providing a Docker-compatible API at `/run/podman/podman.sock` for tools […](#variable-host_podman_docker_compat) |
| `host_podman_rootless_users` | `list` | No | `[]` | List of existing user accounts to be enabled for rootless Podman usage.<br><br>For each listed account, the role ensures the prerequisites for running rootless containers that start at boot and keep running without an interactive login session: […](#variable-host_podman_rootless_users) |
| `host_podman_selinux_manage` | `bool` | No | `true` | Whether the role manages SELinux settings related to container storage (file contexts for the paths listed in `host_podman_selinux_label_paths`, including running `restorecon` on changes). Only effective on platforms with SELinux; ignored elsewhere. |
| `host_podman_selinux_label_paths` | `list` | No | `[]` | List of additional filesystem paths that should carry the SELinux file contexts required for container storage (equivalent to `/var/lib/containers`). Use this when relocating storage, e.g. for a custom rootful `graphroot` or a central rootless […](#variable-host_podman_selinux_label_paths) |

### `host_podman_state`<a id="variable-host_podman_state"></a>

[*⇑ Back to ToC ⇑*](#toc)

Determines whether the managed resources should be `present` or `absent`.

`present` ensures that required components, such as software packages, are
installed and configured.

`absent` reverts changes as much as possible, such as removing packages,
deleting created configuration files, stopping services, restoring modified
settings, …

- **Type**: `str`
- **Required**: No
- **Default**: `"present"`
- **Choices**: `present`, `absent`



### `host_podman_autoupgrade`<a id="variable-host_podman_autoupgrade"></a>

[*⇑ Back to ToC ⇑*](#toc)

If set to `true`, all managed packages will be upgraded during each Ansible
run (e.g., when the package provider detects a newer version than the
currently installed one).

Note: This only affects the packages that provide Podman and its companion
tooling itself (e.g., `podman`, `netavark`, `aardvark-dns`, `passt`). It does
not control updates of container images; those are governed by
`host_podman_auto_update` and the `AutoUpdate=` setting of the deployed
Quadlet units (see the `foundata.podman.quadlet` role).

- **Type**: `bool`
- **Required**: No
- **Default**: `false`



### `host_podman_service_state`<a id="variable-host_podman_service_state"></a>

[*⇑ Back to ToC ⇑*](#toc)

Defines the status of the service(s).

`enabled`: Service is running and will start automatically at boot.

`disabled`: Service is stopped and will not start automatically at boot.

`running`: Service is running but will not start automatically at boot.
This can be used to start a service on the first Ansible run without
enabling it for boot.

`unmanaged`: Service will not start at boot, and Ansible will not manage
its running state. This is primarily useful when services are monitored
and managed by systems other than Ansible.

The singular form (`service`) is used for simplicity. However, the defined
status applies to all services if multiple are being managed by this role.

Podman itself is daemonless, so there is no main service to manage. The
units covered by this variable are the system `podman-auto-update.timer`
(if `host_podman_auto_update` is `true`), the `podman.socket` API socket
(if `host_podman_docker_compat` is `true`), and the per-user
`podman-auto-update.timer` units of the rootless users listed in
`host_podman_rootless_users` (if their `auto_update` is `true`).

- **Type**: `str`
- **Required**: No
- **Default**: `"enabled"`
- **Choices**: `enabled`, `disabled`, `running`, `unmanaged`



### `host_podman_containers`<a id="variable-host_podman_containers"></a>

[*⇑ Back to ToC ⇑*](#toc)

Additional settings for the system-wide Podman container engine
configuration at `/etc/containers/containers.conf`.

Dictionary structure:

- Keys: TOML section names of `containers.conf` (e.g. `containers`,
  `engine`, `network`, `secrets`, `machine`).
- Values: Dictionaries of `key: value` pairs for that section. Values are
  rendered as native TOML types (booleans as `true`/`false`, strings
  quoted, numbers unquoted, lists as arrays).

The settings are deep-merged over the role's internal platform defaults;
values set here take precedence. Example:

```yaml
host_podman_containers:
  containers:
    log_driver: "journald"
  engine:
    database_backend: "sqlite"
  network:
    firewall_driver: "nftables"
```

See `man containers.conf` or
https://github.com/containers/common/blob/main/docs/containers.conf.5.md
for all available settings.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`



### `host_podman_storage`<a id="variable-host_podman_storage"></a>

[*⇑ Back to ToC ⇑*](#toc)

Additional settings for the system-wide Podman storage configuration at
`/etc/containers/storage.conf`.

Dictionary structure:

- Keys: TOML section names of `storage.conf` (e.g. `storage`,
  `storage.options`, `storage.options.overlay`).
- Values: Dictionaries of `key: value` pairs for that section. Values are
  rendered as native TOML types (booleans as `true`/`false`, strings
  quoted, numbers unquoted, lists as arrays).

The settings are deep-merged over the role's internal platform defaults;
values set here take precedence. Example:

```yaml
host_podman_storage:
  storage:
    rootless_storage_path: "$HOME/storage"
```

The role only manages this configuration file. Providing and mounting the
underlying storage (dedicated disks, filesystems, `/etc/fstab`) is out of
scope. See the role's README for examples on centralizing rootless storage
(e.g. via `rootless_storage_path` or service account home directories).

See `man containers-storage.conf` or
https://github.com/containers/storage/blob/main/docs/containers-storage.conf.5.md
for all available settings.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`



### `host_podman_registries`<a id="variable-host_podman_registries"></a>

[*⇑ Back to ToC ⇑*](#toc)

Additional settings for the system-wide container image registry
configuration at `/etc/containers/registries.conf`.

Dictionary structure:

- Keys: Top-level TOML settings of `registries.conf` (e.g.
  `unqualified-search-registries`, `short-name-mode`) or `registry`.
- Values: Rendered as native TOML types. The `registry` key takes a list
  of dictionaries which is rendered as `[[registry]]` array-of-table
  entries (e.g. for mirrors or insecure registries).

The settings are deep-merged over the role's internal platform defaults;
values set here take precedence. Example:

```yaml
host_podman_registries:
  unqualified-search-registries:
    - "docker.io"
    - "quay.io"
  registry:
    - prefix: "docker.io"
      location: "registry-mirror.example.com:5000"
```

See `man containers-registries.conf` or
https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md
for all available settings.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`



### `host_podman_policy`<a id="variable-host_podman_policy"></a>

[*⇑ Back to ToC ⇑*](#toc)

Container image trust / signature-verification policy, written to
`/etc/containers/policy.json`.

If empty (the default), the distribution's stock `policy.json` is left
untouched. If a policy is supplied, it is rendered verbatim as JSON and
replaces the file. Distributing signing keys and configuring signature
lookaside storage (`registries.d`) is out of scope for this role.

Example (require GPG-signed images from one registry, accept anything
else):

```yaml
host_podman_policy:
  default:
    - type: "insecureAcceptAnything"
  transports:
    docker:
      "registry.example.com":
        - type: "signedBy"
          keyType: "GPGKeys"
          keyPath: "/etc/pki/containers/example.gpg"
```

See `man containers-policy.json` or
https://github.com/containers/image/blob/main/docs/containers-policy.json.5.md
for the full syntax.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`



### `host_podman_auto_update`<a id="variable-host_podman_auto_update"></a>

[*⇑ Back to ToC ⇑*](#toc)

Whether the role manages the system-wide `podman-auto-update.timer` which
periodically pulls newer images for containers labeled with
`AutoUpdate=registry` or `AutoUpdate=local` and restarts their systemd
units.

If `true`, the timer is managed and its state follows
`host_podman_service_state`. If `false`, the role ensures the timer is
disabled and stopped (use `host_podman_service_state: "unmanaged"` if
the timer should not be touched at all).

Rootless containers are not covered by the system timer; use the
`auto_update` option of `host_podman_rootless_users` entries for the
per-user equivalent.

- **Type**: `bool`
- **Required**: No
- **Default**: `true`



### `host_podman_auto_update_timer_settings`<a id="variable-host_podman_auto_update_timer_settings"></a>

[*⇑ Back to ToC ⇑*](#toc)

Configuration for the systemd timer that triggers the automatic container
image update run (`podman-auto-update.timer`). This dictionary controls
when and how often updates are checked and applied.

These settings map to systemd timer unit directives and are applied via a
drop-in override file. They take highest priority, overriding internal
defaults (see `__host_podman_auto_update_timer_settings_defaults` in
`vars/main.yml`).

Dictionary structure:

- Keys: Standard systemd `[Timer]` directives.
- Values: Corresponding configuration values. Common options include:
  - `OnCalendar`: Defines the schedule on which the update timer is
    triggered. Defaults to `daily`.
  - `RandomizedDelaySec`: Adds a random delay before execution to avoid
    simultaneous runs across systems. Defaults to `60m`.
  - `Persistent`: Ensures missed timer runs are executed at the next boot
    if the system was powered off. Defaults to `true`.

Special cases:

- For boolean values, use `true`/`false` (these will be converted to
  strings by the role as needed).

Only effective when `host_podman_auto_update` is `true` and
`host_podman_service_state` is not `unmanaged`.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`



### `host_podman_docker_compat`<a id="variable-host_podman_docker_compat"></a>

[*⇑ Back to ToC ⇑*](#toc)

If set to `true`, Docker compatibility gets installed and enabled: the
`podman-docker` package (providing a `docker` command alias) and the
Podman API socket (`podman.socket`, providing a Docker-compatible API at
`/run/podman/podman.sock` for tools like Traefik). The socket's state
follows `host_podman_service_state`.

If `false` (the default), the role ensures these components are absent.

- **Type**: `bool`
- **Required**: No
- **Default**: `false`



### `host_podman_rootless_users`<a id="variable-host_podman_rootless_users"></a>

[*⇑ Back to ToC ⇑*](#toc)

List of existing user accounts to be enabled for rootless Podman usage.

For each listed account, the role ensures the prerequisites for running
rootless containers that start at boot and keep running without an
interactive login session: subordinate UID/GID ranges (`/etc/subuid`,
`/etc/subgid`), systemd lingering (`loginctl enable-linger`), the per-user
configuration below `~/.config/containers/`, and optionally the per-user
`podman-auto-update.timer`.

The role does NOT create the accounts themselves and will fail if a listed
account does not exist. Create accounts beforehand (e.g. with the
`foundata.linux.user` role). Choosing a home directory like
`/var/lib/podman/<name>` for a service account keeps all of its container
data in a central location; see the role's README for details and examples.

Example:

```yaml
host_podman_rootless_users:
  - name: "zammad"
  - name: "example"
    linger: false
    auto_update: false
    storage:
      storage:
        rootless_storage_path: "$HOME/storage"
```

- **Type**: `list`
- **Required**: No
- **Default**: `[]`
- **List Elements**: `dict`

#### `host_podman_rootless_users['name']`<a id="variable-host_podman_rootless_users-sub-name"></a>

[*⇑ Back to ToC ⇑*](#toc)

Name of the user account to enable for rootless Podman usage. The
account must already exist.

- **Type**: `str`
- **Required**: Yes

#### `host_podman_rootless_users['state']`<a id="variable-host_podman_rootless_users-sub-state"></a>

[*⇑ Back to ToC ⇑*](#toc)

`present` ensures the rootless enablement for this account,
`absent` reverts it (disables lingering and the per-user timer,
removes the per-user configuration managed by this role; subordinate
ID ranges and the account itself are kept).

- **Type**: `str`
- **Required**: No
- **Default**: `"present"`
- **Choices**: `present`, `absent`

#### `host_podman_rootless_users['linger']`<a id="variable-host_podman_rootless_users-sub-linger"></a>

[*⇑ Back to ToC ⇑*](#toc)

Whether to enable systemd lingering (`loginctl enable-linger`) for the
account. Lingering starts the user's systemd instance at boot and keeps
it running without an interactive session; it is required for rootless
containers that should start at boot and survive logout. Only disable
this if you know what you are doing (e.g. containers are only run
interactively).

- **Type**: `bool`
- **Required**: No
- **Default**: `true`

#### `host_podman_rootless_users['subuid']`<a id="variable-host_podman_rootless_users-sub-subuid"></a>

[*⇑ Back to ToC ⇑*](#toc)

Subordinate UID range for the account in `start:count` format (e.g.
`100000:65536`, see `man subuid`). If unset (the default), the role
ensures a range exists and automatically allocates a free one if
missing.

- **Type**: `str`
- **Required**: No

#### `host_podman_rootless_users['subgid']`<a id="variable-host_podman_rootless_users-sub-subgid"></a>

[*⇑ Back to ToC ⇑*](#toc)

Subordinate GID range for the account in `start:count` format (e.g.
`100000:65536`, see `man subgid`). If unset (the default), the role
ensures a range exists and automatically allocates a free one if
missing.

- **Type**: `str`
- **Required**: No

#### `host_podman_rootless_users['auto_update']`<a id="variable-host_podman_rootless_users-sub-auto_update"></a>

[*⇑ Back to ToC ⇑*](#toc)

Whether to manage the account's `podman-auto-update.timer` (user scope)
which periodically updates this user's containers labeled with
`AutoUpdate=`. If `true`, the timer state follows
`host_podman_service_state`. If `false`, the role ensures the per-user
timer is disabled and stopped.

- **Type**: `bool`
- **Required**: No
- **Default**: `true`

#### `host_podman_rootless_users['containers']`<a id="variable-host_podman_rootless_users-sub-containers"></a>

[*⇑ Back to ToC ⇑*](#toc)

Additional per-user Podman container engine settings, written to
`~/.config/containers/containers.conf`. Same structure as
`host_podman_containers`. Per-user settings take precedence over the
system-wide configuration.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`

#### `host_podman_rootless_users['storage']`<a id="variable-host_podman_rootless_users-sub-storage"></a>

[*⇑ Back to ToC ⇑*](#toc)

Additional per-user Podman storage settings, written to
`~/.config/containers/storage.conf`. Same structure as
`host_podman_storage`. Per-user settings take precedence over the
system-wide configuration.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`

#### `host_podman_rootless_users['registries']`<a id="variable-host_podman_rootless_users-sub-registries"></a>

[*⇑ Back to ToC ⇑*](#toc)

Additional per-user container image registry settings, written to
`~/.config/containers/registries.conf`. Same structure as
`host_podman_registries`. Per-user settings take precedence over the
system-wide configuration.

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`



### `host_podman_selinux_manage`<a id="variable-host_podman_selinux_manage"></a>

[*⇑ Back to ToC ⇑*](#toc)

Whether the role manages SELinux settings related to container storage
(file contexts for the paths listed in `host_podman_selinux_label_paths`,
including running `restorecon` on changes). Only effective on platforms
with SELinux; ignored elsewhere.

- **Type**: `bool`
- **Required**: No
- **Default**: `true`



### `host_podman_selinux_label_paths`<a id="variable-host_podman_selinux_label_paths"></a>

[*⇑ Back to ToC ⇑*](#toc)

List of additional filesystem paths that should carry the SELinux file
contexts required for container storage (equivalent to
`/var/lib/containers`). Use this when relocating storage, e.g. for a
custom rootful `graphroot` or a central rootless storage tree like
`/var/lib/podman`.

Do not add paths below `/var/lib/containers` itself; they are already
covered by the distribution's default policy.

Only effective when `host_podman_selinux_manage` is `true` and the
platform uses SELinux.

Example:

```yaml
host_podman_selinux_label_paths:
  - "/var/lib/podman"
```

- **Type**: `list`
- **Required**: No
- **Default**: `[]`
- **List Elements**: `str`




<!-- ANSIBLE DOCSMITH MAIN END -->

## Dependencies<a id="dependencies"></a>

See `dependencies` in [`meta/main.yml`](./meta/main.yml).



## Compatibility<a id="compatibility"></a>

See `min_ansible_version` in [`meta/main.yml`](./meta/main.yml) and `__host_podman_supported_platforms` in [`vars/main.yml`](./vars/main.yml).



## External requirements<a id="requirements"></a>

There are no special requirements not covered by Ansible itself.

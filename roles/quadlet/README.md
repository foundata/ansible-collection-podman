# Ansible role: `foundata.podman.quadlet`

The `foundata.podman.quadlet` Ansible role (part of the `foundata.podman` Ansible collection).



## Table of contents<a id="toc"></a>

- [Example playbooks, using this role](#examples)
- [Run-once containers and timers](#run-once)
- [Rootless applications: data, UID mapping and SELinux](#rootless-notes)
- [Supported tags](#tags)<!-- ANSIBLE DOCSMITH TOC START -->
- [Role variables](#variables)
  - [`quadlet_podman_app`](#variable-quadlet_podman_app)
  - [`quadlet_podman_state`](#variable-quadlet_podman_state)
  - [`quadlet_podman_service_state`](#variable-quadlet_podman_service_state)
  - [`quadlet_podman_user`](#variable-quadlet_podman_user)
  - [`quadlet_podman_units`](#variable-quadlet_podman_units)
    - [`quadlet_podman_units['name']`](#variable-quadlet_podman_units-sub-name)
    - [`quadlet_podman_units['type']`](#variable-quadlet_podman_units-sub-type)
    - [`quadlet_podman_units['sections']`](#variable-quadlet_podman_units-sub-sections)
    - [`quadlet_podman_units['service_state']`](#variable-quadlet_podman_units-sub-service_state)
  - [`quadlet_podman_systemd_units`](#variable-quadlet_podman_systemd_units)
    - [`quadlet_podman_systemd_units['name']`](#variable-quadlet_podman_systemd_units-sub-name)
    - [`quadlet_podman_systemd_units['sections']`](#variable-quadlet_podman_systemd_units-sub-sections)
    - [`quadlet_podman_systemd_units['service_state']`](#variable-quadlet_podman_systemd_units-sub-service_state)
  - [`quadlet_podman_unit_defaults`](#variable-quadlet_podman_unit_defaults)
    - [`quadlet_podman_unit_defaults['container']`](#variable-quadlet_podman_unit_defaults-sub-container)
    - [`quadlet_podman_unit_defaults['pod']`](#variable-quadlet_podman_unit_defaults-sub-pod)
    - [`quadlet_podman_unit_defaults['volume']`](#variable-quadlet_podman_unit_defaults-sub-volume)
    - [`quadlet_podman_unit_defaults['network']`](#variable-quadlet_podman_unit_defaults-sub-network)
    - [`quadlet_podman_unit_defaults['kube']`](#variable-quadlet_podman_unit_defaults-sub-kube)
    - [`quadlet_podman_unit_defaults['image']`](#variable-quadlet_podman_unit_defaults-sub-image)
    - [`quadlet_podman_unit_defaults['build']`](#variable-quadlet_podman_unit_defaults-sub-build)
    - [`quadlet_podman_unit_defaults['artifact']`](#variable-quadlet_podman_unit_defaults-sub-artifact)
  - [`quadlet_podman_secrets`](#variable-quadlet_podman_secrets)
    - [`quadlet_podman_secrets['name']`](#variable-quadlet_podman_secrets-sub-name)
    - [`quadlet_podman_secrets['value']`](#variable-quadlet_podman_secrets-sub-value)
    - [`quadlet_podman_secrets['state']`](#variable-quadlet_podman_secrets-sub-state)
  - [`quadlet_podman_registry_auth`](#variable-quadlet_podman_registry_auth)
    - [`quadlet_podman_registry_auth['registry']`](#variable-quadlet_podman_registry_auth-sub-registry)
    - [`quadlet_podman_registry_auth['username']`](#variable-quadlet_podman_registry_auth-sub-username)
    - [`quadlet_podman_registry_auth['password']`](#variable-quadlet_podman_registry_auth-sub-password)
  - [`quadlet_podman_directories`](#variable-quadlet_podman_directories)
    - [`quadlet_podman_directories['path']`](#variable-quadlet_podman_directories-sub-path)
    - [`quadlet_podman_directories['owner']`](#variable-quadlet_podman_directories-sub-owner)
    - [`quadlet_podman_directories['group']`](#variable-quadlet_podman_directories-sub-group)
    - [`quadlet_podman_directories['mode']`](#variable-quadlet_podman_directories-sub-mode)
    - [`quadlet_podman_directories['state']`](#variable-quadlet_podman_directories-sub-state)
  - [`quadlet_podman_files`](#variable-quadlet_podman_files)
    - [`quadlet_podman_files['dest']`](#variable-quadlet_podman_files-sub-dest)
    - [`quadlet_podman_files['content']`](#variable-quadlet_podman_files-sub-content)
    - [`quadlet_podman_files['owner']`](#variable-quadlet_podman_files-sub-owner)
    - [`quadlet_podman_files['group']`](#variable-quadlet_podman_files-sub-group)
    - [`quadlet_podman_files['mode']`](#variable-quadlet_podman_files-sub-mode)
    - [`quadlet_podman_files['state']`](#variable-quadlet_podman_files-sub-state)
  - [`quadlet_podman_min_podman_version`](#variable-quadlet_podman_min_podman_version)
<!-- ANSIBLE DOCSMITH TOC END -->
- [Dependencies](#dependencies)
- [Compatibility](#compatibility)
- [External requirements](#requirements)



## Example playbooks, using this role<a id="examples"></a>

One role invocation manages one application: a coherent set of Quadlet units (plus secrets, directories and files). Deploying a rootful application:

```yaml
---

- name: "Initialize the foundata.podman.quadlet role"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Trigger invocation of the foundata.podman.quadlet role"
      ansible.builtin.include_role:
        name: "foundata.podman.quadlet"
      vars:
        quadlet_podman_app: "exampleapp"
        quadlet_podman_secrets:
          - name: "exampleapp_db_password"
            value: "{{ vault_exampleapp_db_password }}"
        quadlet_podman_unit_defaults:
          container:
            Service:
              Restart: "always"
              TimeoutStartSec: 300
            Install:
              WantedBy:
                - "multi-user.target"
        quadlet_podman_units:
          - name: "exampleapp"
            type: "pod"
            sections:
              Unit:
                Description: "Example application pod"
              Pod:
                PodName: "exampleapp"
                PublishPort:
                  - "127.0.0.1:8080:8080"
              Install:
                WantedBy:
                  - "multi-user.target"
          - name: "exampleapp-db-data"
            type: "volume"
            sections:
              Volume:
                VolumeName: "exampleapp-db-data"
          - name: "exampleapp-db"
            type: "container"
            sections:
              Unit:
                Description: "Example application database"
                After:
                  - "exampleapp-db-data" # gets expanded to exampleapp-db-data-volume.service
              Container:
                ContainerName: "exampleapp-db"
                Image: "docker.io/library/postgres:18"
                Pod: "exampleapp.pod"
                Volume:
                  - "exampleapp-db-data.volume:/var/lib/postgresql"
                Secret:
                  - "exampleapp_db_password,type=env,target=POSTGRES_PASSWORD"
                AutoUpdate: "registry"
```

The same application deployed rootless is a one-line change; the account must exist and be enabled for rootless Podman usage beforehand (see `host_podman_rootless_users` of the `foundata.podman.host` role):

```yaml
      vars:
        quadlet_podman_app: "exampleapp"
        quadlet_podman_user: "exampleapp" # empty (the default): rootful
        # [...]
```

Removing an application (stops the services, removes units, secrets and managed files; host directories and named volumes are kept to protect persistent data):

```yaml
---

- name: "Initialize the foundata.podman.quadlet role"
  hosts: localhost
  gather_facts: false
  tasks:

    - name: "Trigger invocation of the foundata.podman.quadlet role"
      ansible.builtin.include_role:
        name: "foundata.podman.quadlet"
      vars:
        quadlet_podman_app: "exampleapp"
        quadlet_podman_state: "absent"
```



## Run-once containers and timers<a id="run-once"></a>

Containers that run a job and exit (initialization, migrations, backups) need different handling than daemons. A run-once service ends up `inactive (dead)` after success, so a naive "ensure started" would re-run the job on every Ansible run and permanently report `changed`. The role supports the two common shapes; do not mix them up, they are mutually exclusive:

**Run once per deployment and configuration change** (e.g. database migrations): keep the unit managed and set `RemainAfterExit` in its `Service` section. The service parks in `active (exited)` after success, so later runs are a no-op (`changed=0`), while the restart-on-change handler still re-runs the job whenever the application's units or managed files change.

```yaml
quadlet_podman_units:
  - name: "exampleapp-init"
    type: "container"
    sections:
      Container:
        ContainerName: "exampleapp-init"
        Image: "registry.example.com/exampleapp:6"
        Exec: "exampleapp-init"
      Service:
        Restart: "on-failure"
        RemainAfterExit: true
      Install:
        WantedBy:
          - "multi-user.target"
```

**Run on a schedule** (e.g. backups): set the unit's `service_state` to `unmanaged` so the role only deploys the file (it neither starts the job on converge nor stops an in-flight run), and ship the timer as a plain systemd unit of the same application via `quadlet_podman_systemd_units`. Do NOT use `RemainAfterExit` here: systemd does not re-trigger a unit that is still active, so the timer would fire into a no-op. Keep an `Install` section on the container if the job should additionally run once per boot, or set `Install: ~` to drop one inherited from `quadlet_podman_unit_defaults`. If the job container is a pod member, also set `StartWithPod: false` in its `Container` section: the generated pod service pulls its members in via `Wants=`, so without it the job runs at every pod (re)start no matter what `service_state` and `Install` say.

```yaml
quadlet_podman_units:
  - name: "exampleapp-backup"
    type: "container"
    service_state: "unmanaged"
    sections:
      Container:
        ContainerName: "exampleapp-backup"
        Image: "registry.example.com/exampleapp:6"
        Exec: "exampleapp-backup"
      Service:
        Restart: "no"
        TimeoutStartSec: 3600
quadlet_podman_systemd_units:
  - name: "exampleapp-backup.timer"
    sections:
      Unit:
        Description: "Example application backup timer"
      Timer:
        Unit: "exampleapp-backup" # gets expanded to exampleapp-backup.service
        OnCalendar: "*-*-* 03:00:00"
        RandomizedDelaySec: "15m"
        Persistent: true
      Install:
        WantedBy:
          - "timers.target"
```

Plain units from `quadlet_podman_systemd_units` are managed like the application's Quadlet files (ownership marker, cleanup of de-listed units, removal on `quadlet_podman_state: "absent"`) and support real `systemctl enable`/`disable`. They also cover non-container helpers, e.g. a oneshot `.service` wrapping a host-side script plus its `.timer`.



## Rootless applications: data, UID mapping and SELinux<a id="rootless-notes"></a>

**Prefer named volumes for persistent data.** For rootless applications, named volumes (`.volume` units) live below the account's home directory and Podman handles the user namespace ownership for you. Combined with a central home directory (e.g. `/var/lib/podman/<application>`, see the `foundata.podman.host` role's README section on centralizing rootless storage), all persistent data stays in one predictable tree.

**Bind mounts need UID mapping awareness.** A rootless container's UIDs are mapped to the account's subordinate ID range on the host: a process running as UID 999 *inside* the container writes files owned by `<subuid start> + 999` *on the host*. A bind-mounted host directory created via `quadlet_podman_directories` is owned by the account itself, which the container may not be able to write to. Options, in order of preference:

- Use a named volume instead (no mapping issues).
- Add the `:U` volume option (`Volume: "/var/lib/podman/exampleapp/data:/data:U"`) so Podman chowns the content to the container user on start.
- Pre-create the content with the mapped owner (e.g. via `podman unshare chown` or by computing `<subuid start> + <container UID>`).

**SELinux labels on bind mounts.** On SELinux platforms (RedHat family, openSUSE Leap 16), bind-mounted paths usually need the `:z` (shared between containers) or `:Z` (private to one container) volume option so Podman relabels the content. Use `:Z` only for paths dedicated to a single container - it makes the content inaccessible to others. Never bind-mount system paths with `:z`/`:Z` (the relabeling is applied to the host content).

**Rotated secrets need container recreation.** Secrets injected as environment variables (`type=env`) are set at container *creation*. After rotating a secret (item `force: true` in `quadlet_podman_secrets`), restart is not enough; the container must be recreated (e.g. `podman rm --force <name>` followed by a service start, or a host reboot).



## Supported tags<a id="tags"></a>

It might be useful and faster to only call parts of the role by using tags:

- `quadlet_podman_setup`: Manage basic resources, such as packages or service users.
- `quadlet_podman_config`: Manage settings, such as adapting or creating configuration files.
- `quadlet_podman_service`: Manage services and daemons, such as running states and service boot configurations.

There are also tags that are generally not intended to be called directly but are included for completeness and to cover edge cases:

- `quadlet_podman_always`, `always`: Tasks needed by the role itself for internal role setup and the Ansible environment.


<!-- ANSIBLE DOCSMITH MAIN START -->

## Role variables<a id="variables"></a>

Main entry point for the foundata.podman.quadlet role

The following variables can be configured for this role:

| Variable | Type | Required | Default | Description (abstract) |
|----------|------|----------|---------|------------------------|
| `quadlet_podman_app` | `str` | Yes | N/A | Identifier of the application this set of Quadlet units belongs to (e.g. `zammad`, `keycloak`). Allowed characters: lowercase letters, digits, `_` and `-`.<br><br>One role invocation manages one application: a coherent set of Quadlet units (plus […](#variable-quadlet_podman_app) |
| `quadlet_podman_state` | `str` | No | `"present"` | Determines whether the managed resources should be `present` or `absent`.<br><br>`present` ensures that the application's Quadlet units, secrets, directories and files are deployed and its services are managed.<br><br>`absent` stops the application's […](#variable-quadlet_podman_state) |
| `quadlet_podman_service_state` | `str` | No | `"enabled"` | Defines the status of the service(s).<br><br>`enabled`: Service is running and will start automatically at boot.<br><br>`disabled`: Service is stopped and will not start automatically at boot.<br><br>`running`: Service is running but will not start […](#variable-quadlet_podman_service_state) |
| `quadlet_podman_user` | `str` | No | `""` | Name of the user account to deploy the application for, selecting rootless or rootful operation:<br><br>- Empty (the default): rootful. Quadlet units are placed below `/etc/containers/systemd/` and the generated services run as system services. - Set […](#variable-quadlet_podman_user) |
| `quadlet_podman_units` | `list` | No | `[]` | List of the Quadlet units that make up the application. Each list item describes one Quadlet file and is rendered from its `sections` dictionary; the full set of available sections and settings is documented in `man podman-systemd.unit` or […](#variable-quadlet_podman_units) |
| `quadlet_podman_systemd_units` | `list` | No | `[]` | List of plain systemd units that belong to the application but are not Quadlet types, typically timers that trigger run-once containers or small oneshot helper services. The files are written to `/etc/systemd/system/` (rootful) or […](#variable-quadlet_podman_systemd_units) |
| `quadlet_podman_unit_defaults` | `dict` | No | `{}` | Default sections and settings applied to every unit in `quadlet_podman_units`, keyed by unit type. Use this to avoid repeating common settings (e.g. restart policy or boot enablement) in each unit; the per-unit `sections` are deep-merged on top and […](#variable-quadlet_podman_unit_defaults) |
| `quadlet_podman_secrets` | `list` | No | `[]` | List of Podman secrets to manage for the application, created in the scope selected by `quadlet_podman_user` (rootful or the given user's rootless scope).<br><br>Declared secrets are authoritative for existence and content: the role compares the […](#variable-quadlet_podman_secrets) |
| `quadlet_podman_registry_auth` | `list` | No | `[]` | List of container registries to log into before images are pulled, in the scope selected by `quadlet_podman_user`. Needed for images from private registries; the credentials are also used by the automatic image updates […](#variable-quadlet_podman_registry_auth) |
| `quadlet_podman_directories` | `list` | No | `[]` | List of host directories to create before the application's services are started, typically used as bind-mount sources or to hold configuration created via `quadlet_podman_files`.<br><br>For rootless applications keep in mind that a container process […](#variable-quadlet_podman_directories) |
| `quadlet_podman_files` | `list` | No | `[]` | List of files to manage for the application, e.g. environment files referenced via `EnvironmentFile` or configuration files bind-mounted into containers. Declared files are owned by the application: content and attributes are enforced, a pre-existing […](#variable-quadlet_podman_files) |
| `quadlet_podman_min_podman_version` | `str` | No | N/A | Minimum Podman version this application requires (e.g. `5.7.0` when using `artifact` units or Quadlet settings introduced after the role's own baseline). If the Podman version on the target host is lower, the role fails early with a clear message […](#variable-quadlet_podman_min_podman_version) |

### `quadlet_podman_app`<a id="variable-quadlet_podman_app"></a>

[*⇑ Back to ToC ⇑*](#toc)

Identifier of the application this set of Quadlet units belongs to
(e.g. `zammad`, `keycloak`). Allowed characters: lowercase letters,
digits, `_` and `-`.

One role invocation manages one application: a coherent set of Quadlet
units (plus secrets, directories and files) that is deployed, updated
and removed together. Include the role multiple times to manage several
applications, each with a distinct `quadlet_podman_app` value.

The role marks every Quadlet file it writes with this identifier and
uses the marker to re-identify its files later: units removed from
`quadlet_podman_units` get cleaned up on the next run, and
`quadlet_podman_state: "absent"` removes all of the application's
units even if the current list is empty or different. Files belonging
to other applications or not managed by this role are never touched.

- **Type**: `str`
- **Required**: Yes



### `quadlet_podman_state`<a id="variable-quadlet_podman_state"></a>

[*⇑ Back to ToC ⇑*](#toc)

Determines whether the managed resources should be `present` or `absent`.

`present` ensures that the application's Quadlet units, secrets,
directories and files are deployed and its services are managed.

`absent` stops the application's services and removes its Quadlet
units, secrets and managed files. Directories from
`quadlet_podman_directories` and named volumes are NOT removed to
protect persistent application data; remove those manually or with
dedicated tasks if really needed.

If the rootless account was already deleted, the application's unit
files are still removed from the recorded (possibly preserved)
directories together with the manifest. Podman secrets and
container storage inside a preserved home cannot be removed
selectively without the account; remove the home directory itself
if nothing in it is needed anymore.

- **Type**: `str`
- **Required**: No
- **Default**: `"present"`
- **Choices**: `present`, `absent`



### `quadlet_podman_service_state`<a id="variable-quadlet_podman_service_state"></a>

[*⇑ Back to ToC ⇑*](#toc)

Defines the status of the service(s).

`enabled`: Service is running and will start automatically at boot.

`disabled`: Service is stopped and will not start automatically at boot.

`running`: Service is running but will not start automatically at boot.
This can be used to start a service on the first Ansible run without
enabling it for boot.

`unmanaged`: Ansible does not manage the service at all: both the
running state and the boot (enablement) state are left exactly as they
are. This is primarily useful when services are monitored and managed
by systems other than Ansible.

The singular form (`service`) is used for simplicity. However, the defined
status applies to all services if multiple are being managed by this role.
Individual units can deviate via their own `service_state` entry in
`quadlet_podman_units` or `quadlet_podman_systemd_units` (e.g. a run-once
container that only its timer may start).

The role manages the systemd services generated from the units in
`quadlet_podman_units`, in dependency order (e.g. a pod before its member
containers). Note that systemd units generated from Quadlet files cannot
be enabled or disabled via `systemctl`; whether a service starts at boot
is governed by the `Install` section of its Quadlet unit (usually
`WantedBy: "multi-user.target"` or `WantedBy: "default.target"`). For
`enabled`, ensure the relevant units carry an `Install` section; for
`disabled`, omit it (the role still stops running services). Plain units
from `quadlet_podman_systemd_units` are real unit files, so for them
`enabled` and `disabled` do use `systemctl enable` / `disable`.
For plain units the semantics are explicit: `enabled` requires an
`Install` section (the role fails otherwise); `running` and
`disabled` actively disable the unit, so an enablement symlink left
behind by an earlier `Install`-carrying version of the file cannot
survive a state that demands no boot start.

- **Type**: `str`
- **Required**: No
- **Default**: `"enabled"`
- **Choices**: `enabled`, `disabled`, `running`, `unmanaged`



### `quadlet_podman_user`<a id="variable-quadlet_podman_user"></a>

[*⇑ Back to ToC ⇑*](#toc)

Name of the user account to deploy the application for, selecting
rootless or rootful operation:

- Empty (the default): rootful. Quadlet units are placed below
  `/etc/containers/systemd/` and the generated services run as
  system services.
- Set to a username: rootless. Quadlet units are placed below
  `~/.config/containers/systemd/` of that account, secrets and
  registry logins are created in its scope, and the generated
  services run in its systemd user instance (`systemctl --user`).

The account must already exist and must be enabled for rootless Podman
usage, including systemd lingering (see `host_podman_rootless_users` of
the `foundata.podman.host` role). The role fails early if these
prerequisites are missing.

- **Type**: `str`
- **Required**: No
- **Default**: `""`



### `quadlet_podman_units`<a id="variable-quadlet_podman_units"></a>

[*⇑ Back to ToC ⇑*](#toc)

List of the Quadlet units that make up the application. Each list item
describes one Quadlet file and is rendered from its `sections`
dictionary; the full set of available sections and settings is
documented in `man podman-systemd.unit` or
https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html

Rendering rules:

- Sections are rendered in a canonical order (`Unit`, then the
  type-specific section such as `Container` or `Pod`, then `Service`
  and `Install`); any other section is passed through as-is.
- Scalar values are rendered as single `Key=value` lines.
- List values are rendered as repeated `Key=value` lines (e.g. for
  `Volume`, `PublishPort`, `Environment`).
- Boolean values are rendered as `true`/`false`.
- Values of the assignment-list settings `Environment`, `Label` and
  `Annotation` are quoted for systemd automatically when they
  contain whitespace, quotes or backslashes, so one list item is
  always exactly one assignment (e.g. one environment variable,
  even with spaces in its value). Provide such items unquoted and
  do not put several assignments into one item. All other settings
  (e.g. `Exec`, `PodmanArgs`) are rendered verbatim because their
  grammar word-splits deliberately; quote parts of them yourself
  where needed.
- A `null` value drops the section or setting entirely, which is
  mainly useful to unset something a unit would otherwise inherit
  from `quadlet_podman_unit_defaults` (e.g. `Install: ~` on a unit
  that must not start at boot).

The name of the systemd service generated by a unit depends on its
type: `<name>.service` for `container` and `kube`,
`<name>-<type>.service` for all other types (e.g. a pod `zammad`
becomes `zammad-pod.service`). A `ServiceName` setting in the type
section overrides this (Quadlet generates `<ServiceName>.service`);
the role's service management, dependency expansion and cleanup
follow the override.

A container's `Pod` setting has to reference the Quadlet file name
including the `.pod` suffix (e.g. `Pod: "shared.pod"`), as Podman
demands. The pod may belong to another application of the same
scope; Podman's generator creates the service relationship itself.
The role only orders starts for pods of the same application, so
converge the pod's application first.

Unit file names and generated service names form ONE scope-wide
systemd namespace across all applications: collisions with the
units or services another application's manifest records are
rejected, and a destination file that already exists is only
overwritten when it carries this application's ownership marker
(files of other applications and unmarked administrator files are
rejected, never touched).

As a convenience, values in the `Unit`
section's `After`, `Requires`, `Wants`, `BindsTo`, `PartOf` and
`Before` settings that match the `name` of another unit in this list
are expanded to that unit's generated service name automatically;
values containing a dot (e.g. `network-online.target`) are passed
through verbatim.

Example (a pod with one member container and a named volume):

```yaml
quadlet_podman_units:
  - name: "zammad"
    type: "pod"
    sections:
      Unit:
        Description: "Zammad Pod"
      Pod:
        PodName: "zammad"
        PublishPort:
          - "127.0.0.1:8080:8080"
      Install:
        WantedBy: "multi-user.target"
  - name: "zammad-psql-data"
    type: "volume"
    sections:
      Volume:
        VolumeName: "zammad-psql-data"
  - name: "zammad-psql"
    type: "container"
    sections:
      Unit:
        Description: "Zammad PostgreSQL Database"
        After:
          - "zammad-psql-data"
      Container:
        ContainerName: "zammad-psql"
        Image: "docker.io/postgres:18"
        Pod: "zammad.pod"
        Volume: "zammad-psql-data.volume:/var/lib/postgresql"
        AutoUpdate: "registry"
      Service:
        Restart: "always"
      Install:
        WantedBy: "multi-user.target"
```

- **Type**: `list`
- **Required**: No
- **Default**: `[]`
- **List Elements**: `dict`

#### `quadlet_podman_units['name']`<a id="variable-quadlet_podman_units-sub-name"></a>

[*⇑ Back to ToC ⇑*](#toc)

Name of the Quadlet unit (the filename without extension, e.g.
`zammad-psql` results in `zammad-psql.container`). Also the basis
of the generated systemd service name. Must be unique within
`quadlet_podman_units`. Allowed characters: letters, digits, `_`,
`.` and `-`.

- **Type**: `str`
- **Required**: Yes

#### `quadlet_podman_units['type']`<a id="variable-quadlet_podman_units-sub-type"></a>

[*⇑ Back to ToC ⇑*](#toc)

Type of the Quadlet unit, determining the file extension and the
expected type-specific section (e.g. `container` expects a
`Container` section).

Note: `artifact` requires Podman 5.7 or later (see
`quadlet_podman_min_podman_version`).

- **Type**: `str`
- **Required**: Yes
- **Choices**: `container`, `pod`, `volume`, `network`, `kube`, `image`, `build`, `artifact`

#### `quadlet_podman_units['sections']`<a id="variable-quadlet_podman_units-sub-sections"></a>

[*⇑ Back to ToC ⇑*](#toc)

Content of the Quadlet unit as a dictionary of sections. Keys are
section names (e.g. `Unit`, `Container`, `Pod`, `Volume`,
`Network`, `Service`, `Install`), values are dictionaries of the
settings within that section. See the description of
`quadlet_podman_units` for the rendering rules and an example.

- **Type**: `dict`
- **Required**: Yes

#### `quadlet_podman_units['service_state']`<a id="variable-quadlet_podman_units-sub-service_state"></a>

[*⇑ Back to ToC ⇑*](#toc)

Per-unit override of `quadlet_podman_service_state`, with the
same values and semantics. If unset, the unit follows the
application-wide value.

The main use case is `unmanaged` for a run-once container that
only its systemd timer (see `quadlet_podman_systemd_units`) or
a boot-time `Install` section may start: the role deploys the
unit file but neither starts the service on converge (no job
run per deployment, no `changed` report) nor stops it (an
in-flight run survives converges). Prefer `unmanaged` over
`disabled` for such units, `disabled` stops the service on
every run and would kill an in-flight timer-triggered run.

Caution for pod members: the generated pod service pulls its
member containers in via `Wants=`, so a member starts at every
pod (re)start regardless of `unmanaged` and `Install`. A
timer-only pod member additionally needs
`StartWithPod: false` in its `Container` section.

For a run-once container that SHOULD run once per deployment
and configuration change (e.g. database migrations), no
override is needed: keep the unit managed and set
`RemainAfterExit: true` in its `Service` section instead. The
service then parks in `active (exited)` after success, so the
role's start task is a no-op on later runs while the
restart-on-change handler still re-runs it. Do not combine
`RemainAfterExit` with a timer: systemd does not re-trigger a
unit that is still active.

- **Type**: `str`
- **Required**: No
- **Choices**: `enabled`, `disabled`, `running`, `unmanaged`



### `quadlet_podman_systemd_units`<a id="variable-quadlet_podman_systemd_units"></a>

[*⇑ Back to ToC ⇑*](#toc)

List of plain systemd units that belong to the application but are
not Quadlet types, typically timers that trigger run-once containers
or small oneshot helper services. The files are written to
`/etc/systemd/system/` (rootful) or `~/.config/systemd/user/` of the
`quadlet_podman_user` account (rootless) and are managed exactly like
the application's Quadlet files: marked with the application
identifier, removed again when de-listed and removed by
`quadlet_podman_state: "absent"`.

Each item's `sections` dictionary follows the same rendering rules as
`quadlet_podman_units` (`quadlet_podman_unit_defaults` does not
apply). As a convenience, the `Unit` setting of `Timer` and `Path`
sections, the `Service` setting of `Socket` sections and the
dependency settings of the `Unit` section expand unit names from
`quadlet_podman_units` to their generated service names
automatically.

Unlike Quadlet-generated services, these are real unit files, so
`enabled`/`disabled` (from the item's `service_state` or the
application-wide `quadlet_podman_service_state`) use
`systemctl enable`/`disable` and honor the unit's `Install` section.
Changed unit files get a daemon-reload and managed units are
restarted (so e.g. a changed `OnCalendar` takes effect; with
`Persistent: true` a schedule moved into the past fires once
immediately).

Example (a timer starting the generated service of a run-once
container named `example-job` every night):

```yaml
quadlet_podman_systemd_units:
  - name: "example-job.timer"
    sections:
      Unit:
        Description: "Example nightly job timer"
      Timer:
        Unit: "example-job"
        OnCalendar: "*-*-* 03:00:00"
        RandomizedDelaySec: "15m"
        Persistent: true
      Install:
        WantedBy: "timers.target"
```

- **Type**: `list`
- **Required**: No
- **Default**: `[]`
- **List Elements**: `dict`

#### `quadlet_podman_systemd_units['name']`<a id="variable-quadlet_podman_systemd_units-sub-name"></a>

[*⇑ Back to ToC ⇑*](#toc)

File name of the systemd unit including its type suffix (e.g.
`example-job.timer`); one of `.timer`, `.service`, `.target`,
`.path` or `.socket`. Must be unique and must not collide with a
unit file or generated service name of `quadlet_podman_units`.
Allowed characters: letters, digits, `_`, `.` and `-`.

Note for `.socket` units: systemd only starts a socket whose
service unit exists, so pair it with a `.service` in this list
(or name it after an existing service).

- **Type**: `str`
- **Required**: Yes

#### `quadlet_podman_systemd_units['sections']`<a id="variable-quadlet_podman_systemd_units-sub-sections"></a>

[*⇑ Back to ToC ⇑*](#toc)

Content of the unit as a dictionary of sections (e.g. `Unit`,
`Timer`, `Service`, `Install`), following the rendering rules of
`quadlet_podman_units`.

- **Type**: `dict`
- **Required**: Yes

#### `quadlet_podman_systemd_units['service_state']`<a id="variable-quadlet_podman_systemd_units-sub-service_state"></a>

[*⇑ Back to ToC ⇑*](#toc)

Per-unit override of `quadlet_podman_service_state`, with the
same values. If unset, the unit follows the application-wide
value. Use `unmanaged` for units that are only started by other
units (e.g. a oneshot service triggered by a timer of the same
application).

- **Type**: `str`
- **Required**: No
- **Choices**: `enabled`, `disabled`, `running`, `unmanaged`



### `quadlet_podman_unit_defaults`<a id="variable-quadlet_podman_unit_defaults"></a>

[*⇑ Back to ToC ⇑*](#toc)

Default sections and settings applied to every unit in
`quadlet_podman_units`, keyed by unit type. Use this to avoid
repeating common settings (e.g. restart policy or boot enablement)
in each unit; the per-unit `sections` are deep-merged on top and take
precedence.

Example (restart all containers of the application on failure and
start them at boot):

```yaml
quadlet_podman_unit_defaults:
  container:
    Service:
      Restart: "always"
      TimeoutStartSec: 300
    Install:
      WantedBy: "multi-user.target"
```

- **Type**: `dict`
- **Required**: No
- **Default**: `{}`

#### `quadlet_podman_unit_defaults['container']`<a id="variable-quadlet_podman_unit_defaults-sub-container"></a>

[*⇑ Back to ToC ⇑*](#toc)

Default sections for all units of type `container`. Same structure
as the per-unit `sections`.

- **Type**: `dict`
- **Required**: No

#### `quadlet_podman_unit_defaults['pod']`<a id="variable-quadlet_podman_unit_defaults-sub-pod"></a>

[*⇑ Back to ToC ⇑*](#toc)

Default sections for all units of type `pod`. Same structure as
the per-unit `sections`.

- **Type**: `dict`
- **Required**: No

#### `quadlet_podman_unit_defaults['volume']`<a id="variable-quadlet_podman_unit_defaults-sub-volume"></a>

[*⇑ Back to ToC ⇑*](#toc)

Default sections for all units of type `volume`. Same structure as
the per-unit `sections`.

- **Type**: `dict`
- **Required**: No

#### `quadlet_podman_unit_defaults['network']`<a id="variable-quadlet_podman_unit_defaults-sub-network"></a>

[*⇑ Back to ToC ⇑*](#toc)

Default sections for all units of type `network`. Same structure
as the per-unit `sections`.

- **Type**: `dict`
- **Required**: No

#### `quadlet_podman_unit_defaults['kube']`<a id="variable-quadlet_podman_unit_defaults-sub-kube"></a>

[*⇑ Back to ToC ⇑*](#toc)

Default sections for all units of type `kube`. Same structure as
the per-unit `sections`.

- **Type**: `dict`
- **Required**: No

#### `quadlet_podman_unit_defaults['image']`<a id="variable-quadlet_podman_unit_defaults-sub-image"></a>

[*⇑ Back to ToC ⇑*](#toc)

Default sections for all units of type `image`. Same structure as
the per-unit `sections`.

- **Type**: `dict`
- **Required**: No

#### `quadlet_podman_unit_defaults['build']`<a id="variable-quadlet_podman_unit_defaults-sub-build"></a>

[*⇑ Back to ToC ⇑*](#toc)

Default sections for all units of type `build`. Same structure as
the per-unit `sections`.

- **Type**: `dict`
- **Required**: No

#### `quadlet_podman_unit_defaults['artifact']`<a id="variable-quadlet_podman_unit_defaults-sub-artifact"></a>

[*⇑ Back to ToC ⇑*](#toc)

Default sections for all units of type `artifact`. Same structure
as the per-unit `sections`.

- **Type**: `dict`
- **Required**: No



### `quadlet_podman_secrets`<a id="variable-quadlet_podman_secrets"></a>

[*⇑ Back to ToC ⇑*](#toc)

List of Podman secrets to manage for the application, created in the
scope selected by `quadlet_podman_user` (rootful or the given user's
rootless scope).

Declared secrets are authoritative for existence and content: the
role compares the current value (read back internally, never logged
or shown in diffs) and replaces the secret when it differs, so
rotation is implicit; change the declared value and converge.
Declaring a pre-existing secret adopts it: its value is reconciled
to the declared one and de-listing it later removes it. A secret
name belongs to exactly one application per scope; conflicting
claims by another application's manifest are rejected. Podman
copies secret data into containers at creation time, so a changed
value also restarts the application services (their containers are
recreated on start, which re-materializes the secret).

Reference the secrets from Quadlet units via the `Secret` setting of
the `Container` section, e.g.
`Secret: "zammad_db_password,type=env,target=POSTGRES_PASSWORD"`.

The role records the names of the secrets it manages in a
per-application manifest (root-owned, below `/var/lib/ansible-podman-quadlet/`): secrets renamed or
removed from this list get removed again on the next run, and
`quadlet_podman_state: "absent"` removes all of the application's
secrets even when the current list is empty or different.

Example (reference a vaulted variable instead of a literal value in
real deployments; note that Jinja expressions cannot be shown in this
description as Ansible templates it during argument validation):

```yaml
quadlet_podman_secrets:
  - name: "zammad_db_password"
    value: "place-a-vaulted-value-here"
```

- **Type**: `list`
- **Required**: No
- **Default**: `[]`
- **List Elements**: `dict`

#### `quadlet_podman_secrets['name']`<a id="variable-quadlet_podman_secrets-sub-name"></a>

[*⇑ Back to ToC ⇑*](#toc)

Name of the Podman secret. Allowed characters: letters, digits,
`_`, `.` and `-`.

- **Type**: `str`
- **Required**: Yes

#### `quadlet_podman_secrets['value']`<a id="variable-quadlet_podman_secrets-sub-value"></a>

[*⇑ Back to ToC ⇑*](#toc)

Value of the secret. Required when `state` is `present`. Trailing
newlines are stripped (a common source of subtle authentication
failures when values come from files or lookups).

- **Type**: `str`
- **Required**: No
- **Sensitive**: Yes (`no_log`, values are masked in logs)

#### `quadlet_podman_secrets['state']`<a id="variable-quadlet_podman_secrets-sub-state"></a>

[*⇑ Back to ToC ⇑*](#toc)

`present` ensures the secret exists with exactly the declared
value (a differing existing secret is replaced and the
application services are restarted), `absent` removes it.

- **Type**: `str`
- **Required**: No
- **Default**: `"present"`
- **Choices**: `present`, `absent`



### `quadlet_podman_registry_auth`<a id="variable-quadlet_podman_registry_auth"></a>

[*⇑ Back to ToC ⇑*](#toc)

List of container registries to log into before images are pulled,
in the scope selected by `quadlet_podman_user`. Needed for images
from private registries; the credentials are also used by the
automatic image updates (`AutoUpdate=registry`).

The role records its logins in a per-application manifest
(root-owned, below `/var/lib/ansible-podman-quadlet/`): registries
removed from this list are logged out
again on the next run, and `quadlet_podman_state: "absent"` logs out
of all recorded registries, so no credentials linger after removal.
Note that the authentication file is shared per scope (one per
rootless account, `/etc/containers/auth.json` for rootful). The role
therefore only logs out of a registry when no other application's
manifest in the same scope still declares it: shared credentials
stay in place until the last application using them is removed or
stops declaring the registry.

Example (reference a vaulted variable instead of a literal value in
real deployments):

```yaml
quadlet_podman_registry_auth:
  - registry: "registry.example.com"
    username: "deploy"
    password: "place-a-vaulted-value-here"
```

- **Type**: `list`
- **Required**: No
- **Default**: `[]`
- **List Elements**: `dict`

#### `quadlet_podman_registry_auth['registry']`<a id="variable-quadlet_podman_registry_auth-sub-registry"></a>

[*⇑ Back to ToC ⇑*](#toc)

Registry host to authenticate against (e.g.
`registry.example.com`).

- **Type**: `str`
- **Required**: Yes

#### `quadlet_podman_registry_auth['username']`<a id="variable-quadlet_podman_registry_auth-sub-username"></a>

[*⇑ Back to ToC ⇑*](#toc)

Username for the registry login.

- **Type**: `str`
- **Required**: Yes

#### `quadlet_podman_registry_auth['password']`<a id="variable-quadlet_podman_registry_auth-sub-password"></a>

[*⇑ Back to ToC ⇑*](#toc)

Password or token for the registry login.

- **Type**: `str`
- **Required**: Yes
- **Sensitive**: Yes (`no_log`, values are masked in logs)



### `quadlet_podman_directories`<a id="variable-quadlet_podman_directories"></a>

[*⇑ Back to ToC ⇑*](#toc)

List of host directories to create before the application's services
are started, typically used as bind-mount sources or to hold
configuration created via `quadlet_podman_files`.

For rootless applications keep in mind that a container process UID
maps to a subordinate UID on the host; directories written to by the
container may therefore need ownership adjustments or the `:U` volume
mount option. See the role's README for details.

Directories are NOT removed when `quadlet_podman_state` is `absent`
(they may contain persistent application data); set an item's `state`
to `absent` to remove it explicitly.

Example:

```yaml
quadlet_podman_directories:
  - path: "/var/lib/podman/zammad/config"
```

- **Type**: `list`
- **Required**: No
- **Default**: `[]`
- **List Elements**: `dict`

#### `quadlet_podman_directories['path']`<a id="variable-quadlet_podman_directories-sub-path"></a>

[*⇑ Back to ToC ⇑*](#toc)

Absolute path of the directory.

- **Type**: `str`
- **Required**: Yes

#### `quadlet_podman_directories['owner']`<a id="variable-quadlet_podman_directories-sub-owner"></a>

[*⇑ Back to ToC ⇑*](#toc)

Owner of the directory. Defaults to `root` for rootful
applications and to `quadlet_podman_user` for rootless ones.

- **Type**: `str`
- **Required**: No

#### `quadlet_podman_directories['group']`<a id="variable-quadlet_podman_directories-sub-group"></a>

[*⇑ Back to ToC ⇑*](#toc)

Group of the directory. Defaults to the primary group of the
directory's owner.

- **Type**: `str`
- **Required**: No

#### `quadlet_podman_directories['mode']`<a id="variable-quadlet_podman_directories-sub-mode"></a>

[*⇑ Back to ToC ⇑*](#toc)

Permissions of the directory, in symbolic or octal notation.
Defaults to `u=rwx,g=rx,o=` (`0750`).

- **Type**: `str`
- **Required**: No

#### `quadlet_podman_directories['state']`<a id="variable-quadlet_podman_directories-sub-state"></a>

[*⇑ Back to ToC ⇑*](#toc)

`present` ensures the directory exists, `absent` removes it
including its content. Handle with care.

- **Type**: `str`
- **Required**: No
- **Default**: `"present"`
- **Choices**: `present`, `absent`



### `quadlet_podman_files`<a id="variable-quadlet_podman_files"></a>

[*⇑ Back to ToC ⇑*](#toc)

List of files to manage for the application, e.g. environment files
referenced via `EnvironmentFile` or configuration files bind-mounted
into containers. Declared files are owned by the application:
content and attributes are enforced, a pre-existing file is adopted,
and de-listing (or `quadlet_podman_state: "absent"`) removes it. A
managed file belongs to exactly one application per scope;
conflicting claims by another application's manifest are rejected. Parent directories must exist (see
`quadlet_podman_directories`).

Do not put secrets into environment or configuration files if it can
be avoided; prefer `quadlet_podman_secrets`.

Changed files trigger a restart of the application's services (a
container only picks up a changed environment file when it is
restarted).

The role records the paths of the files it created in a
per-application manifest (root-owned, below `/var/lib/ansible-podman-quadlet/`): files renamed or
removed from this list get removed again on the next run, and
`quadlet_podman_state: "absent"` removes all of the application's
managed files even when the current list is empty or different.

Example:

```yaml
quadlet_podman_files:
  - dest: "/var/lib/podman/zammad/config/zammad.env"
    content: |
      POSTGRES_DB=zammad_production
      POSTGRES_HOST=127.0.0.1
```

- **Type**: `list`
- **Required**: No
- **Default**: `[]`
- **List Elements**: `dict`

#### `quadlet_podman_files['dest']`<a id="variable-quadlet_podman_files-sub-dest"></a>

[*⇑ Back to ToC ⇑*](#toc)

Absolute path of the file.

- **Type**: `str`
- **Required**: Yes

#### `quadlet_podman_files['content']`<a id="variable-quadlet_podman_files-sub-content"></a>

[*⇑ Back to ToC ⇑*](#toc)

Content of the file. Required when `state` is `present`.

- **Type**: `str`
- **Required**: No

#### `quadlet_podman_files['owner']`<a id="variable-quadlet_podman_files-sub-owner"></a>

[*⇑ Back to ToC ⇑*](#toc)

Owner of the file. Defaults to `root` for rootful applications
and to `quadlet_podman_user` for rootless ones.

- **Type**: `str`
- **Required**: No

#### `quadlet_podman_files['group']`<a id="variable-quadlet_podman_files-sub-group"></a>

[*⇑ Back to ToC ⇑*](#toc)

Group of the file. Defaults to the primary group of the file's
owner.

- **Type**: `str`
- **Required**: No

#### `quadlet_podman_files['mode']`<a id="variable-quadlet_podman_files-sub-mode"></a>

[*⇑ Back to ToC ⇑*](#toc)

Permissions of the file, in symbolic or octal notation. Defaults
to `u=rw,g=r,o=` (`0640`).

- **Type**: `str`
- **Required**: No

#### `quadlet_podman_files['state']`<a id="variable-quadlet_podman_files-sub-state"></a>

[*⇑ Back to ToC ⇑*](#toc)

`present` ensures the file exists with the given content,
`absent` removes it.

- **Type**: `str`
- **Required**: No
- **Default**: `"present"`
- **Choices**: `present`, `absent`



### `quadlet_podman_min_podman_version`<a id="variable-quadlet_podman_min_podman_version"></a>

[*⇑ Back to ToC ⇑*](#toc)

Minimum Podman version this application requires (e.g. `5.7.0` when
using `artifact` units or Quadlet settings introduced after the
role's own baseline). If the Podman version on the target host is
lower, the role fails early with a clear message instead of
deploying units the host cannot start. If unset, only the role's own
baseline check applies.

- **Type**: `str`
- **Required**: No




<!-- ANSIBLE DOCSMITH MAIN END -->

## Dependencies<a id="dependencies"></a>

See `dependencies` in [`meta/main.yml`](./meta/main.yml).



## Compatibility<a id="compatibility"></a>

See `min_ansible_version` in [`meta/main.yml`](./meta/main.yml) and `__quadlet_podman_supported_platforms` in [`vars/main.yml`](./vars/main.yml).



## External requirements<a id="requirements"></a>

There are no special requirements not covered by Ansible itself.

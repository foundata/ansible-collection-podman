# Ansible collection: `foundata.podman`

This repository contains the `foundata.podman` Ansible Collection.



## Table of contents<a id="toc"></a>

- [Included content](#content)
- [Dependencies](#dependencies)
- [Licensing, copyright](#licensing-copyright)
- [Author information](#author-information)



## Included content<a id="content"></a>

### Role: `foundata.podman.host`

The host role installs and configures Podman and its host-wide settings, including storage, registries, automatic updates, Docker compatibility, and rootless user enablement. [Its `README.md`](./roles/host/README.md) covers configuration, usage examples, and more:

<!-- ANSIBLE DOCSMITH TOC-FULL host START -->
- [Ansible role: `foundata.podman.host`](roles/host/README.md#ansible-role-foundatapodmanhost)
  - [Table of contents](roles/host/README.md#toc)
  - [Example playbooks, using this role](roles/host/README.md#examples)
  - [Centralizing rootless storage](roles/host/README.md#rootless-storage)
  - [Supported tags](roles/host/README.md#tags)
  - [Role variables](roles/host/README.md#variables)
    - [`host_podman_state`](roles/host/README.md#variable-host_podman_state)
    - [`host_podman_autoupgrade`](roles/host/README.md#variable-host_podman_autoupgrade)
    - [`host_podman_service_state`](roles/host/README.md#variable-host_podman_service_state)
    - [`host_podman_containers`](roles/host/README.md#variable-host_podman_containers)
    - [`host_podman_storage`](roles/host/README.md#variable-host_podman_storage)
    - [`host_podman_registries`](roles/host/README.md#variable-host_podman_registries)
    - [`host_podman_policy`](roles/host/README.md#variable-host_podman_policy)
    - [`host_podman_auto_update`](roles/host/README.md#variable-host_podman_auto_update)
    - [`host_podman_auto_update_timer_settings`](roles/host/README.md#variable-host_podman_auto_update_timer_settings)
    - [`host_podman_docker_compat`](roles/host/README.md#variable-host_podman_docker_compat)
    - [`host_podman_rootless_users`](roles/host/README.md#variable-host_podman_rootless_users)
      - [`host_podman_rootless_users['name']`](roles/host/README.md#variable-host_podman_rootless_users-sub-name)
      - [`host_podman_rootless_users['state']`](roles/host/README.md#variable-host_podman_rootless_users-sub-state)
      - [`host_podman_rootless_users['linger']`](roles/host/README.md#variable-host_podman_rootless_users-sub-linger)
      - [`host_podman_rootless_users['subuid']`](roles/host/README.md#variable-host_podman_rootless_users-sub-subuid)
      - [`host_podman_rootless_users['subgid']`](roles/host/README.md#variable-host_podman_rootless_users-sub-subgid)
      - [`host_podman_rootless_users['auto_update']`](roles/host/README.md#variable-host_podman_rootless_users-sub-auto_update)
      - [`host_podman_rootless_users['containers']`](roles/host/README.md#variable-host_podman_rootless_users-sub-containers)
      - [`host_podman_rootless_users['storage']`](roles/host/README.md#variable-host_podman_rootless_users-sub-storage)
      - [`host_podman_rootless_users['registries']`](roles/host/README.md#variable-host_podman_rootless_users-sub-registries)
    - [`host_podman_selinux_manage`](roles/host/README.md#variable-host_podman_selinux_manage)
    - [`host_podman_selinux_label_paths`](roles/host/README.md#variable-host_podman_selinux_label_paths)
  - [Dependencies](roles/host/README.md#dependencies)
  - [Compatibility](roles/host/README.md#compatibility)
  - [External requirements](roles/host/README.md#requirements)
<!-- ANSIBLE DOCSMITH TOC-FULL host END -->



### Role: `foundata.podman.quadlet`

The Quadlet role deploys and manages rootful or rootless OCI applications as declarative Podman Quadlet units, together with secrets, registry authentication, directories, files, and generated systemd services. [Its `README.md`](./roles/quadlet/README.md) covers configuration, usage examples, and more:

<!-- ANSIBLE DOCSMITH TOC-FULL quadlet START -->
- [Ansible role: `foundata.podman.quadlet`](roles/quadlet/README.md#ansible-role-foundatapodmanquadlet)
  - [Table of contents](roles/quadlet/README.md#toc)
  - [Example playbooks, using this role](roles/quadlet/README.md#examples)
  - [Rootless applications: data, UID mapping and SELinux](roles/quadlet/README.md#rootless-notes)
  - [Supported tags](roles/quadlet/README.md#tags)
  - [Role variables](roles/quadlet/README.md#variables)
    - [`quadlet_podman_app`](roles/quadlet/README.md#variable-quadlet_podman_app)
    - [`quadlet_podman_state`](roles/quadlet/README.md#variable-quadlet_podman_state)
    - [`quadlet_podman_service_state`](roles/quadlet/README.md#variable-quadlet_podman_service_state)
    - [`quadlet_podman_user`](roles/quadlet/README.md#variable-quadlet_podman_user)
    - [`quadlet_podman_units`](roles/quadlet/README.md#variable-quadlet_podman_units)
      - [`quadlet_podman_units['name']`](roles/quadlet/README.md#variable-quadlet_podman_units-sub-name)
      - [`quadlet_podman_units['type']`](roles/quadlet/README.md#variable-quadlet_podman_units-sub-type)
      - [`quadlet_podman_units['sections']`](roles/quadlet/README.md#variable-quadlet_podman_units-sub-sections)
    - [`quadlet_podman_unit_defaults`](roles/quadlet/README.md#variable-quadlet_podman_unit_defaults)
      - [`quadlet_podman_unit_defaults['container']`](roles/quadlet/README.md#variable-quadlet_podman_unit_defaults-sub-container)
      - [`quadlet_podman_unit_defaults['pod']`](roles/quadlet/README.md#variable-quadlet_podman_unit_defaults-sub-pod)
      - [`quadlet_podman_unit_defaults['volume']`](roles/quadlet/README.md#variable-quadlet_podman_unit_defaults-sub-volume)
      - [`quadlet_podman_unit_defaults['network']`](roles/quadlet/README.md#variable-quadlet_podman_unit_defaults-sub-network)
      - [`quadlet_podman_unit_defaults['kube']`](roles/quadlet/README.md#variable-quadlet_podman_unit_defaults-sub-kube)
      - [`quadlet_podman_unit_defaults['image']`](roles/quadlet/README.md#variable-quadlet_podman_unit_defaults-sub-image)
      - [`quadlet_podman_unit_defaults['build']`](roles/quadlet/README.md#variable-quadlet_podman_unit_defaults-sub-build)
      - [`quadlet_podman_unit_defaults['artifact']`](roles/quadlet/README.md#variable-quadlet_podman_unit_defaults-sub-artifact)
    - [`quadlet_podman_secrets`](roles/quadlet/README.md#variable-quadlet_podman_secrets)
      - [`quadlet_podman_secrets['name']`](roles/quadlet/README.md#variable-quadlet_podman_secrets-sub-name)
      - [`quadlet_podman_secrets['value']`](roles/quadlet/README.md#variable-quadlet_podman_secrets-sub-value)
      - [`quadlet_podman_secrets['state']`](roles/quadlet/README.md#variable-quadlet_podman_secrets-sub-state)
      - [`quadlet_podman_secrets['force']`](roles/quadlet/README.md#variable-quadlet_podman_secrets-sub-force)
    - [`quadlet_podman_registry_auth`](roles/quadlet/README.md#variable-quadlet_podman_registry_auth)
      - [`quadlet_podman_registry_auth['registry']`](roles/quadlet/README.md#variable-quadlet_podman_registry_auth-sub-registry)
      - [`quadlet_podman_registry_auth['username']`](roles/quadlet/README.md#variable-quadlet_podman_registry_auth-sub-username)
      - [`quadlet_podman_registry_auth['password']`](roles/quadlet/README.md#variable-quadlet_podman_registry_auth-sub-password)
    - [`quadlet_podman_directories`](roles/quadlet/README.md#variable-quadlet_podman_directories)
      - [`quadlet_podman_directories['path']`](roles/quadlet/README.md#variable-quadlet_podman_directories-sub-path)
      - [`quadlet_podman_directories['owner']`](roles/quadlet/README.md#variable-quadlet_podman_directories-sub-owner)
      - [`quadlet_podman_directories['group']`](roles/quadlet/README.md#variable-quadlet_podman_directories-sub-group)
      - [`quadlet_podman_directories['mode']`](roles/quadlet/README.md#variable-quadlet_podman_directories-sub-mode)
      - [`quadlet_podman_directories['state']`](roles/quadlet/README.md#variable-quadlet_podman_directories-sub-state)
    - [`quadlet_podman_files`](roles/quadlet/README.md#variable-quadlet_podman_files)
      - [`quadlet_podman_files['dest']`](roles/quadlet/README.md#variable-quadlet_podman_files-sub-dest)
      - [`quadlet_podman_files['content']`](roles/quadlet/README.md#variable-quadlet_podman_files-sub-content)
      - [`quadlet_podman_files['owner']`](roles/quadlet/README.md#variable-quadlet_podman_files-sub-owner)
      - [`quadlet_podman_files['group']`](roles/quadlet/README.md#variable-quadlet_podman_files-sub-group)
      - [`quadlet_podman_files['mode']`](roles/quadlet/README.md#variable-quadlet_podman_files-sub-mode)
      - [`quadlet_podman_files['state']`](roles/quadlet/README.md#variable-quadlet_podman_files-sub-state)
    - [`quadlet_podman_min_podman_version`](roles/quadlet/README.md#variable-quadlet_podman_min_podman_version)
  - [Dependencies](roles/quadlet/README.md#dependencies)
  - [Compatibility](roles/quadlet/README.md#compatibility)
  - [External requirements](roles/quadlet/README.md#requirements)
<!-- ANSIBLE DOCSMITH TOC-FULL quadlet END -->



## Dependencies<a id="dependencies"></a>

See `dependencies` in [`galaxy.yml`](./galaxy.yml).



## Licensing, copyright<a id="licensing-copyright"></a>

<!--REUSE-IgnoreStart-->
Copyright (c) 2026 foundata GmbH

This project is licensed under the GNU General Public License v3.0 or later (SPDX-License-Identifier: `GPL-3.0-or-later`), see [`LICENSES/GPL-3.0-or-later.txt`](LICENSES/GPL-3.0-or-later.txt) for the full text.

The [`REUSE.toml`](REUSE.toml) file provides detailed licensing and copyright information in a human- and machine-readable format. This includes parts that may be subject to different licensing or usage terms, such as third-party components. The repository conforms to the [REUSE specification](https://reuse.software/spec/). You can use [`reuse spdx`](https://reuse.readthedocs.io/en/latest/readme.html#cli) to create a [SPDX software bill of materials (SBOM)](https://en.wikipedia.org/wiki/Software_Package_Data_Exchange).
<!--REUSE-IgnoreEnd-->



## Author information<a id="author-information"></a>

This project was created and is maintained by foundata GmbH.

Initially based on an [Ansible skeleton](https://foundata.com/en/projects/ansible-skeletons/) developed by [foundata](https://foundata.com/).

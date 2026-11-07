================================================
foundata.podman Ansible collection Release Notes
================================================

.. contents:: Topics

v1.2.0
======

Minor Changes
-------------

- Add role-prefixed handler topics and use them for internal notifications. Retain the previous topics as migration aliases until version 2.0.0.
- Add the handler topic host_podman_reload_systemd_daemon as the replacement for "host_podman: systemd daemon-reload".
- Add the handler topic quadlet_podman_reload_systemd_daemon as the replacement for "quadlet_podman: systemd daemon-reload".
- Add the handler topic quadlet_podman_restart_application_services as the replacement for "quadlet_podman: restart application services".
- Add the handler topic quadlet_podman_restart_application_systemd_units as the replacement for "quadlet_podman: restart application systemd units".
- quadlet - add per-unit ``restart_on_change`` (default ``true``) to exclude selected services from change-triggered application restarts while still starting them for service states ``enabled`` and ``running``.
- quadlet - expose changed Quadlet filenames and generated service names in the per-invocation ``quadlet_podman_result`` fact, including unmanaged services and check mode.

Bugfixes
--------

- Check the minimum ansible-core version before gathering role facts so unsupported versions fail with the intended diagnostic.
- Limit automatic fact gathering to each role's declared subsets while retaining missing-fact recovery and reuse of cached facts.
- The declared ansible-core minimum is corrected from 2.16 to 2.19. The roles write Jinja's own filters and tests as ``ansible.builtin.<name>``, which ansible-core resolves from 2.19 only; on 2.16 to 2.18 every run failed at the first such expression, so no working installation is affected.

v1.1.0
======

Release Summary
---------------

Release Date: 2026-08-28

Maintenance release.

Minor Changes
-------------

- ``host``, ``quadlet`` - Improve Ansible compatibility for roles and collections used in the wild by giving every loop in the roles a private, purpose-specific loop variable derived from the role's public variable prefix (for example ``__quadlet_podman_*_item``) instead of ``item``. This avoids rebinding a caller's ``item`` when role parameters reference it, such as when a role itself is included from a loop, and makes the roles behave more robustly across common Ansible usage patterns.

v1.0.0
======

Release Summary
---------------

Release Date: 2026-08-13

First public release, providing all functionality and files.

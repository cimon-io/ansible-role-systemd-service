Create system service role
=========

An Ansible Role that creates systemd service file.

Requirements
------------

This role requires root access, so either run it in a playbook with a global `become: yes`, or invoke the role in your playbook like:

```yaml
- hosts: apps
  roles:
    - role: systemd-service
      become: yes
```

Role Variables
--------------

```yaml
---
# Whether the service should start on boot.
systemd_service_enabled: False

systemd_service_name: ""

# A free-form string describing the unit.
systemd_service_description: "Service for {{ systemd_service_name }}"
# https://www.freedesktop.org/software/systemd/man/systemd.unit.html#Before=
systemd_service_after: "network.target syslog.target"
systemd_service_before: ""

# https://www.freedesktop.org/software/systemd/man/systemd.exec.html#User=
systemd_service_user: ""
systemd_service_group: ""

# Configures the process start-up type for this service unit. One of simple, forking, oneshot, dbus, notify or idle.
systemd_service_type: ""
# Takes an absolute file name pointing to the PID file of this daemon.
systemd_service_pidfile: ""

# https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Nice=
systemd_service_oomscoreadjust: ""
systemd_service_nice: ""

# https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecStart=
systemd_service_execstart: ""
systemd_service_execstartpre: ""
systemd_service_execstartpost: ""
# https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecReload=
systemd_service_execstop: ""
systemd_service_execstoppost: ""
systemd_service_execreload: ""

# https://www.freedesktop.org/software/systemd/man/systemd.service.html#Restart=
systemd_service_restart: ""
# Configures the time to sleep before restarting a service (as configured with Restart=).
systemd_service_restartsec: ""
systemd_service_timeoutsec: 30

# https://www.freedesktop.org/software/systemd/man/systemd.unit.html#Requires=
systemd_service_wants: ""
systemd_service_requires: ""

# https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Environment=
systemd_service_environment: ""
systemd_service_environmentfile: ""
# https://www.freedesktop.org/software/systemd/man/systemd.exec.html#StandardInput=
systemd_service_workingdirectory: ""

# https://www.freedesktop.org/software/systemd/man/systemd.exec.html#StandardInput=
systemd_service_standardinput: "null"
systemd_service_standardoutput: "syslog"
systemd_service_standarderror: "inherit"

# https://www.freedesktop.org/software/systemd/man/systemd.unit.html#WantedBy=
systemd_service_wantedby: "multi-user.target"
systemd_service_requiredby: ""
```

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: app
  roles:
    - role: systemd_service
      systemd_service_enabled: yes
      systemd_service_name: "railsapp"
      systemd_service_user: "deploy"
      systemd_service_group: "deploy"
      systemd_service_execstart: "/bin/bash -lc 'puma -C config/puma.rb'"
      systemd_service_restart: "on-failure"
      systemd_service_wants: "redis.service"
      systemd_service_requires: "postgresql.service"
      systemd_service_workingdirectory: "/var/www/myapp"
      systemd_service_wantedby: "multi-user.target"
```

License
-------

MIT

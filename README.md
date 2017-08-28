Create system service role
=========

An Ansible Role that creates upstart/systemd unit files.

Requirements
------------

None.

Role Variables
--------------



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

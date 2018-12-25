# Ansible systemd service role

An ansible role that creates and configures `systemd service unit` files. It allows you to run certain services automatically in the background, enable and disable them for specific events, manage groups of processes and configure dependencies on other units.

The role includes the following tasks:
1. Create a  systemd service file at `/etc/systemd/system/` with a specified name `service_name`.service for each `systemd_service` item.
2. Configure important Unit, Service and Install section options.
3. Notify systemd that new service files exist. Restart the service.
4. Enable the service autorun on boot if necessary.

This role can be run under all versions of Ubuntu.

## Requirements

This role requires the root access, so either run it in a playbook with a global `become: yes` parameter, or invoke the role in your playbook as follows:

```yaml
- hosts: apps
  roles:
    - role: systemd-service
      become: yes
```

## Role Variables

All necessary services can be specified by the `systemd_service` dictionary variable (see `defaults/main.yml`):

```yaml
systemd_service: {}
```

For each service you need to set `service_name` and necessary parameter values. For example, to specify whether the service should start on boot use the parameter `enabled`.

```yaml
systemd_service:
  service_name:
    enabled:
```

All other available parameters which should be specified for a specific `service_name` key (as nested parameters) are given below.

### Unit section options

This group of parameters includes generic information about the unit.

```yaml
description:   # A free-form string describing the unit
```

Next two parameters configure dependencies on other units. If the service gets activated, the units listed there will be activated as well. If one of the `requires` units can not be run or suddenly fails, the service will be stopped too. As for the `wants` list, the service won't be stopped if some unit from it gets deactivated. The parameters can consist of several space-separated unit names. They may also be specified more than once.

```yaml
requires:   # Units which must be started along with the service
wants:      # Units which should be started along with the service
```

To configure the order in which services are started or stopped use the following parameters. Note, that if units don't have ordering dependencies between them, they are shut down or started up simultaneously. Both parameters are set by space-separated lists of units

```yaml
after:   # Units which must be started after the service
before:  # Units which must be started before the service
```

### Service section options

This section includes information about the service and the process it supervises. The parameter `type` configures the process start-up type for this service unit.

```yaml
type:
```

It can take values:
- `simple` - this type assumes that the service will be launched immediately. The process must not branch out. Do not use this type if other services have ordering dependencies on the service launching. If the service offers functionality to other processes on the system, its communication channels should be installed before the daemon is started up.
- `forking` - assumes that the service is started once and the process branches out with the completion of the parent process. This type is used to launch classic daemons. If this mode is used, it is recommended to use the `pid_file` parameter too (see below), so that systemd can identify the main process of the daemon.

Other values behavior is similar to simple value. However, they have some differences:
- `oneshot` - the service is expected to exit before systemd starts follow-up units;
- `dbus` - it is expected that the daemon acquires a name on the `D-Bus` bus;
- `notify` - the daemon sends a notification message via `sd_notify(3)` or a similar call when it has finished starting up;
- `idle` - actual execution of the service binary is delayed until all active jobs are dispatched. Note that this type is useful only to improve console output, it is not useful as a general unit ordering tool.

Set a path to the PID file to use `forking` start-up type.

```yaml
pid_file:    # Takes an absolute file name pointing to the PID file of this daemon
```

You can specify the UNIX user and a group under which the service should be executed. The parameters take a single user or group name, or a numeric ID as a value. For system services and for user services of the root user the default is `root` that may be switched to another one. For user services of any other user switching user identity is not permitted. So the only allowed value is the same user the user's service manager is running as. If no group is set, the default user group is used.

```yaml
user:
group:
```

Set a scheduling priority for the unit with the next parameter. It takes an integer between -20 (the highest priority) and 19 (the lowest priority).

```yaml
nice:    # The default nice level for the service
```

An adjustment level for the Out-Of-Memory killer for the process is specified with the following option. It takes an integer value from -1000 (disable OOM killing) to 1000 (OOM killing is preferable).

```yaml
oom_score_adjust:
```

Next parameters allows you to specify commands that will be executed depending on the state of your service. The parameters may be used more than once or their values may include several commands. Multiple command lines may be concatenated in a single directive by separating them with semicolons. The command to execute must be an absolute path name. It may contain spaces, but control characters are not allowed. For each command the first argument must be an absolute path to an executable. An empty string will reset the list of commands specified before for the parameter.

```yaml
# Commands that are executed when this service is started
# Unless `type` is `oneshot`, exactly one command must be given here
exec_start:

# Commands that are executed before `exec_start` commands
exec_start_pre:

# Commands that are executed after `exec_start` commands
exec_start_post:

# Commands to execute to stop the service started via `exec_start`
exec_stop:

# Commands that are executed after the service is stopped
exec_stop_post:

# Commands to execute to trigger a configuration reload in the service
exec_reload:
```

Set whether the service should be restarted when the service process (the main service process or one specified by 'exec_start_pre', 'exec_start_post', 'exec_stop', 'exec_stop_post' or 'exec_reload' parameters) exits, is killed, or a timeout is reached. The `restart` parameter takes one of the following values:
- `no` (by default) - the service will not be restarted;
- `on-success` - the service will be restarted only when the service process exits cleanly (with an exit code of 0, or one of the signals SIGHUP, SIGINT, SIGTERM or SIGPIPE);
- `on-failure` - the service will be restarted when the process exits with a non-zero exit code, is terminated by a signal, when an operation times out, and when the configured watchdog timeout is triggered;
- `on-abnormal` - the service will be restarted when the process is terminated by a signal, when an operation times out, or when the watchdog timeout is triggered;
- `on-watchdog` - the service will be restarted only if the service process exits due to an uncaught signal not specified as a clean exit status;
- `on-abort` - the service will be restarted only if the watchdog timeout for the service expires;
- `always` - the service will be restarted anyway.

```yaml
# When the service must be restarted
restart:
```

You are able to specify a time delay for above-mentioned commands with next parameters. They take a value in seconds or a time span value such as '5min 20s'. The `restart_sec` parameter configures the time to sleep before restarting a service (as configured with `restart`). The `timeout_sec` option defines time to wait for start/stop commands processing.

```yaml
restart_sec:
timeout_sec:
```

Use the `environment` parameter to set a list of environment variables for executed processes. It includes a space-separated list of variables and their values. The parameter can be used more than once. If the empty string is assigned to this option, the list of environment variables will be reset. If some value contains a space, use double quotes for the assignment.

You are also able to read the environment variables from a text file. For this set the `environment_file` parameter value as the file path.

```yaml
environment:       # A space-separated list of variable assignments
environment_file:  # Path to a file with environment variables
```

A working directory is specified by the next parameter. It is set as current before startup commands are launched.

```yaml
working_directory:
```

The following parameters allow to choose where file descriptors (STDIN, STDOUT, STDERR) of the executed processes should be connected to. The `standard_input` parameter takes values "null", "tty", "tty-force", "tty-fail", "socket" or "fd". The `standard_output` parameter can be equal to "inherit", "null", "tty", "journal", "syslog", "kmsg", "journal+console", "syslog+console", "kmsg+console", "socket" or "fd". The available values of `standard_error` are identical to `standard_output`.

```yaml
standard_input:
standard_output:
standard_error:
```

### Install section options

This section variables carry installation information for the unit. The following two parameters can be used more than once, or space-separated lists of unit names may be specified. The lists include units which refers to this service from their `requires` and `wants` fields.

```yaml
wanted_by:
required_by:
```

## Dependencies

None

## Example playbook

```yaml
- hosts: app
  roles:
    - role: systemd-service
      systemd_service:
        # Service name
        railsapp:
          # Start the service on boot
          enabled: Yes
          # Execute the command with specified arguments when the service is started
          exec_start: "/bin/bash -lc 'puma -C config/puma.rb'"
          # Use the specified directory as current
          working_directory: "/var/www/myapp"
          # Run the processes under this user and group
          user: "deploy"
          group: "deploy"
          # Restart the service only when a clean exit code or signal wasn't gotten
          restart: "on-failure"
          # Try to activate 'redis' if possible
          wants: "redis.service"
          # Activate 'postgresql' or stop working in case of failure
          requires: "postgresql.service"
          # multi-user.target unit prefers the service to be run
          wanted_by: "multi-user.target"
```

## License

Licensed under the [MIT License](https://opensource.org/licenses/MIT).

# Ansible system service role

An ansible role that creates and configures a `systemd service unit` file. It allows you to run certain services automatically in the background, enable and disable them for specific events, manage groups of processes and configure dependencies on other units.

The role includes the following tasks:
1. Create a  systemd service file at `/etc/systemd/system/` with a specified name `systemd_service_name`.service.
2. Configure important Unit, Service and Install section options.
3. Notify systemd that a new systemd_service_name.service file exists. Restart the service.
3. Enable the service autorun on boot if necessary.

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

To specify whether the service should start on boot use the parameter `systemd_service_enabled`. By default, it's false:

```yaml
systemd_service_enabled: False
```

### Unit section options

This group of variables includes generic information about the unit.

```yaml
# A service unit file name
systemd_service_name: ""

# A free-form string describing the unit
systemd_service_description: "Service for {{ systemd_service_name }}"
```

Next two parameters configure dependencies on other units. If the service gets activated, the units listed there will be activated as well. If one of the `systemd_service_requires` units can not be run or suddenly fails, the service will be stopped too. As for the `systemd_service_wants` list, the service won't be stopped if some unit from it gets deactivated.

```yaml
# Units which must be started along with the service
systemd_service_requires: ""

# Units which should be started along with the service
systemd_service_wants: ""
```

The parameters can consist of several space-separated unit names. They may also be specified more than once. To configure the order in which services are started or stopped use the following parameters. Note, that if units don't have ordering dependencies between them, they are shut down or started up simultaneously.

```yaml
# A space-separated list of units which must be started after (and before) the service
systemd_service_after: "network.target syslog.target"
systemd_service_before: ""
```

### Service section options

This section includes information about the service and the process it supervises. The parameter `systemd_service_type` configures the process start-up type for this service unit.

```yaml
systemd_service_type: ""
```

It can take values:
- `simple` - this type assumes that the service will be launched immediately. The process must not branch out. Do not use this type if other services have ordering dependencies on the service launching. If the service offers functionality to other processes on the system, its communication channels should be installed before the daemon is started up.
- `forking` - assumes that the service is started once and the process branches out with the completion of the parent process. This type is used to launch classic daemons. If this mode is used, it is recommended to use the `systemd_service_pidfile` parameter too (see below), so that systemd can identify the main process of the daemon.

Other values behavior is similar to simple value. However, they have some differences:
- `oneshot` - the service is expected to exit before systemd starts follow-up units;
- `dbus` - it is expected that the daemon acquires a name on the `D-Bus` bus;
- `notify` - the daemon sends a notification message via `sd_notify(3)` or a similar call when it has finished starting up;
- `idle` - actual execution of the service binary is delayed until all active jobs are dispatched. Note that this type is useful only to improve console output, it is not useful as a general unit ordering tool.

Set a path to the PID file to use `forking` start-up type.

```yaml
# Takes an absolute file name pointing to the PID file of this daemon
systemd_service_pidfile: ""
```

You can specify the UNIX user and a group under which the service should be executed. The parameters take a single user or group name, or a numeric ID as a value. For system services and for user services of the root user the default is `root` that may be switched to another one. For user services of any other user switching user identity is not permitted. So the only allowed value is the same user the user's service manager is running as. If no group is set, the default user group is used.

```yaml
systemd_service_user: ""
systemd_service_group: ""
```

Set a scheduling priority for the unit with a parameter:

```yaml
# The default nice level for the service
systemd_service_nice: ""
```

An adjustment level for the Out-Of-Memory killer for the process is specified with the following option. It takes an integer value from -1000 (disable OOM killing) to 1000 (OOM killing is preferable).

```yaml
systemd_service_oomscoreadjust: ""
```

Next parameters allows you to specify commands that will be executed depending on the state of your service. The parameters may be used more than once or their values may include several commands. Multiple command lines may be concatenated in a single directive by separating them with semicolons. The command to execute must be an absolute path name. It may contain spaces, but control characters are not allowed. For each command the first argument must be an absolute path to an executable. An empty string will reset the list of commands specified before for the parameter.

```yaml
# Commands that are executed when this service is started
# Unless `systemd_service_type` is `oneshot`, exactly one command must be given here
systemd_service_execstart: ""

# Commands that are executed before `systemd_service_execstart` commands
systemd_service_execstartpre: ""

# Commands that are executed after `systemd_service_execstart` commands
systemd_service_execstartpost: ""

# Commands to execute to stop the service started via `systemd_service_execstart`
systemd_service_execstop: ""

# Commands that are executed after the service is stopped
systemd_service_execstoppost: ""

# Commands to execute to trigger a configuration reload in the service
systemd_service_execreload: ""
```

Set whether the service should be restarted when the service process (the main service process or one specified by 'execstartpre', 'execstartpost', 'execstop', 'execstoppost' or 'execreload' parameters) exits, is killed, or a timeout is reached. The `systemd_service_restart` parameter takes one of the following values:
- `no` (by default) - the service will not be restarted;
- `on-success` - the service will be restarted only when the service process exits cleanly (with an exit code of 0, or one of the signals SIGHUP, SIGINT, SIGTERM or SIGPIPE);
- `on-failure` - the service will be restarted when the process exits with a non-zero exit code, is terminated by a signal, when an operation times out, and when the configured watchdog timeout is triggered;
- `on-abnormal` - the service will be restarted when the process is terminated by a signal, when an operation times out, or when the watchdog timeout is triggered;
- `on-watchdog` - the service will be restarted only if the service process exits due to an uncaught signal not specified as a clean exit status;
- `on-abort` - the service will be restarted only if the watchdog timeout for the service expires;
- `always` - the service will be restarted anyway.

```yaml
# When the service must be restarted
systemd_service_restart: ""
```

You are able to specify a time delay for above-mentioned commands with next parameters. They take a value in seconds or a time span value such as '5min 20s'.

```yaml
# Configures the time to sleep before restarting a service (as configured with `systemd_service_restart`).
systemd_service_restartsec: ""
# Configures the time to wait for start/stop commands processing
systemd_service_timeoutsec: 30
```

Use the `systemd_service_environment` parameter to set environment variables for executed processes. It includes a space-separated list of variables and their values. The parameter can be used more than once. If the empty string is assigned to this option, the list of environment variables will be reset. If some value contains a space, use double quotes for the assignment.

You are also able to read the environment variables from a text file. For this set the `systemd_service_environmentfile` parameter value as the file path.

```yaml
# Environment variables for executed processes
systemd_service_environment: ""
systemd_service_environmentfile: ""
```

A working directory is specified by the next parameter.

```yaml
# A working directory which is set as current before startup commands are launched
systemd_service_workingdirectory: ""
```

The following parameters allow to choose where file descriptors (STDIN, STDOUT, STDERR) of the executed processes should be connected to.

```yaml
# Controls where file descriptor 0 (STDIN) of the executed processes is connected to
# Takes values 'null', 'tty', 'tty-force', 'tty-fail', 'socket' or 'fd'.
systemd_service_standardinput: "null"

# Controls where file descriptor 1 (STDOUT) of the executed processes is connected to
# Takes values 'inherit', 'null', 'tty', 'journal', 'syslog', 'kmsg', 'journal+console',
# 'syslog+console', 'kmsg+console', 'socket' or 'fd'.
systemd_service_standardoutput: "syslog"

# Controls where file descriptor 2 (STDERR) of the executed processes is connected to
# The available values are identical to 'systemd_service_standardoutput'
systemd_service_standarderror: "inherit"
```
### Install section options

This section variables carry installation information for the unit. The following two parameters can be used more than once, or space-separated lists of unit names may be specified. The lists include units which refers to this service from their `systemd_service_requires` and `systemd_service_wants` fields.

```yaml
systemd_service_wantedby: "multi-user.target"
systemd_service_requiredby: ""
```

## Dependencies

None

## Example playbook

```yaml
- hosts: app
  roles:
    - role: systemd_service
      # Start the service on boot
      systemd_service_enabled: yes
      # The service name
      systemd_service_name: "railsapp"
      # Run the processes under this user
      systemd_service_user: "deploy"
      systemd_service_group: "deploy"
      # Execute the command with specified arguments when the service is started
      systemd_service_execstart: "/bin/bash -lc 'puma -C config/puma.rb'"
      # Restart the service only when a clean exit code or signal wasn't gotten
      systemd_service_restart: "on-failure"
      # Try to activate 'redis' if possible
      systemd_service_wants: "redis.service"
      # Activate 'postgresql' or stop working in case of failure
      systemd_service_requires: "postgresql.service"
      # Use the specified directory as current
      systemd_service_workingdirectory: "/var/www/myapp"
      # multi-user.target unit prefers the service to be run
      systemd_service_wantedby: "multi-user.target"
```

## License

Licensed under the [MIT License](https://opensource.org/licenses/MIT).

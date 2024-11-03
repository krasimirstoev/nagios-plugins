# Nagios Multi-Process Checker Script

This script, ```check_multi_procs.sh```, is designed to monitor multiple processes on a system using Nagios. It reads configuration files that specify the processes to check, along with their warning and critical thresholds, and aggregates the results into a single service check within Nagios.

## Features

- **Customizable Monitoring**: Define any number of processes to monitor by creating simple `.check` files.
- **User-Specific Checks**: Optionally specify the user who owns the process for more precise monitoring.
- **Aggregated Results**: All process checks are combined, and the overall status is reported as a single service in Nagios.
- **Threshold Alerts**: Set custom warning and critical thresholds for each process based on the number of running instances.
- **Performance Data**: Outputs performance data compatible with Nagios graphing tools.

## How It Works

1 .**Configuration Files**: The script scans the directory ``/usr/local/nagios/check_procs/`` for ``.check`` files. Each file represents a process to monitor and contains key-value pairs:
```bash
user: root
name: nginx
warning: 1:48
critical: 1:55
```
- ``user``: (Optional) The username who owns the process.
- ``name``: The exact name of the process as it appears in the system's process list.
- ``warning``: Warning threshold for the number of processes, in the format ``min:max``.
- ``critical``: Critical threshold for the number of processes, in the format ``min:max``.

2. **Process Checks**: For each ``.check`` file, the script:
- Reads the process name, user, and thresholds.
- Executes the ``check_procs`` Nagios plugin with the specified parameters.
- Collects the output and exit code.

3. **Status Aggregation**: The script determines the highest severity level among all checks:
- **OK (0)**: All processes are within specified thresholds.
- **WARNING (1)**: At least one process exceeds the warning threshold.
- **CRITICAL (2)**: At least one process exceeds the critical threshold.
- **UNKNOWN (3)**: An error occurred during the check.

4. **Output Generation**: The script compiles a summary message and performance data for Nagios, which includes:

- The status of each process check.
- Aggregated performance data for graphing.

5. **Nagios Integration**: The script is executed as a custom command in Nagios, allowing you to view all process checks under a single service in the Nagios web interface.

## Installation and Setup

1. **Place the Script**

Save the script as ``/usr/local/nagios/libexec/check_multi_procs.sh`` and make it executable:
```bash
chmod +x /usr/local/nagios/libexec/check_multi_procs.sh
```
2. **Create the Configuration Directory**

Ensure the directory for ``.check`` files exists:
```bash
mkdir -p /usr/local/nagios/check_procs
```
3. **Create ``.check`` Files**

Create individual ``.check`` files for each process you wish to monitor.

**Example**: ``/usr/local/nagios/check_procs/nginx.check``
```bash
user: root
name: nginx
warning: 1:48
critical: 1:55
```
4. **Set Permissions**

Ensure Nagios has the necessary permissions:
```bash
chown -R nagios:nagios /usr/local/nagios/check_procs
```
5. **Define the Nagios Command**

Add the following to your Nagios ``commands.cfg`` file (usually located at ``/usr/local/nagios/etc/objects/commands.cfg``):
```bash
define command {
    command_name    check_multi_procs
    command_line    /usr/local/nagios/libexec/check_multi_procs
}
```
6. **Define the Nagios Service**

Add the service definition to your Nagios configuration file (e.g., ``/usr/local/nagios/etc/objects/localhost.cfg``):
```bash
define service {
    use                     generic-service
    host_name               localhost
    service_description     Process Checks
    check_command           check_multi_procs
}
```
7. **Verify and Restart Nagios**

Check your Nagios configuration for errors:
```bash
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```
Restart the Nagios service:
```bash
sudo systemctl restart nagios
```

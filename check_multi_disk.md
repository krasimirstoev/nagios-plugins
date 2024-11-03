# Nagios Multi-Disk Checker Script

This script, ``check_multi_disk.sh``, is designed to monitor multiple disk partitions and their inode usage on a system using Nagios. It reads configuration files that specify the disks to check, along with their warning and critical thresholds for disk space and inode usage, and aggregates the results into a single service check within Nagios.

## Features

- **Customizable Monitoring**: Define any number of disk partitions to monitor by creating a simple configuration file.
- **Aggregated Results**: All disk checks are combined, and the overall status is reported as a single service in Nagios.
- **Threshold Alerts**: Set custom warning and critical thresholds for each disk based on disk space and inode usage.
- **Performance Data**: Outputs performance data compatible with Nagios graphing tools.

## How It Works

1 .**Configuration Files**: The script reads from a configuration file located at ``/usr/local/nagios/check_disk/all.conf``. This file specifies the disks to monitor and their respective thresholds. Each disk entry contains key-value pairs:
```bash
device: /
disk_warning: 15%
disk_critical: 5%
inode_warning: 30%
inode_critical: 10%

device: /tmp
disk_warning: 20%
disk_critical: 10%
inode_warning: 20%
inode_critical: 10%
```
- **Fields**:

    - ``device``: The mount point or device path to monitor.
    - ``disk_warning``: Warning threshold for disk space usage.
    - ``disk_critical``: Critical threshold for disk space usage.
    - ``inode_warning``: Warning threshold for inode usage.
    - ``inode_critical``: Critical threshold for inode usage.

2. **Disk Checks**: For each device entry in the configuration file, the script:

- **Reads** the device path and thresholds.
- **Executes** the check_disk Nagios plugin with the specified parameters.
- **Collects** the output and exit code.

3. **Status Aggregation**: The script determines the highest severity level among all checks:

- **OK (0)**: All disks are within specified thresholds.
- **WARNING (1)**: At least one disk exceeds the warning threshold.
- **CRITICAL (2)**: At least one disk exceeds the critical threshold.
- **UNKNOWN (3)**: An error occurred during the check.

4. **Output Generation**: The script compiles a summary message and performance data for Nagios, which includes:

- The status of each process check.
- Aggregated performance data for graphing.

5. **Nagios Integration**: The script is executed as a custom command in Nagios, allowing you to view all process checks under a single service in the Nagios web interface.

## Installation and Setup

1. **Place the Script**

Save the script as ``/usr/local/nagios/libexec/check_multi_disk`` and make it executable:
```bash
chmod +x /usr/local/nagios/libexec/check_multi_disk
```
2. **Create the Configuration Directory**

```bash
mkdir -p /usr/local/nagios/check_disk
```

3. **Create the Configuration File**

Create the ``all.conf`` file with the disk partitions you wish to monitor.

**Example**: ``/usr/local/nagios/check_disk/all.conf``
```bash
device: /
disk_warning: 50%
disk_critical: 95%
inode_warning: 50%
inode_critical: 95%

device: /tmp
disk_warning: 80%
disk_critical: 90%
inode_warning: 80%
inode_critical: 90%
```
4. **Set Permissions**

Ensure Nagios has the necessary permissions:
```bash
chown -R nagios:nagios /usr/local/nagios/check_disk
```
5. **Define the Nagios Command**

Add the following to your Nagios ``commands.cfg`` file (usually located at ``/usr/local/nagios/etc/objects/commands.cfg``):
```bash
define command {
    command_name    check_multi_disk
    command_line    /usr/local/nagios/libexec/check_multi_disk
}
```
6. **Define the Nagios Service**

Add the service definition to your Nagios configuration file (e.g., ``/usr/local/nagios/etc/objects/localhost.cfg``):
```bash
define service {
    use                     generic-service
    host_name               localhost
    service_description     Disk and Inode Usage
    check_command           check_multi_disk
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

# nagios-plugins
Collection of Nagios scripts I've written over the time

[**The Nagios Multi-Process Checker Script**](check_multi_procs) is a custom shell script designed to monitor multiple processes on a system using Nagios. It reads configuration files from a specified directory, where each .check file defines a process to monitor, including optional user ownership and warning/critical thresholds for process counts. The script executes checks for each process, aggregates the results, and outputs a combined status and performance data suitable for a single Nagios service check. This allows for simplified monitoring of multiple processes with consolidated reporting in the Nagios interface. [Documentation](check_multi_procs.md).

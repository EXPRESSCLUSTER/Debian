# DRBD Troubleshooting

DRBD (Distributed Replicated Block Device) troubleshooting refers to a series of tasks to identify the cause and resolve issues when DRBD malfunctions or behaves unexpectedly. Since DRBD achieves high availability by mirroring block devices in real time between two servers, troubleshooting requires consideration of various elements such as network, storage, configuration, and applications.

***

## Common Issues and Troubleshooting Points

Below are common issues encountered during DRBD troubleshooting and points for their resolution.

### 1. Initial Configuration Issues

If DRBD behaves unexpectedly during setup, the problem is often in the configuration files.

* **Check configuration files**: Use the `drbdadm -d adjust <resource name>` command to compare the actual DRBD resource settings with the configuration files and identify the source and cause of errors. Correct any mistakes in the configuration files (`/etc/drbd.d/*` or `/etc/drbd.conf`).
* **Hostname mismatch**: If the hostname definition (`on hostname{...}`) in `/etc/drbd.conf` does not match the result of `uname -n`, communication issues may occur. Make sure these match.
* **Insufficient metadata area**: If the metadata area is insufficient when creating a DRBD resource, an error may occur. Usually, at least 128MB of metadata area is required.
* **Missing partition**: An error may also occur if the device for the DRBD resource does not have a partition.

### 2. Connectivity Issues

Since DRBD synchronizes data over the network, connectivity between nodes is crucial.

* **Check DRBD status**: Use the `drbdadm status` or `cat /proc/drbd` command to check the DRBD status and ensure it is **`Connected`**. If it remains **`StandAlone`** or **`Connecting`**, there may be a network connectivity issue.
* **Firewall**: Check that the port used by DRBD (default is TCP 7788, etc.) is not blocked by the firewall.
* **IP address/hostname resolution**: Ensure that IP addresses and hostnames can be resolved correctly between nodes. Review `/etc/hosts` or DNS settings.
* **Network cable/NIC failure**: Physical network issues are also possible.

### 3. Split-Brain Occurrence

Split-brain is a state where both nodes become primary due to network failure or node down, losing recognition of each other. This compromises data consistency and requires urgent action.

* **Identify the cause**: Determine the cause of the split-brain (network failure, node down, etc.).
* **Data consistency**: If data was updated on both nodes during split-brain, you must choose to discard changes on one node or manually merge them. Generally, discard the changes on the node with fewer or less important updates.
* **Manual recovery**: Demote one of the split-brain nodes to secondary and resynchronize with the other node. Specific commands vary by situation, but use commands like `drbdadm disconnect <resource name>`, `drbdadm -- --discard-my-data connect <resource name>`.
* **Automatic recovery policy**: DRBD has an automatic split-brain recovery policy feature, but it should be configured carefully.

### 4. Disk Failure

Since DRBD mirrors block devices, you must also respond to failures in the underlying disk.

* **DRBD `on-io-error` setting**: By setting the `on-io-error` parameter to `detach` in the `disk` section of the DRBD configuration file (`drbd.conf`), DRBD will automatically detach the disk in case of disk failure. The failed node becomes secondary, and the other node continues as primary.
* **Replacing with a new disk**: Replace the failed disk with a new one and rebind the DRBD device. If using internal metadata, create metadata on the new disk and resynchronize to recover.

### 5. Performance Issues

If DRBD I/O performance does not meet expectations, check the following:

* **Network bandwidth**: Ensure sufficient network bandwidth between nodes. Replication-heavy environments may experience bottlenecks.
* **Disk I/O**: Ensure the underlying disk has sufficient I/O performance.
* **DRBD protocol**: Check if the DRBD replication protocol (A, B, C) is appropriate.
    * **Protocol A**: Asynchronous replication. Good write performance but possible data loss if the primary fails.
    * **Protocol B**: Semi-synchronous replication. Balanced write performance and data protection.
    * **Protocol C**: Synchronous replication. Lowest write performance but minimal data loss. Recommended when high availability is the top priority.
* **Kernel module issues**: Check the version and compatibility of the DRBD kernel module.

***

## General Troubleshooting Procedure

DRBD troubleshooting can be carried out using the following general procedure:

1.  **Check status**: First, use the `drbdadm status` or `cat /proc/drbd` command to understand the current DRBD status. This will show whether DRBD is **`Connected`** and **`UpToDate`** (data is synchronized), or in an abnormal state such as **`StandAlone`**, **`Connecting`**, or **`Inconsistent`** (data is inconsistent).
    
2.  **Check logs**: Check system logs (`/var/log/messages` or `dmesg`, etc.) for DRBD-related error messages. Error messages are very helpful in identifying the cause of the problem.

3.  **Check configuration files**: Use the `drbdadm -d adjust <resource name>` command to check for syntax or logical errors in the configuration files.

4.  **Check network connectivity**: Use tools like `ping` or `netcat` (nc) to check network connectivity between DRBD nodes and the reachability of the ports used by DRBD.

5.  **Restart resources**: If the problem is temporary, stopping and restarting the DRBD resource may resolve it. However, do this carefully considering the impact on services.

6.  **Isolate the problem**: Determine whether the issue is with DRBD itself, the underlying storage or network, or higher-level cluster management software (such as Pacemaker).

7.  **Refer to documentation**: Refer to the official DRBD documentation, related blog articles, and forums to check for similar issues and suggested solutions.

By following these steps, you can efficiently troubleshoot DRBD. 
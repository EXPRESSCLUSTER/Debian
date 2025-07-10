# EXPRESSCLUSTER 5 on Debian 12

Versions:

- EXPRESSCLUSTER X 5.3 for Linux
- Debian 12.11 (kernel 6.1.0-37-amd64)
- MariaDB 10.11.11

Configuration:

- 2 nodes
- Data mirroring

Note:

Now in 2025.Jul.10, Debian 12.11 has an issue that cannot load the watchdog.
Due to it, ECX `userw` cannot utilize `softdog` untill the *forthcoming update* of Debian.

https://www.debian.org/News/2025/20250517
> Known Issues
>
> Linux 6.1.137-1, included with Debian 12.11, is unable to load the watchdog and `w83977f_wdt` modules on the amd64 architecture. This is a regression.
>
> This issue will be fixed in a forthcoming update.
>
> Users who rely on the watchdog functionality should disable their watchdog or avoid upgrading to this kernel version until a fix is available.

One of the possible options would be that omitting `userw` from the configuration and adding `userw` once the problem on Debian will be fixed.

The disadvantage of omitting `userw` is that there is no automatic OS reset when userland gets long delay. In practice, the system operation will include manual operation such like "Detect delay > **Manual stop** > Automatic failover."

Thus the installation and configuration steps remove `userw` resource from the cluster for now intentionally.

## Installation and configuration steps

1. Servers preparation

    Prepare 2 servers. Each server has

    - 2 NICs
    - 2 DISKs

2. Debian installation and configuration

    Install Debian to `/dev/sda`

    Configure disk:
    - `/dev/sdb1` for the Cluster Partition of the Mirror Disk resource which should be **1 GB**.
    - `/dev/sdb2` for the Data Partition of the Mirror Disk resource.

    Configure network:
    - `eth0` to have `10.0.0.11/24` for example of public network.
    - `eth1` to have `192.168.0.11/24` for example of private network.

3. ECX installation and configuration

    1. Install ECX

       ```bash
       $ sudo apt install ./expresscls-5.3.0-1.amd64.deb
       $ sudo clplcnsc -i *
       $ sudo reboot
       ```

    2. Open *EXPRESSCLUSTER WebUI*, make the configuration of the cluster.

    3. **Remove *userw* monitor resource from the configuration,** then apply the configuration.

4. MariaDB installation and configuration

    ```bash
    $ sudo apt update
    $ sudo apt install -y mariadb-server mariadb-client
    ```

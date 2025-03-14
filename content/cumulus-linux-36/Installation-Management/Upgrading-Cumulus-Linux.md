---
title: Upgrading Cumulus Linux
author: Cumulus Networks
weight: 45
pageID: 8362142
---
Cumulus Networks software melds the Linux host world with the networking
devices world. Each world comes with its own paradigm on how to upgrade
software. Before we discuss the various ways to upgrade Cumulus Linux
switches, let's review the general considerations and strategies used to
upgrade network devices and Linux hosts.

## Upgrades: Comparing the Network Device Worldview and the Linux Host Worldview

### Manual and Automated Configuration

Historically, *network devices* were configured in place, and most
network devices required customized configurations, which led
predominantly to configuring the hardware manually. A lack of
standardization between vendors, device types, and device roles hampered
the development of APIs and automation tools. However, in the case of
very large data centers, configurations became uniform and repeatable,
and therefore scriptable. Some larger enterprises had to develop their
own custom scripts to roll out data center network configurations.
Virtually no industry-standard provisioning tools existed.

In contrast to data center network devices, *Linux hosts* in the data
center number in the thousands and tend to have similar configurations.
This increased scale led Linux sysadmins long ago to move to common
tools to automate installation and configuration, since manually
installing and configuring hosts did not work at the scale of a data
center. Nearly all tasks are done via commonly available provisioning
and orchestration tools.

### Pre-deployment Testing of Production Environments

Historically, the cost of *network device* testing has been hampered by
the cost of a single device. Setting up an appropriately sized lab
topology can be very expensive. As a result, it is difficult to do
comprehensive topology-based testing of a release before deploying it.
Thus, many network admins cannot or will not do comprehensive system
testing of a new release before deploying it.

Alternatively, the cost of a *Linux host* is cheap (or nearly free when
using virtualization), so rigorous testing of a release before deploying
it is not encumbered by budgeting concerns. Most sysadmins extensively
test new releases in the complete application environment.

### Locations of Configuration Data Versus Executables

*Network devices* generally separate configuration data from the
executable code. On bootup, the executable code looks into a different
file system and retrieves the configuration file or files, parses the
text and uses that data to configure the software options for each
software subsystem. The model is very centralized, with the executables
generally being packaged together, and the configuration data following
a set of rules that can be read by a centralized parser. Each vendor
controls the configuration format for the entire box, since each vendor
generally supports only their own software. This made sense since the
platform was designed as an application-specific appliance.

Since a *Linux host* is a general purpose platform, with applications
running on top of it, the locations of the files are much more
distributed. Applications install and read their configuration data from
text files usually stored in the /etc directory tree. Executables are
generally stored in one of several *bin* directories, but the bin and
etc directories are often on the same physical device. Since each
*module* (application or executable) was often developed by a different
organization and shared with the world, each module was responsible for
its own configuration data format. Most applications are community
supported, and while there are some generally accepted guiding
principles on how their configuration data is formatted, no central
authority exists to control or ensure compliance.

### Upgrade Procedure

Both network admins and sysadmins generally plan upgrades only to gain
new functionality or to get bug fixes when the workarounds become too
onerous. The goal is to reduce the number of upgrades as much as
possible.

The *network device* upgrade paradigm is to leave the configuration data
in place, and *replace the executable files* either all at once from a
single binary image or in large chunks (subsystems). A full release
upgrade comes with risk due to unexpected behavior changes in subsystems
where the admin did not anticipate or need changes.

The *Linux host* upgrade paradigm is to independently *upgrade a small
list of packages* while leaving most of the OS untouched. Changing a
small list of packages reduces the risk of unintended consequences.
Usually upgrades are a "forward only" paradigm, where the sysadmins
generally plan to move to the latest code within the same major release
when needed. Every few years, when a new kernel train is released, a
major upgrade is planned. A major upgrade involves wiping and replacing
the entire OS and migrating configuration data.

### Rollback Procedure

Even the most well planned and tested upgrades can result in unforeseen
problems, and sometimes the best solution to new problems is to roll
back to the previous state.

Since *network devices* clearly separate data and executables, generally
the process is to *overwrite the new release executable* with the
previously running executable. If the configuration was changed by the
newer release, then you either have to manually back out or repair the
changes, or restore from an already backed up configuration.

The *Linux host* scenario can be more complicated. There are three main
approaches:

  - Back out individual packages: If the problematic package is
    identified, the sysadmin can downgrade the affected package
    directly. In rare cases the configuration files may have to be
    restored from backup, or edited to back out any changes that were
    automatically made by the upgrade package.
  - Flatten and rebuild: If the OS becomes unusable, you can use
    orchestration tools to reinstall the previous OS release from
    scratch and then automatically rebuild the configuration.
  - Backup and restore: Another common strategy is to restore to a
    previous state via a backup captured before the upgrade.

### Third Party Packages

Third party packages are rare in the *network device* world. Because the
network OS is usually proprietary, third party packages are usually
packaged by the network device vendor and upgrades of those packages is
handled by the network device upgrade system.

Third party packages in *Linux host* world often use the same package
system as the distribution into which it is to be installed (for
example, Debian uses `apt-get`). Or the package may be compiled and
installed by the sysadmin. Configuration and executable files generally
follow the same filesystem hierarchy standards as other applications.

## Upgrading Cumulus Linux Devices: Strategies and Processes

Because Cumulus Linux is both Linux *and* a network device, it has
characteristics of both paradigms. The following describes the Cumulus
Linux paradigm with respect to upgrade planning and execution.

### Automated Configuration Is Preferred over Manual Configuration

Because Cumulus Linux *is* Linux, Cumulus Networks recommends that even
with small networks or test labs, network admins should make the jump to
deploy, provision, configure, and upgrade switches using automation from
the very beginning. The small up front investment of time spent learning
an orchestration tool, even to provision a small number of Cumulus Linux
devices, will pay back dividends for a long time. The biggest gain is
realized during the upgrade process, where the network admin can quickly
upgrade dozens of devices in a repeatable manner.

Switches, like servers, should be treated like *[cattle, not
pets](https://www.google.com/search?q=cattle+not+pets).*

### Out-of-Band Management Is Worth the Investment

Because network devices are reachable via the IP addresses on the front
panel ports, many network admins of small-to-medium sized networks use
*in-band* networks to manage their switches. In this design, management
traffic like SSH, SNMP, and console server connections use the same
networks that regular network traffic traverses - there is no separation
between the *management plane* and the *data plane*. Larger data centers
create a separate *out-of-band* network with a completely separate
subnet and reachability path to attach to the management ports - that is
accessible via eth0 and the serial console.

There are many scenarios where in-band networking can fail and leave the
network admin waiting for someone to drive to the data center or remote
site to connect directly to the console of a misconfigured or failing
device. The cost of one outage would usually more than pay for the
investment in a separate network. For even more security, attach
remote-controllable power distribution units (PDUs) in each rack to the
management network, so you can have complete control to remote power
cycle every device in that rack.

{{<notice tip>}}

However, if an out-of-band network is not available for you to upgrade,
you can use {{<kb_link url="knowledge-base/Installing-and-Upgrading/Upgrading/Using-dtach-for-In-band-apt-get-Upgrades/" text="the dtach tool" >}}
instead to upgrade in band.

{{</notice>}}

### Pre-Deployment Testing of New Releases Is Advised and Enabled 

White box switches and virtualization (Cumulus VX) bring the cost of
networking devices down, so the ability for network admins to test their
own procedures, configurations, applications, and network topology in an
appropriately-sized lab topology becomes extremely affordable.

### Understanding the Locations of Configuration Data is Required for Successful Upgrades, Migration, and Backup

As with other Linux distributions, the `/etc` directory is the primary
location for all configuration data in Cumulus Linux. The following list
is a likely set of files that should be backed up and migrated to a new
release, but any file that has been changed would need to be examined.
Cumulus Networks recommends you consider making the following files and
directories part of a backup strategy.

#### Network Configuration Files

| File Name and Location    | Explanation                                                                                          | Cumulus Linux Documentation                                                                                                                                                                 | Debian Documentation                                                                                                         |
| ------------------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| /etc/network/             | Network configuration files, most notably `/etc/network/interfaces` and `/etc/network/interfaces.d/` | [Switch Port Attributes](/cumulus-linux-36/Layer-1-and-Switch-Ports/Interface-Configuration-and-Management/Switch-Port-Attributes)                                                  | N/A                                                                                                                          |
| /etc/resolv.conf          | DNS resolution                                                                                       | Not unique to Cumulus Linux: [wiki.debian.org/NetworkConfiguration\#The\_resolv.conf\_configuration\_file](https://wiki.debian.org/NetworkConfiguration#The_resolv.conf_configuration_file) | [www.debian.org/doc/manuals/debian-reference/ch05.en.html](https://www.debian.org/doc/manuals/debian-reference/ch05.en.html) |
| /etc/frr/                 | Routing application (responsible for BGP and OSPF)                                                   | [FRRouting Overview](/cumulus-linux-36/Layer-3/FRRouting-Overview/)                                                                                                                 | N/A                                                                                                                          |
| /etc/hostname             | Configuration file for the hostname of the switch                                                    | [Quick Start Guide](/cumulus-linux-36/Quick-Start-Guide/#configure-the-hostname-and-timezone)                                                | [wiki.debian.org/HowTo/ChangeHostname](https://wiki.debian.org/HowTo/ChangeHostname)                                         |
| /etc/cumulus/acl/\*       | Netfilter configuration                                                                              | [Netfilter - ACLs](/cumulus-linux-36/System-Configuration/Netfilter-ACLs/)                                                                                                        | N/A                                                                                                                          |
| /etc/cumulus/ports.conf   | Breakout cable configuration file                                                                    | [Switch Port Attributes\#ConfiguringBreakoutPorts](/cumulus-linux-36/Layer-1-and-Switch-Ports/Interface-Configuration-and-Management/Switch-Port-Attributes/#configuring-breakout-ports)                                                   | N/A; please read the guide on breakout cables                                                                                |
| /etc/cumulus/switchd.conf | Switchd configuration                                                                                | [Configuring switchd](/cumulus-linux-36/System-Configuration/Configuring-switchd)                                                                                                   | N/A; please read the guide on switchd configuration                                                                          |

#### Additional Commonly Used Files

| File Name and Location          | Explanation                                                                                                                                                                                                                                                                                                           | Cumulus Linux Documentation                                                                                                                                       | Debian Documentation                                                                                                         |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| /etc/motd                       | Message of the day                                                                                                                                                                                                                                                                                                    | Not unique to Cumulus Linux                                                                                                                                       | [wiki.debian.org/motd\#Wheezy](https://wiki.debian.org/motd#Wheezy)                                                          |
| /etc/passwd                     | User account information                                                                                                                                                                                                                                                                                              | Not unique to Cumulus Linux                                                                                                                                       | [www.debian.org/doc/manuals/debian-reference/ch04.en.html](https://www.debian.org/doc/manuals/debian-reference/ch04.en.html) |
| /etc/shadow                     | Secure user account information                                                                                                                                                                                                                                                                                       | Not unique to Cumulus Linux                                                                                                                                       | [www.debian.org/doc/manuals/debian-reference/ch04.en.html](https://www.debian.org/doc/manuals/debian-reference/ch04.en.html) |
| /etc/group                      | Defines user groups on the switch                                                                                                                                                                                                                                                                                     | Not unique to Cumulus Linux                                                                                                                                       | [www.debian.org/doc/manuals/debian-reference/ch04.en.html](https://www.debian.org/doc/manuals/debian-reference/ch04.en.html) |
| /etc/lldpd.conf                 | Link Layer Discover Protocol (LLDP) daemon configuration                                                                                                                                                                                                                                                              | [Link Layer Discovery Protocol](/cumulus-linux-36/Layer-2/Link-Layer-Discovery-Protocol/)                                                                 | [packages.debian.org/wheezy/lldpd](https://packages.debian.org/wheezy/lldpd)                                                 |
| /etc/lldpd.d/                   | Configuration directory for `lldpd`                                                                                                                                                                                                                                                                                   | [Link Layer Discovery Protocol](/cumulus-linux-36/Layer-2/Link-Layer-Discovery-Protocol/)                                                                 | [packages.debian.org/wheezy/lldpd](https://packages.debian.org/wheezy/lldpd)                                                 |
| /etc/nsswitch.conf              | Name Service Switch (NSS) configuration file                                                                                                                                                                                                                                                                          | [TACACS Plus](/cumulus-linux-36/System-Configuration/Authentication-Authorization-and-Accounting/TACACS-Plus)                                             | N/A                                                                                                                          |
| /etc/ssh/                       | SSH configuration files                                                                                                                                                                                                                                                                                               | [SSH for Remote Access](/cumulus-linux-36/System-Configuration/Authentication-Authorization-and-Accounting/SSH-for-Remote-Access)                         | [wiki.debian.org/SSH](https://wiki.debian.org/SSH)                                                                           |
| /etc/sudoers and /etc/sudoers.d | Best practice is to place these changes in `/etc/sudoers.d/` instead of `/etc/sudoers` itself, as changes in the former directory are not lost on upgrade. Customers upgrading from a release prior to 3.2 (such as 3.1.2) to a 3.2 or later release should be aware the `sudoers` file changed in Cumulus Linux 3.2. | [Using sudo to Delegate Privileges](/cumulus-linux-36/System-Configuration/Authentication-Authorization-and-Accounting/Using-sudo-to-Delegate-Privileges) |                                                                                                                              |

  - If you are using the root user account, consider including `/root/`.
  - If you have custom user accounts, consider including `/home/<username>/`.

#### Files That Should Never Be Migrated Between Versions or Boxes

| File Name and Location | Explanation                                                                                                                      |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| /etc/adjtime           | System clock adjustment data. NTP manages this automatically. It is incorrect when the switch hardware is replaced. Do not copy. |
| /etc/bcm.d/            | Per-platform hardware configuration directory, created on first boot. Do not copy.                                               |
| /etc/mlx/              | Per-platform hardware configuration directory, created on first boot. Do not copy.                                               |
| /etc/blkid.tab         | Partition table. It should not be modified manually. Do not copy.                                                                |
| /etc/blkid.tab.old     | A previous partition table; it should not be modified manually. Do not copy.                                                     |
| /etc/cumulus/init      | Platform hardware-specific files. Do not copy.                                                                                   |
| /etc/default/clagd     | Created and managed by `ifupdown2`. Do not copy.                                                                                 |
| /etc/default/grub      | Grub `init` table; it should not be modified manually.                                                                           |
| /etc/default/hwclock   | Platform hardware-specific file. Created during first boot. Do not copy.                                                         |
| /etc/init              | Platform initialization files. Do not copy.                                                                                      |
| /etc/init.d/           | Platform initialization files. Do not copy.                                                                                      |
| /etc/fstab             | Static info on filesystem. Do not copy.                                                                                          |
| /etc/image-release     | System version data. Do not copy.                                                                                                |
| /etc/os-release        | System version data. Do not copy.                                                                                                |
| /etc/lsb-release       | System version data. Do not copy.                                                                                                |
| /etc/lvm/archive       | Filesystem files. Do not copy.                                                                                                   |
| /etc/lvm/backup        | Filesystem files. Do not copy.                                                                                                   |
| /etc/modules           | Created during first boot. Do not copy.                                                                                          |
| /etc/modules-load.d/   | Created during first boot. Do not copy.                                                                                          |
| /etc/sensors.d         | Platform-specific sensor data. Created during first boot. Do not copy.                                                           |
| /root/.ansible         | Ansible tmp files. Do not copy.                                                                                                  |
| /home/cumulus/.ansible | Ansible tmp files. Do not copy.                                                                                                  |

### Upgrading Switches in an MLAG Pair

If you have a pair of Cumulus Linux switches as part of an 
[MLAG (multi-chassis link aggregation) pair](/cumulus-linux-36/Layer-2/Multi-Chassis-Link-Aggregation-MLAG),
you should only upgrade each switch when it is in the *secondary role*.

{{%notice warning%}}

If you are upgrading from Cumulus Linux 2.y.z to Cumulus Linux 3.y.z,
during the time when one switch in the pair is on Cumulus Linux 3.y.z
and the other switch in the pair is on Cumulus Linux 2.y.z, a complete
outage occurs on these switches and their associated network segments.

{{%/notice%}}

The upgrade path is as follows:

1.  Upgrade Cumulus Linux on the switch already in the secondary role.
    This is the switch with the higher `clagd-priority` value.

2.  Set the switch in the secondary role into the primary role by
    setting its `clagd-priority` to a value lower than the
    `clagd-priority` setting on the switch in the primary role.
    
        cumulus@switch:~$ sudo clagctl priority VALUE

3.  Upgrade the switch that just took on the secondary role.

4.  Put that switch into the primary role again if you so choose.
    
        cumulus@switch:~$ sudo clagctl priority VALUE

{{%notice note%}}

Do not use NCLU to change priority. The NCLU command causes `clagd` to
restart, which might not be desired.

{{%/notice%}}

For more information about setting the priority, see 
[Understanding Switch Roles](/cumulus-linux-36/Layer-2/Multi-Chassis-Link-Aggregation-MLAG/#understanding-switch-roles-and-setting-priority).

## Upgrading Cumulus Linux: Choosing between a Binary Install vs. Package Upgrade

You can upgrade Cumulus Linux in one of two ways:

  - Upgrade only the changed packages using the`  sudo -E apt-get
    update `and` sudo` `-E apt-get upgrade` command. **This is the
    preferred method**.
  - Perform a binary (full image) install of the new version, using
    ONIE. This is used when moving between major versions or if you want
    to install a clean image.

There are advantages and disadvantages to using these methods, which are
outlined below.

### Upgrading Using Package Installs (apt-get update && apt-get upgrade)

Pros:

  - Configuration data stays in place while the packages are upgraded.
    In the event that the new version changes a configuration file, and
    you have also changed the configuration file, a prompt appears
    during the upgrade process asking which version you want to use or
    whether you want to evaluate the differences.
  - Third-party applications stay in place.

Cons:

  - This method works only if you are upgrading to a later minor release
    (like 3.1.x to 3.2.y), or to a later maintenance release from an
    earlier version of that minor release (for example, 2.5.2 to 2.5.5
    or 3.0.0 to 3.0.1).
  - Rollback is quite difficult and tedious.
  - You can't choose the exact release version that you want to run.
  - When you upgrade, you upgrade all packages to the latest available
    version.
  - The upgrade process takes a while to complete, and various switch
    functions may be intermittently available during the upgrade.
  - Some upgrade operations will terminate SSH sessions on the in-band
    (front panel) ports, leaving the user unable to monitor the upgrade
    process. As a workaround, use the {{<kb_link url="knowledge-base/Installing-and-Upgrading/Upgrading/Using-dtach-for-In-band-apt-get-Upgrades/" text="dtach tool" >}}.
  - Just like the binary install method, you may have to reboot after
    the upgrade, lengthening the downtime.
  - After you upgrade, user names and group names created by packages
    may be different on different switches, depending the configuration
    and package installation history.

{{%notice warning%}}

**Network Disruptions When Updating/Upgrading**

The `sudo  -E  apt-get upgrade` and `sudo -E  apt-get install` commands 
cause disruptions to network services:

  - The `sudo -E apt-get upgrade` command might result in
    services being restarted or stopped as part of the upgrade process.
  - The `sudo -E apt-get install` command might disrupt core
    services by changing core service dependency packages.

In some cases, installing new packages with `sudo -E apt-get install` 
might also upgrade additional existing packages due to
dependencies. To review potential issues before installing, run 
`sudo -E apt-get install --dry-run` to see the additional packages that will
be installed or upgraded.

{{%/notice%}}

{{%notice warning%}}

If services are stopped, a reboot may be required before those services
can be started again.

{{%/notice%}}

To upgrade the switch by updating the packages:

1.  Back up the configurations from the switch.

2.  Fetch the latest update meta-data from the repository.
    
        cumulus@switch$ sudo -E apt-get update

3.  Upgrade all the packages to the latest distribution.
    
        cumulus@switch$ sudo -E apt-get upgrade
    
    {{%notice note%}}
    
**Important**
    
During an upgrade to 3.6.0 from 3.5 or earlier, certain services
    might be stopped. These services are not restarted until after the
    switch reboots, which results in some functionality being lost
    during the upgrade process.
    
During the upgrade process, you will see messages similar to the
    following:
    
        /usr/sbin/policy-rc.d returned 101, not running 'stop switchd.service'
        usr/sbin/policy-rc.d returned 101, not running 'start switchd.service'
    
At the end of the upgrade, if a reboot is required, you see the
    following message:
    
        *** Caution: Service restart prior to reboot could cause unpredictable behavior
        *** System reboot required ***
    
**Do not restart services manually until after rebooting**, or
    services will fail.
    
For upgrades post 3.6.0, if no reboot is required after the upgrade
    completes, the upgrade will stop and restart all upgraded services
    and will log messages in the `/var/log/syslog` file similar to the
    ones shown below. (In the examples below, only the `frr` package was
    upgraded.)
    
        Policy: Service frr.service action stop postponed
        Policy: Service frr.service action start postponed
        Policy: Restarting services: frr.service
        Policy: Finished restarting services
        Policy: Removed /usr/sbin/policy-rc.d
        Policy: Upgrade is finished
    
If the upgrade process encounters changed configuration files that
    have new versions in the release to which you are upgrading, you see
    a message similar to this:  
    `Configuration file '/etc/frr/daemons'`  
    `  ==> Modified (by you or by a script) since installation. `  
    `  ==> Package distributor has shipped an updated version. `  
    `  What would you like to do about it ? Your options are: `  
    `  Y or I : install the package maintainer's version `  
    `  N or O : keep your currently-installed version `  
    `  D : show the differences between the versions `  
    `  Z : start a shell to examine the situation `  
    `  The default action is to keep your current version. `  
    `*** daemons (Y/I/N/O/D/Z) [default=N] ?`
    
To see the differences between the currently installed version and
    the new version, type `D`.
    
To keep the currently installed version, type `N`. The new package
    version is installed with the suffix `_.dpkg-dist` (for example,
    `/etc/frr/daemons.dpkg-dist`). When upgrade is complete and
    **before** you reboot, merge your changes with the changes from the
    newly installed file.
    
To install the new version, type `I`. Your currently installed
    version is saved with the suffix `.dpkg-old`.
    
When the upgrade is complete, you can search for the files with the
    `sudo find / -mount -type f -name '*.dpkg-*'` command.
    
    {{%/notice%}}

4.  Reboot the switch if the upgrade messages indicate that a system
    restart is required.
    
        cumulus@switch$ sudo -E apt-get upgrade
              ... upgrade messages here ...
         
        *** Caution: Service restart prior to reboot could cause unpredictable behavior
        *** System reboot required ***
        cumulus@switch$ sudo reboot

5.  Verify correct operation with the old configurations on new version.

{{%notice note%}}

After you successfully upgrade Cumulus Linux, you may notice some some
results that you may or may not have expected:

  - The `sudo -E apt-get upgrade` command always updates the
    operating system to the most current version. If you are currently
    running Cumulus Linux 3.0.1 and run the `sudo -E apt-get upgrade` 
    command on that switch, the packages are upgraded to the
    latest versions contained in the latest 3.y.z release.
  - When you run the `cat /etc/image-release` command, the output still
    shows the version of Cumulus Linux from the last binary install. If
    you installed Cumulus Linux 3.1.0 as a full image install and then
    upgraded to 3.2.0 using the `sudo -E apt-get upgrade` command, the 
    output from `/etc/image-release` still shows: `IMAGE_RELEASE=3.0.0`. 
    To see the current version of all the Cumulus Linux packages running 
    on the switch, run the `dpkg --list` or `dpkg -l` command.

{{%/notice%}}

#### Package Upgrade Notes

  - If you are using some forms of [network virtualization](/cumulus-linux-36/Network-Virtualization/),
    including [VMware NSX-V](/cumulus-linux-36/Network-Virtualization/Virtualization-Integrations/Integrating-Hardware-VTEPs-with-VMware-NSX-V)
    or [Midokura MidoNet](/cumulus-linux-36/Network-Virtualization/Virtualization-Integrations/Integrating-Hardware-VTEPs-with-Midokura-MidoNet-and-OpenStack),
    you may have updated the
    `/usr/share/openvswitch/scripts/ovs-ctl-vtep` file. This file is not
    marked as a configuration file, so if the file contents change in a
    newer version of Cumulus Linux, they will overwrite any changes you
    made to the file. Cumulus Networks recommends you back up this file
    before upgrading.

### Upgrading via Binary Install (ONIE)

Pros:

  - You choose the exact version that you want to upgrade to.
  - This is the only method for upgrading to a new major (X.0) version.
    For example, when you are upgrading from 2.5.5 to 3.0.

Cons:

  - Configuration data must be moved to the new OS via ZTP while the OS
    is first booted, or soon afterwards via out-of-band management.
  - Moving the configuration file can go wrong in various ways:
      - Identifying all the locations of config data is not always an
        easy task. See section above on 
        [Understanding the Locations of Configuration Data](/cumulus-linux-36/Installation-Management/Upgrading-Cumulus-Linux/#understanding-the-locations-of-configuration-data-is-required-for-successful-upgrades-migration-and-backup).
      - Config file changes in the new version may cause merge conflicts
        that go undetected.
  - If config files aren't restored correctly, the user may be unable to
    attach to the switch from in-band management. Hence, out-of-band
    connectivity (eth0 or console) is recommended.
  - The installer takes a while to complete.
  - Third-party apps must be reinstalled and reconfigured afterwards.

To upgrade the switch by running a binary install:

1.  Back up the configurations off the switch.

2.  Install the binary image, following the instructions at 
    [Installing a New Cumulus Linux Image](/cumulus-linux-36/Installation-Management/Installing-a-New-Cumulus-Linux-Image).

3.  Restore the configuration files to the new version - ideally via
    automation.

4.  Verify correct operation with the old configurations on the new
    version.

5.  Reinstall third party apps and associated configurations.

## Rolling Back a Cumulus Linux Installation

Rolling back to an earlier release after upgrading the packages on the
switch follows the same procedure as described for the Linux host OS
rollback above. There are three main strategies, and all require
detailed planning and execution:

  - Back out individual packages: If the problematic package is
    identified, the network admin can downgrade the affected package
    directly. In rare cases the configuration files may have to be
    restored from backup, or edited to back out any changes that were
    automatically made by the upgrade package.
  - Flatten and rebuild: If the OS becomes unusable, you can use
    orchestration tools to reinstall the previous OS release from
    scratch and then automatically rebuild the configuration.
  - Backup and restore: Another common strategy is to restore to a
    previous state via a backup captured before the upgrade.

Which method you employ is specific to your deployment strategy, so
providing detailed steps for each scenario is outside the scope of this
document.

## Third Party Package Considerations

Note that if you install any third party apps on a Cumulus Linux switch,
any configuration data will likely be installed into the `/etc`
directory, but it is not guaranteed. It is the responsibility of the
network admin to understand the behavior and config file information of
any third party packages installed on a Cumulus Linux switch.

After you upgrade the OS using a full binary install, you will need to
reinstall any third party packages or any Cumulus Linux add-on packages,
such as `vxsnd` or `vxrd`.

## Installation and Upgrade Workflow in Cumulus Linux 3.0 and Later

Beginning with version 3.0, Cumulus Linux completely embraces the Linux
and Debian upgrade workflow. In this paradigm, a base image is installed
using an installer, then any upgrades within that release train (major
version, like 3.y.z) are done using `apt-get update && apt-get upgrade`.
Any packages that have been changed since the base install get upgraded
in place from the repository.

The huge advantage of this approach is that all switch configuration
files remain untouched, or in rare cases merged (using the Debian merge
function) during the package upgrade.

However, when upgrading a switch from a previous release train - for
example, Cumulus Linux 2.5 - a mechanism is required to migrate the
configuration files over to the new installation. This is the perfect
opportunity to use automation and orchestration tools to backup the
configuration files, examine them to verify correctness with the new
version, and then to redeploy the configuration files on the new
installation.

## Using Snapshots during Upgrades

[Snapshots](/cumulus-linux-36/Installation-Management/Using-Snapshots)
can aid you when upgrading the switch operating system. Cumulus Linux
takes two snapshots automatically during the upgrade, one right before
you run `apt-get upgrade`, and one right after. This way, if something
goes wrong with the upgrade, or you need to revert to the earlier
version, you can roll back to the snapshot.

## Caveats When Migrating Configuration Files Between Cumulus Linux 2.5.z and 3.0 and Later

Generally, the configuration files in Cumulus Linux 2.5.z should be able
to migrate to version 3.0 or later without any problems, but there are
some known issues listed below and there may be additional issues with a
customer's particular setup.

Known caveats when migrating files from version 2.x to 3.0 or later:

  - Some configuration files should never be migrated between versions
    or while replacing hardware. The 
    [Files that Should Never be Migrated](#files-that-should-never-be-migrated-between-versions-or-boxes)
    table above contains a list of files that should never be migrated.
  - `/etc/passwd` and `/etc/shadow` should not be migrated to the new
    version directly. The example below and the ansible script included
    with [Config File Migration Script](https://github.com/CumulusNetworks/config-backup-upgrade-helper)
    explicitly excludes these two files from the backup archive. The
    default password for the *cumulus* user must be changed, and any
    locally created users should be added to the new installation after
    the upgrade completes.
  - `/etc/apt/sources.list` must be completely updated with a new 3.0 or
    later repository and repository structure. Repositories from Cumulus
    Linux 2.5 must be removed. If there are any custom repositories on
    the switch, they need to be migrated into the new `sources.list`
    file or the `sources.d/` directory.

## Using the Config File Migration Script to Identify and Move Files to Cumulus Linux 3.0 and Later

You can use the [Config File Migration Script](https://github.com/CumulusNetworks/config-backup-upgrade-helper)
with the `--backup` option to create a backup archive of configuration
files in version 2.5, copy them off the box, then install them on the
new version switch. Note that you need to follow the previous section
about caveats when migrating configuration files.

The following example excludes `/etc/apt`, `/etc/passwd` and
`/etc/shadow` from the backup archive.

1.  Back up the version 2.5.z files.
    
    **Optional:** Use the Ansible playbook included with the 
    [Config File Migration script](https://github.com/CumulusNetworks/config-backup-upgrade-helper)
    to automate the backup of all your Cumulus Linux 2.5 switches. See
    the section below on 
    [Using Automation Tools to Backup Configurations](#using-automation-tools-to-back-up-2-5-z-configurations)
    for more details.
    
        # Make a temp dir
        loc=$(mktemp -d)
        # Create a backup archive to the temp dir
        sudo ./config_file_changes --backup --backup_dir $loc --exclude /etc/apt,/etc/passwd,/etc/shadow
        # Copy the archive and log file to an external server
        sudo scp -r $loc/* user@my_external_server:.

2.  [Install Cumulus Linux 3.0 or later onto the switch using ONIE](/cumulus-linux-36/Installation-Management/Installing-a-New-Cumulus-Linux-Image).

3.  Reinstall the files from the config file archive to the newly
    installed switch.
    
        # On the switch, copy the config file archive back from the server:
        scp user@my_external_server:PATH/SWITCHNAME-config-archive-DATE_TIME.tar.gz .
        # Untar the archive to the root of the box
        sudo tar -C / -xvf SWITCHNAME-config-archive-DATE_TIME.tar.gz
    
    {{%notice warning%}}
    
Be aware that version 2.5.z configurations are not guaranteed to
    work in Cumulus Linux 3.0 or later. You should test the restoration
    and proper operation of the Cumulus Linux 2.5.z configuration in
    Cumulus Linux 3.0 or later on a non-production switch or in a
    Cumulus VX image, since every deployment is unique.
    
    {{%/notice%}}

## Using Automation Tools to Back Up 2.5.z Configurations

Adopting the use of orchestration tools like Ansible, Chef or Puppet for
configuration management greatly increases the speed and accuracy of the
next major upgrade; they also enable the quick swap of failed switch
hardware. Included with the [Config Migration Script](https://github.com/CumulusNetworks/config-backup-upgrade-helper)
is an Ansible playbook that can be used to create a backup archive of
Cumulus Linux 2.5.z switch configuration files and to retrieve them to a
central server - automating step 1 of the previous section for all
deployed Cumulus Linux 2.5.z switches. This is a quick start on the road
to setting up automated configuration and control for your deployment.
For more details on integrating automation into your Cumulus Linux
deployment, see the 
[Automation Solutions section](https://cumulusnetworks.com/solutions/automation/) 
on cumulusnetworks.com.

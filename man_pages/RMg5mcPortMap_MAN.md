Copyright 2017 Dell, Inc. All rights reserved.

# RMg5mcPortMap
---

## About

***RMg5mcPortMap*** is a BASH utility to enable and disable port forwarding to and from management controllers (MC's).

* By default on the RackManager, all port forwardings to and from MC's are disabled
* This utility must be executed with the `enable` subcommand to enable port fowarding rules for target MC's
* All applied changes will be logged to: `/var/log/rackmanager/RMg5mcPortMap.log`

## Usage

### To run:

* Login as root on the RackManager
* Enter `RMg5mcPortMap [OPTIONS] <SUBCOMMAND>`

### SUBCOMMANDS:
                
    show     -- show port map status for specified target MC,
                by default will show port map for MC1_1 if target is not specified
                e.g. RMg5mcPortMap [-vvv] -t MC1_1 show
                e.g. RMg5mcPortMap [-vvv] -m 1 show

    enable   -- enable port forwarding for target MC,
                by default will enable port forwarding on MC1_1 if target is not specified,
                depending on the config, some specific ports may be enabled while others are disabled
                e.g. RMg5mcPortMap [-vvv] -t MC1_1 enable
                e.g. RMg5mcPortMap [-vvv] -m 1 enable

    disable  -- disable all port forwarding rules for target MC,
                by default will disable port forwarding on MC1_1 if target is not specified
                e.g. RMg5mcPortMap [-vvv] -t MC1_1 disable
                e.g. RMg5mcPortMap [-vvv] -m 1 disable


### OPTIONS:

    -V                -- display version and exit
    -h                -- display usage help
    -t <target>       -- target to enable/disable/show
                         e.g. -t MC1_1
    -m <mc_num>       -- target MC num, will be ignored if used with -t
    -v                -- verbose output (can be repeated for additional level of verbosity)
       -v             -- level 1 verbosity - show command will show enabled ports
       -vv            -- level 2 verbosity - show command will show port mappings with more detail
       -vvv           -- level 3 verbosity - show debug messages


### Examples:

    # get RMg5mcPortMap version
    [root@RackManager ~]# RMg5mcPortMap -V
    RMg5mcPortMap:  Version:   1.0.0
    
    # apply default port forwarding rules to MC1_1
    [root@RackManager ~]# RMg5mcPortMap -vv -t MC1_1 enable
    MC1_1: Enabled
      
    # disable all port forwarding rules for MC1_1
    [root@RackManager ~]# RMg5mcPortMap -vv -t MC1_1 disable
    MC1_1: Disabled
      
    # show applied port forwarding rules for MC1_1
    [root@RackManager ~]# RMg5mcPortMap -vv -t MC1_1 show
    MC1_1: Enabled
      Mgmt1 Incoming:
        SSH:               tcp 60010 --> tcp 22     (10.208.63.145:60010)
        HTTP:              tcp 60012 --> tcp 80     (10.208.63.145:60012)
        SNMP:              udp 60013 --> udp 161    (10.208.63.145:60013)
        HTTPS:             tcp 60014 --> tcp 443    (10.208.63.145:60014)
        VIRTUAL MEDIA:     tcp 60016 --> tcp 5900   (10.208.63.145:60016)
        VNC:               tcp 60017 --> tcp 5901   (10.208.63.145:60017)
      Mgmt2 Incoming:
        SSH:               tcp 60010 --> tcp 22     (192.168.15.85:60010)
        HTTP:              tcp 60012 --> tcp 80     (192.168.15.85:60012)
        SNMP:              udp 60013 --> udp 161    (192.168.15.85:60013)
        HTTPS:             tcp 60014 --> tcp 443    (192.168.15.85:60014)
        VIRTUAL MEDIA:     tcp 60016 --> tcp 5900   (192.168.15.85:60016)
        VNC:               tcp 60017 --> tcp 5901   (192.168.15.85:60017)
      Mgmt1 Outgoing:
        ICMP:              ACCEPT
        ALL:               ACCEPT
      Mgmt2 Outgoing:
        ICMP:              DROP
        ALL:               DROP

## Description

***RMg5mcPortMap*** offers three subcommands: `enable`, `disable`, and `show`.

### Enable

The enable subcommand will mark the targeted MC as enabled for RMG5MCPortMapService to apply the actual port forwarding rules.
* The ruls listed in the config file `/etc/opt/dell/rm-tools/RMG5MCPortMap.conf` will be applied by RMG5MCPortMapService
  * The config file format is basically that of a list of rules to apply to specific MC's
    * By default, this config file will specify the default rules for every possible managed MC
  * Each rule either maps an incoming RackManager port to a mc port, or an outgoing mc port to a RackManager port
    * Packets incoming on mgmt1 and/or mgmt2 ports can be configured to be forwarded to specific mc ports
    * Packets outgoing mc's that are destined for some remote host on the RackManager's mgmt1 or mgmt2 network,
  can be configured to be forwarded out the mgmt1 ***or*** mgmt2 interface, but ***not both***
    * For either incoming or outgoing packets, specific protocols (destination ports) can be disabled (or dropped)
      * For instance, the default port forwarding rules disable incoming Telnet (tcp port 23)
* Once a target has been enabled, RMG5MCPortMapService will keep the rules persistent across reboots/hotplugs/network restarts
* All changes made by RMg5mcPortMap will logged
  
The enable subcommand requires a target:
* The target can be any present MC in the `/etc/hosts` file
* If no target is specified in the command, by default the target will be `MC1_1`

### Disable

The disable subcommand will mark the targeted MC as disabled for RMG5MCPortMapService to disable all port forwarding rules (both incoming and outgoing) for the specified target.

Thus, the disable subcommand requires a target:
* The target can be any present MC in the `/etc/hosts` file
* If no target is specified in the command, by default the target will be `MC1_1`

### Show

The show subcommand will show all the applied mapped port forwarding rules for the specified target.

Note that each level of additional verbosity will show more details.

The show subcommand also requires a target:
* The target can be any present MC in the `/etc/hosts` file
* If no target is specified in the command, by default the target will be `MC1_1`

## Installation, Path, and Dependencies:
* ***RMg5mcPortMap*** is included in the rackmanager-tools RPM, and is installed by default as part of the RackManager Toolkit
* It is installed on the RM at `/opt/dell/rm-tools/RMg5mcPortMap/*`
* The first line of the program includes a sheebang line `#!/usr/bin/bash`
* Depends on bash, RMG5MCPortMapService, iptables, and a valid mc port map config file
* Logs events to `/var/log/rackmanager/RMg5mcPortMap.log`

## See Also:
* RMG5MCPortMapService docs -- applies the actual port forwarding rules based on the config file and targets enabled by RMg5mcPortMap
* RMnodePortMap and RMNodePortMapService docs -- for configuring/enabling/disabling port forwarding to and from nodes (iDRACs)
  
## Limitations:
* Target must be of format `MCX_1`
* MC must be present in `/etc/hosts` in order to configure port forwarding
* Outgoing port forwarding rules can only be applied to one mgmt interface (mgmt1 or mgmt2) per protocol
  * For instance, you can configure DNS packets from a MC to route out either mgmt1 or mgmt2, not both
* Outgoing port forwarding requires additional configuring to the MC itself:
  * MC must be configured to route outgoing traffic internally through the Rackmanager instead of its own external port
  * It also recommended to set the MC's DNS server to a server that exists on either the mgmt1 or mgmt2 networks
* It can take up to 20 seconds between when the target is enabled and when RMG5MCPortMapService actually applies the rules

## Known Issues:
* None

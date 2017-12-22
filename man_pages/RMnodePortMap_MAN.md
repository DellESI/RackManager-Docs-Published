Copyright 2017 Dell, Inc. All rights reserved.

# RMnodePortMap
---

## About

***RMnodePortMap*** is a BASH utility to enable and disable port forwarding to and from nodes.

* By default on the RackManager, all port forwardings to and from nodes are disabled
* This utility must be executed with the `enable` subcommand to enable port fowarding rules for target nodes
* All applied changes will be logged to: `/var/log/rackmanager/RMnodePortMap.log`

## Usage

### To run:

* Login as root on the RackManager
* Enter `RMnodePortMap [OPTIONS] <SUBCOMMAND>`

### SUBCOMMANDS:

    show     -- show port map status for specified target (rack, block, or single node),
                by default will show port map for all nodes if target is not specified
                e.g. RMnodePortMap [-vvv] -t Rack1 show
                e.g. RMnodePortMap [-vvv] -r 1 show

    enable   -- enable port forwarding for target (rack, block, or single node),
                by default will enable port forwarding on all nodes if target is not specified,
                depending on the config, some specific ports may be enabled while others are disabled
                e.g. RMnodePortMap [-vvv] -t Rack1-Block1-Sled1-Node1 enable
                e.g. RMnodePortMap [-vvv] -r 1 -b 1 -s 1 enable

    disable  -- disable all port forwarding rules for target (rack, block, or single node),
                by default will disable port forwarding on all nodes if target is not specified
                e.g. RMnodePortMap [-vvv] -t Rack1-Block1 disable
                e.g. RMnodePortMap [-vvv] -r 1 -b 1 disable


### OPTIONS:

    -V                -- display version and exit
    -h                -- display usage help
    -t <target>       -- target to enable/disable/show, defaults to Rack1
                         e.g. -t Rack1  -- all nodes in rack
                         e.g. -t Rack1-Block1  -- all nodes in block
                         e.g. -t Rack1-Block1-Sled1  -- single sled node
    -r <rack>         -- target rack num, will be ignored if used with -t
    -b <block>        -- target block num, will be ignored if used with -t, or without -r
    -s <sled>         -- target sled num, will be ignored if used with -t, or without -r and -b
    -v                -- verbose output (can be repeated for additional level of verbosity)
       -v             -- level 1 verbosity - show command will show enabled ports
       -vv            -- level 2 verbosity - show command will show port mappings with more detail
       -vvv           -- level 3 verbosity - show debug messages


### Examples:

    # get RMnodePortMap version
    [root@RackManager ~]# RMnodePortMap -V
    RMnodePortMap:  Version:   1.0.0
    
    # apply default port forwarding rules to Rack1-Block3-Sled3
    [root@RackManager ~]# RMnodePortMap -vv -t Rack1-Block3-Sled3 enable
    Rack1-Block3-Sled3-Node1: Enabled
      
    # disable all port forwarding rules for Rack1-Block3-Sled3
    [root@RackManager ~]# RMnodePortMap -vv -t Rack1-Block3-Sled3 disable
    Rack1-Block3-Sled3-Node1: Disabled
      
    # show applied port forwarding rules for Rack1-Block3-Sled4-Node1
    [root@RackManager ~]# RMnodePortMap -vv -t Rack1-Block3-Sled4-Node1 show
    Rack1-Block3-Sled4-Node1: Enabled
      Mgmt1 Incoming:
        SSH:               tcp 53040 --> tcp 22     (10.208.63.145:53040)
        HTTP:              tcp 53042 --> tcp 80     (10.208.63.145:53042)
        SNMP:              udp 53043 --> udp 161    (10.208.63.145:53043)
        HTTPS:             tcp 53044 --> tcp 443    (10.208.63.145:53044)
        VIRTUAL MEDIA:     tcp 53046 --> tcp 5900   (10.208.63.145:53046)
        VNC:               tcp 53047 --> tcp 5901   (10.208.63.145:53047)
      Mgmt2 Incoming:
        SSH:               tcp 53040 --> tcp 22     (192.168.15.85:53040)
        HTTP:              tcp 53042 --> tcp 80     (192.168.15.85:53042)
        SNMP:              udp 53043 --> udp 161    (192.168.15.85:53043)
        HTTPS:             tcp 53044 --> tcp 443    (192.168.15.85:53044)
        VIRTUAL MEDIA:     tcp 53046 --> tcp 5900   (192.168.15.85:53046)
        VNC:               tcp 53047 --> tcp 5901   (192.168.15.85:53047)
      Mgmt1 Outgoing:
        ICMP:              ACCEPT
        ALL:               ACCEPT
      Mgmt2 Outgoing:
        ICMP:              DROP
        ALL:               DROP

## Description

***RMnodePortMap*** offers three subcommands: `enable`, `disable`, and `show`.

### Enable

The enable subcommand will mark the target nodes as enabled for RMNodePortMapService to apply the actual port forwarding rules.
* The rules listed in the config file `/etc/opt/dell/rm-tools/RMNodePortMap.conf` will be applied by RMNodePortMapService
  * The config file format is basically that of a list of rules to apply to specific nodes
    * By default, this config file will specify the default rules for every possible node
  * Each rule either maps an incoming RackManager port to a node port, or an outgoing node port to a RackManager port
    * Packets incoming on mgmt1 and/or mgmt2 ports can be configured to forward to specific node ports
    * Packets outgoing nodes that are destined for some remote host on the RackManager's mgmt1 or mgmt2 network,
      can be configured to be forwarded out the mgmt1 ***or*** mgmt2 interface, but ***not both***
    * For either incoming or outgoing packets, specific protocols (destination ports) can be disabled (or dropped)
      * For instance, the default port forwarding rules disable incoming Telnet (tcp port 23)
* Once a target has been enabled, RMNodePortMapService will keep the rules persistent across reboots/hotplugs/network restarts
* All changes made by RMnodePortMap will logged
  
The enable subcommand requires a target:
* The target can be an entire rack, a whole block, of a single sled/node
  * For a rack target, port fowarding rules will be enabled for all the present nodes in the rack
  * For a block target, port fowarding rules will be enabled for all the present nodes in the block
  * For a sled/node target, port forwarding rules will be enabled for only the target sled/node
* If no target is specified in the command, by default the target will be `Rack1`

### Disable

The disable subcommand will disable all port forwarding rules (both incoming and outgoing) for the specified target.

Thus, the disable subcommand requires a target:
* The target can be an entire rack, a whole block, of a single sled/node
  * For a rack target, port fowarding rules will be disabled for all the present nodes in the rack
  * For a block target, port fowarding rules will be disabled for all the present nodes in the block
  * For a sled/node target, port forwarding rules will be disabled for only the target sled/node
* If no target is specified in the command, by default the target will be `Rack1`

### Show

The show subcommand will show all the applied mapped port forwarding rules for the specified target.

Note that each level of additional verbosity will show more details.

The show subcommand also requires a target:
* The target can be an entire rack, a whole block, of a single sled/node
  * For a rack target, port fowarding rules will be displayed for all the present nodes in the rack
  * For a block target, port fowarding rules will be displayed for all the present nodes in the block
  * For a sled/node target, port forwarding rules will be displayed for only the target sled/node
* If no target is specified in the command, by default the target will be `Rack1`

## Installation, Path, and Dependencies:
* ***RMnodePortMap*** is included in the rackmanager-tools RPM, and is installed by default as part of the RackManager Toolkit
* It is installed on the RM at `/opt/dell/rm-tools/RMnodePortMap/*`
* The first line of the program includes a sheebang line `#!/usr/bin/bash`
* Depends on bash, RMNodePortMapService, iptables, RMNodeDiscovery, RMRedfishService, and a valid node port map config file
* Logs events to `/var/log/rackmanager/RMnodePortMap.log`

## See Also:
* RMNodePortMapService docs -- applies the actual port forwarding rules based on the config file and targets enabled by RMnodePortMap
* RMg5mcPortMapand RMG5MCPortMapService -- for configuring/enabling/disabling port forwarding to and from Management Controllers (MC)
  
## Limitations:
* Currently only supports port forwarding to/from nodes in the DSS9000 rack architecture
* Target must be of format `RackX`, `RackX-BlockX`, `RackX-BlockX-SledX`, or `RackX-BlockX-SledX-Node1`
* Nodes must be present in `/etc/hosts` in order to configure port forwarding
* Outgoing port forwarding rules can only be applied to one mgmt interface (mgmt1 or mgmt2) per protocol
  * For instance, you can configure DNS packets from a node to route out either mgmt1 or mgmt2, not both
* Outgoing port forwarding requires additional configuring to the node itself:
  * Node must be configured to route outgoing traffic internally through the Rackmanager instead of its own external port
  * To set a node's default gateway to the RackManager with racadm:
    * `racadm -r Rack1-Block1-Sled1-Node1 -u <user> -p <passwd> set iDRAC.IPv4.Gateway 10.253.4.1`
  * To set a node's DNS server to a server on the RM's mgmt1 or mgmt2 network with racadm:
    * `racadm -r Rack1-Block1-Sled1-Node1 -u <user> -p <passwd> set iDRAC.IPv4.DNS1 <DNS_IP_ADDRESS>`
* It can take up to 20 seconds between when the target is enabled and when the RMNodePortMapService actually applies the rules

## Known Issues:
* None

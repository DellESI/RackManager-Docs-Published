Copyright &copy; 2017-2018 Dell Inc. or its subsidiaries. All rights reserved.
 
# RMNodePortMapService -- RackManager's Node Port Map Service

---

## About
The ***RMNodePortMapService*** is a systemd integrated service that applies port forwarding rules to map specific RM ports to node ports,
and vice versa.

***RMNodePortMapService*** specific logs can be found on the RackManager: `/var/log/rackmanager/RMNodePortMapService.log`

### Dependencies
* RMconfig of the RMNodePortMapService subsystem
  * This sets up required default configuration files that RMNodePortMapService depends on
* RMMgtNetworkStart must have already properly set up the namespace mgmt networking
* iptables to apply port forwarding and firewall rules
* RMnodePortMap utility to enable/disable port forwarding for specific G5 MC targets

### Description
* Upon start of ***RMNodePortMapService***, first the user conf file `/etc/opt/dell/rm-tools/RMNodePortMap.conf` will be validated
  * While we do provide a default RMNodePortMap.conf with some default example port forwarding rules for all potentials nodes in
    Rack1, they are just that, default example rules. Users should really examine this file and create their own rules for the specific
    rack, block, and sled-nodes they plan to enable port forwarding on prior to use.
* If the user conf is invalid, a warning will be logged, and the service will continue to use the previously validated conf
  * Note that while invalid settings will get logged, no immediate output of the warning will be displayed to the user in their shell.
    Users should verify in the log file following changes that they were valid and accepted.
* Somge global firewall rules will be applied to setup the ability to forward ports to and from nodes
* Once ready, the service will start applying the rules listed in the conf file one by one for the enabled node targets
* For disabled targets, the service will confirm no port forwarding rules for the target exists
* Once the first run through has completed, a periodic poll will begin at a default of 20 seconds
* Every poll the service will again determine which targets are suppose to be enabled, and which are supposed to be disabled
* Based on this, the correct port forwarding rules will either be applied, confirmed, or removed
* This polling method allows for port forwarding rules to remain persistent across reboots, hotplug, and network restarts
* All events will be logged to `/var/log/rackmanager/RMNodePortMapService.log`

### RMNodePortMap.conf
* `/etc/opt/dell/rm-tools/RMNodePortMap.conf` is the editable user conf file for RMNodePortMapService
* Any edits to this file require a restart of RMNodePortMapService in order for the new configuration rules to take effect
* By default, this conf file will contain default rules for every possible Rack1 node
  * Note that while we do provide a default RMNodePortMap.conf with some default example port forwarding rules for all
    potentials nodes in Rack1, they are just that, default example rules. Users should really examine this file and create
    their own rules for the specific rack, block, and sled-nodes they plan to enable port forwarding on prior to use.
  * Note also we only provide the default settings for Rack1 nodes, thus if the RackNumber has been changed, users must manually
    add the port forwarding rules for any nodes they plan to enable port forwarding for. There is no default rules for non Rack1 nodes.
* The file format is basically that of a list of rules to apply to specific node targets
* Each rule either maps an incoming RackManager port to a node port, or an outgoing node port to a RackManager port
* Rules will only be applied by RMNodePortMapService if they correspond to an "enabled" target nodes
* Each rule must follow this basic format:

```
Rule Format: Direction, TargetID, enable|disable, tcp|udp, NodePort, ExternalMgmtPort, RM_Interface

Direction        -- Incoming or Outgoing
TargetID         -- Target ID for rack, block, or sled
                    -- e.g. Rack1  (rule will be applied to all nodes in rack)
                    -- e.g. Rack1-Block1  (rule will be applied to all nodes in block)
                    -- e.g. Rack1-Block1-Sled1  (rule will be applied to single node)
                    -- e.g. Rack1-Block1-Sled1-Node1  (rule will be applied to single node)
enable|disable   -- whether port forwarding rule should be applied or not
tcp|udp          -- rule applies to tcp or udp packets (outgoing rule can also have icmp value)
NodePort         -- the node port number  (0 - 65535)
ExternalMgmtPort -- the external RM port number  (0 - 65535)
RM_Interface     -- interface to apply port forward to (list of mgmt1 and mgmt2, e.g. mgmt2,mgmt1)
                    -- if only one specified, rule will be applied to that interface
                    -- if list specified, incoming rules will be applied to all interfaces in list
                    -- if list specified, outgoing rules will be applied to only the first interface
                       in the list that is UP

NOTES:
  - You can enable/disable all non specified outgoing protocols by using "*" for tcp|udp, NodePort, and ExternalMgmtPort
    - This only works for outgoing rules, and all 3 values must be "*"
      - e.g. "Outgoing  Rack1-Block1-Sled1-Node1  enable   *     *      *      mgmt1,mgmt2"
  - To specifically enable outgoing icmp (ping), use "icmp" for tcp|udp value, and "*" for NodePort and ExternalMgmtPort"
    - e.g. "Outgoing  Rack1-Block1-Sled1-Node1  enable   icmp  *      *      mgmt1"
  - The following are the default ports that nodes (iDRAC) listens to for connections (Incoming):
    - 22 - TCP (SSH)
    - 23 - TCP (Telnet)
    - 80 - TCP (HTTP)
    - 443 - TCP (HTTPS)
    - 623 - UDP (RMCP/RMCP+, IPMI)
    - 161 - UDP (SNMP)
    - 5900 - TCP (Virtual Console keyboard and mouse redirection, Virtual Media, Virtual Folders, and Remote File Share)
    - 5901 - TCP (VNC, only open if VNC enabled on iDRAC)
  - The following are the default ports that nodes (iDRAC) use as clients (Outgoing):
    - 25 - TCP (SMTP)
    - 53 - TCP/UDP (DNS)
    - 68 - UDP (DHCP-assigned IP address)
    - 69 - UDP (TFTP)
    - 162 - UDP (SNMP TRAP)
    - 445 - TCP (Common Internet File System, CIFS)
    - 636 - TCP/UDP (LDAP Over SSL, LDAPS)
    - 2049 - TCP/UDP (Network File System, NFS)
    - 123 - UDP (Network Time Protocol, NTP)
    - 3269 - TCP (LDAPS for global catalog, GC)

########## Example Node Specific Configuration Rules for Rack1-Block1-Sled1 ##########
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   22     51010  mgmt1,mgmt2   # incoming ssh
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   23     51011  mgmt1,mgmt2   # incoming telnet
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   80     51012  mgmt1,mgmt2   # incoming http
Incoming  Rack1-Block1-Sled1-Node1  enable   udp   161    51013  mgmt1         # incoming snmp without TLS/DTLS
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   443    51014  mgmt1,mgmt2   # incoming https
Incoming  Rack1-Block1-Sled1-Node1  disable  udp   623    51015  mgmt2         # incoming ipmi (RMCP/RMCP+)
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   5900   51016  mgmt1,mgmt2   # incoming virtual media
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   5901   51017  mgmt1,mgmt2   # incoming vnc
Outgoing  Rack1-Block1-Sled1-Node1  enable   *     *      *      mgmt1,mgmt2   # all outgoing protocols
Outgoing  Rack1-Block1-Sled1-Node1  enable   icmp  *      *      mgmt1         # outgoing ping
Outgoing  Rack1-Block1-Sled1-Node1  enable   tcp   25     25     mgmt1,mgmt2   # outgoing SMTP
Outgoing  Rack1-Block1-Sled1-Node1  enable   tcp   53     53     mgmt1,mgmt2   # outgoing DNS (tcp)
Outgoing  Rack1-Block1-Sled1-Node1  enable   udp   53     53     mgmt1,mgmt2   # outgoing DNS (udp)
Outgoing  Rack1-Block1-Sled1-Node1  enable   udp   68     68     mgmt1,mgmt2   # outgoing DHCP
Outgoing  Rack1-Block1-Sled1-Node1  disable  udp   69     69     mgmt2         # outgoing TFTP
Outgoing  Rack1-Block1-Sled1-Node1  enable   udp   123    123    mgmt1         # outgoing NTP
Outgoing  Rack1-Block1-Sled1-Node1  enable   udp   162    162    mgmt1         # outgoing snmp trap
Outgoing  Rack1-Block1-Sled1-Node1  enable   tcp   445    445    mgmt1         # outgoing CIFS
Outgoing  Rack1-Block1-Sled1-Node1  disable  tcp   636    636    mgmt1         # outgoing LDAPS (tcp)
Outgoing  Rack1-Block1-Sled1-Node1  disable  udp   636    636    mgmt1         # outgoing LDAPS (udp)
Outgoing  Rack1-Block1-Sled1-Node1  enable   tcp   2049   2049   mgmt1         # outgoing NFS (tcp)
Outgoing  Rack1-Block1-Sled1-Node1  enable   udp   2049   2049   mgmt1         # outgoing NFS (udp)
Outgoing  Rack1-Block1-Sled1-Node1  enable   tcp   3269   3269   mgmt1         # outgoing GC
######################################################################################
```

* By default, each Rack1 node target will start with the same default configuations:

```
# Rack1-Block1-Sled1
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   22     51010  mgmt1,mgmt2   # incoming ssh
Incoming  Rack1-Block1-Sled1-Node1  disable  tcp   23     51011  mgmt1,mgmt2   # incoming telnet
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   80     51012  mgmt1,mgmt2   # incoming http
Incoming  Rack1-Block1-Sled1-Node1  enable   udp   161    51013  mgmt1,mgmt2   # incoming snmp without TLS/DTLS (requests to agent)
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   443    51014  mgmt1,mgmt2   # incoming https
Incoming  Rack1-Block1-Sled1-Node1  disable  udp   623    51015  mgmt1,mgmt2   # incoming ipmi (RMCP/RMCP+)
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   5900   51016  mgmt1,mgmt2   # incoming virtual media
Incoming  Rack1-Block1-Sled1-Node1  enable   tcp   5901   51017  mgmt1,mgmt2   # incoming vnc
Outgoing  Rack1-Block1-Sled1-Node1  enable   *     *      *      mgmt1,mgmt2   # all non-specified outgoing protocols
```

* Note that while the provided default rules are only examples, the default assigned ports do follow the following convention
  which is only used to create the default rules, and is in no way a requirement. ***The assigned port number for any protocol
  can be   any unused port, and changed at any time.***

* Assigned default port number is 5XYYZ, where:
  * X stands for block number (Block range, 1-10):
    * Blocks 1-9 map to their corresponding single digit integer, e.g. Block 4 -> 4
    * Block 10 maps to 0, e.g. Block 10 -> 0
  * YY stands for sled number (Sled range, 1-12):
    * Each sled maps to its corresponding 2 digit integer, e.g. Sled 4 -> 04
  * Z maps different service protocols with the following default values:
    * SSH (tcp 22) -> 0
    * TELNET (tcp 23) -> 1
    * HTTP (tcp 80) -> 2
    * SNMP (udp 161) -> 3
    * HTTPS (tcp 443) -> 4
    * RMCP/RMCP+ (udp 623) -> 5
    * VIRTUAL MEDIA (tcp 5900) -> 6
    * VNC (tcp 5901) -> 7
* Note also that by default the above TELNET and RMCP/RMCP+ port forwarding rules are disabled and the ports will not be open, even
  when port forwarding is enabled for the node
* All other default protocol port forwarding rules are enabled by default

### See also
* RMnodePortMap - utility to enable/disable port forwarding for specific targets

### Limitations
* Functionality is tightly integrated to the DSS9000 rack architecture
  * Currently does not support forwarding to and from external utility nodes
* Nodes must be present in `/etc/hosts` in order for port forwarding rules to be applied
* Outgoing port forwarding rules can only be applied to one mgmt interface (mgmt1 or mgmt2) per protocol
  * For instance, you can configure DNS packets from a node to route out either mgmt1 or mgmt2, not both
* Outgoing port forwarding requires additional configuring to the node itself:
  * Default Gateway:
    * Node must be configured to route outgoing traffic internally through the Rackmanager instead of its own external port
    * To get a node's default gateway via racadm:
      * `racadm -r Rack1-Block1-Sled1-Node1 -u <user> -p <passwd> get iDRAC.IPv4.Gateway`
    * To set a node's default gateway to the RackManager with racadm:
      * `racadm -r Rack1-Block1-Sled1-Node1 -u <user> -p <passwd> set iDRAC.IPv4.Gateway 10.253.4.1`
  * DNS:
    * If forwarding DNS packets from a node through and out the RackManager, you may need to set the node's DNS server
    * To get a node's DNS server settings via racadm:
      * `racadm -r Rack1-Block1-Sled1-Node1 -u <user> -p <passwd> get iDRAC.IPv4.DNS1`
      * `racadm -r Rack1-Block1-Sled1-Node1 -u <user> -p <passwd> get iDRAC.IPv4.DNS2`
    * To set a node's DNS server to a server on the RM's mgmt1 or mgmt2 network with racadm:
      * `racadm -r Rack1-Block1-Sled1-Node1 -u <user> -p <passwd> set iDRAC.IPv4.DNS1 <DNS_IP_ADDRESS>`
  
## Usage
* ` systemctl {restart, start, stop} RMNodePortMapService`

Copyright &copy; 2017-2018 Dell Inc. or its subsidiaries. All rights reserved.
 
# RMG5MCPortMapService -- RackManager's G5 MC Port Map Service

---

## About
The ***RMG5MCPortMapService*** is a systemd integrated service that applies port forwarding rules to map specific RM ports to G5 MC ports,
and vice versa.

***RMG5MCPortMapService*** specific logs can be found on the RackManager: `/var/log/rackmanager/RMG5MCPortMapService.log`

### Dependencies
* RMconfig of the RMG5MCPortMapService subsystem
  * This sets up required default configuration files that RMG5MCPortMapService depends on
* RMMgtNetworkStart must have already properly set up the namespace mgmt networking
* iptables to apply port forwarding and firewall rules
* RMg5mcPortMap utility to enable/disable port forwarding for specific G5 MC targets

### Description
* Upon start of ***RMG5MCPortMapService***, first the user conf file `/etc/opt/dell/rm-tools/RMG5MCPortMap.conf` will be validated
* If the user conf is invalid, an warning will be logged, and the service will continue to use the previously validated conf
* Somge global firewall rules will be applied to setup the ability to forward ports to and from G5 MC's
* Once ready, the service will start applying the rules listed in the conf file one by one for the enabled MC targets
* For disabled targets, the service will confirm no port forwarding rules for the target exists
* Once the first run through has completed, a periodic poll will begin at a default of 20 seconds
* Every poll the service will again determine which targets are suppose to be enabled, and which are supposed to be disabled
* Based on this, the correct port forwarding rules will either be applied, confirmed, or removed
* This polling method allows for port forwarding rules to remain persistent across reboots, hotplug, and network restarts
* All events will be logged to `/var/log/rackmanager/RMG5MCPortMapService.log`

### RMG5MCPortMap.conf
* `/etc/opt/dell/rm-tools/RMG5MCPortMap.conf` is the editable user conf file for RMG5MCPortMapService
* Any edits to this file require a restart of RMG5MCPortMapService in order for the new configuration rules to take effect
* By default, this conf file will contain default rules for every possible Managed G5 MC
* The file format is basically that of a list of rules to apply to specific MC targets
* Each rule either maps an incoming RackManager port to a mc port, or an outgoing mc port to a RackManager port
* Rules will only be applied by RMG5MCPortMapService if they correspond to an "enabled" target MC
* Each rule must follow this basic format:

```
Rule Format: Direction, TargetID, enable|disable, tcp|udp, MCPort, ExternalMgmtPort, RM_Interface

Direction        -- Incoming or Outgoing
TargetID         -- Target ID for MC
                    -- e.g. MC1_1
enable|disable   -- whether port forwarding rule should be applied or not
tcp|udp          -- rule applies to tcp or udp packets (outgoing rule can also have icmp value)
MCPort           -- the mc port number  (0 - 65535)
ExternalMgmtPort -- the external RM port number  (0 - 65535)
RM_Interface     -- interface to apply port forward to (list of mgmt1 and mgmt2, e.g. mgmt2,mgmt1)
                    -- if only one specified, rule will be applied to that interface
                    -- if list specified, incoming rules will be applied to all interfaces in list
                    -- if list specified, outgoing rules will be applied to only the first interface
                       in the list that is UP

NOTES:
  - You can enable/disable all non specified outgoing protocols by using "*" for tcp|udp, MCPort, and ExternalMgmtPort
    - This only works for outgoing rules, and all 3 values must be "*"
      - e.g. "Outgoing  RMC1_1  enable   *     *      *      mgmt1,mgmt2"
  - To specifically enable outgoing icmp (ping), use "icmp" for tcp|udp value, and "*" for MCPort and ExternalMgmtPort"
    - e.g. "Outgoing  MC1_1  enable   icmp  *      *      mgmt1"

########## Example MC Specific Configuration Rules for MC1_1 ##########
Incoming  MC1_1  enable   tcp   22     51010  mgmt1,mgmt2   # incoming ssh
Incoming  MC1_1  enable   tcp   23     51011  mgmt1,mgmt2   # incoming telnet
Incoming  MC1_1  enable   tcp   80     51012  mgmt1,mgmt2   # incoming http
Incoming  MC1_1  enable   udp   161    51013  mgmt1         # incoming snmp without TLS/DTLS (requests to agent)
Incoming  MC1_1  enable   tcp   443    51014  mgmt1,mgmt2   # incoming https
Incoming  MC1_1  disable  udp   623    51015  mgmt2         # incoming ipmi (RMCP/RMCP+)
Incoming  MC1_1  enable   tcp   5900   51016  mgmt1,mgmt2   # incoming virtual media
Incoming  MC1_1  enable   tcp   5901   51017  mgmt1,mgmt2   # incoming vnc
Outgoing  MC1_1  enable   *     *      *      mgmt1,mgmt2   # all non-specified outgoing protocols
Outgoing  MC1_1  enable   tcp   25     25     mgmt1,mgmt2   # outgoing SMTP
Outgoing  MC1_1  enable   tcp   53     53     mgmt1,mgmt2   # outgoing DNS (tcp)
Outgoing  MC1_1  enable   udp   53     53     mgmt1,mgmt2   # outgoing DNS (udp)
Outgoing  MC1_1  enable   udp   68     68     mgmt1,mgmt2   # outgoing DHCP
Outgoing  MC1_1  disable  udp   69     69     mgmt2         # outgoing TFTP
Outgoing  MC1_1  enable   udp   123    123    mgmt1         # outgoing NTP
Outgoing  MC1_1  enable   udp   162    162    mgmt1         # outgoing snmp trap
Outgoing  MC1_1  enable   tcp   445    445    mgmt1         # outgoing CIFS
Outgoing  MC1_1  disable  tcp   636    636    mgmt1         # outgoing LDAPS (tcp)
Outgoing  MC1_1  disable  udp   636    636    mgmt1         # outgoing LDAPS (udp)
Outgoing  MC1_1  enable   tcp   2049   2049   mgmt1         # outgoing NFS (tcp)
Outgoing  MC1_1  enable   udp   2049   2049   mgmt1         # outgoing NFS (udp)
Outgoing  MC1_1  enable   tcp   3269   3269   mgmt1         # outgoing GC
Outgoing  MC1_1  enable   icmp  *      *      mgmt1         # outgoing ping
#######################################################################
```

* By default, each MC target will start with the same default configuations:

```
# MC1_1
Incoming  MC1_1  enable   tcp   22     60010  mgmt1,mgmt2   # incoming ssh
Incoming  MC1_1  disable  tcp   23     60011  mgmt1,mgmt2   # incoming telnet
Incoming  MC1_1  enable   tcp   80     60012  mgmt1,mgmt2   # incoming http
Incoming  MC1_1  enable   udp   161    60013  mgmt1,mgmt2   # incoming snmp without TLS/DTLS (requests to agent)
Incoming  MC1_1  enable   tcp   443    60014  mgmt1,mgmt2   # incoming https
Incoming  MC1_1  disable  udp   623    60015  mgmt1,mgmt2   # incoming ipmi (RMCP/RMCP+)
Incoming  MC1_1  enable   tcp   5900   60016  mgmt1,mgmt2   # incoming virtual media
Incoming  MC1_1  enable   tcp   5901   60017  mgmt1,mgmt2   # incoming vnc
Outgoing  MC1_1  enable   *     *      *      mgmt1,mgmt2   # all non-specified outgoing protocols
```

### See also
RMg5mcPortMap - utility to enable/disable port forwarding for specific targets

### Limitations
* Functionality is tightly integrated to the DSS9000 rack architecture
* Target MC must be present in `/etc/hosts` in order for port forwarding rules to be applied
* Outgoing port forwarding rules can only be applied to one mgmt interface (mgmt1 or mgmt2) per protocol
  * For instance, you can configure DNS packets from a mc to route out either mgmt1 or mgmt2, not both
* Outgoing port forwarding requires additional configuring to the mc itself:
  * Default gateway:
    * mc must be configured to route outgoing traffic internally through the Rackmanager instead of its own external port
    * To get MC's default gateway:
      * ssh to MC
      * then `ip route` and look for `default` route
    * To set MC's default gateway:
      * ssh to MC
      * delete default route: `ip route del default`
      * add new default route: `ip route add default via 10.253.0.31 dev eth1`
  * DNS:
    * If forwarding DNS packets from a mc through and out the RackManager, you may also need to set the mc's DNS server
    * To get MC's DNS settings:
      * ssh to MC
      * `cat /etc/resolv`
    * To set MC's DNS settings:
      * first get the RM's DNS settings with `cat /etc/resolv`
      * ssh to MC
      * edit the MC's `/etc/resolv` file to mirror that of the RM's

## Usage
* ` systemctl {restart, start, stop} RMG5MCPortMapService`

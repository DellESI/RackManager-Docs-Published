Copyright &copy; 2016-2018 Dell Inc. or its subsidiaries. All rights reserved.
 
# RMMgtNetworkStart -- RackManager Namespace Networking Service V1

---

## About
The ***Rackmanager Namespace Networking Service***
is a systemd unit file that creates the necessary networking infrastructure for RackManager network communication

### Description
* Based on namespaced networking
* Supports Mgmt1/Mgmt2 interfaces

### Current Limitation:
* Not fully integrated with initscripts

## Usage
* ` systemctl {restart, start, stop} RMMgtNetworkStart`

* Provides basic management features for RackManager networking
* Remote access to ssh, http, https over the Mgmt1 and Mgmt2 interfaces
* Default network configuration uses DHCP to obtain addresses; Static option documented in ifcfg file (ifcfg-enp1s0.4021, ifcfg-enp1s0.4022)

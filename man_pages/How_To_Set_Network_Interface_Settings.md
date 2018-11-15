Copyright &copy; 2018 Dell Inc. or its subsidiaries. All rights reserved.

# How to Set Network Interface Settings
---

## About
This document explains the step-by-step process of modifying some common network interface settings for the RackManager's Mgmt1 and
Mgmt2 interfaces.

* [Setting Mgmt Interfaces to DHCP Enabled](#setting-mgmt-interfaces-to-dhcp-enabled)
  * [Permanently Setting Mgmt1 to DHCP](#permanently-setting-mgmt1-to-dhcp)
  * [Permanently Setting Mgmt2 to DHCP](#permanently-setting-mgmt2-to-dhcp)
* [Setting Mgmt Interfaces to Static](#setting-mgmt-interfaces-to-static)
  * [Permanently Setting Mgmt1 to Static](#permanently-setting-mgmt1-to-static)
  * [Permanently Setting Mgmt2 to Static](#permanently-setting-mgmt2-to-static)

## Setting Mgmt Interfaces to DHCP Enabled

By default, the mgmt1 and mgmt2 interfaces are automatically set to DHCP enabled, meaning they will request an IP address from an available
DHCP server on the connected network.

If the interface have previously been changed to static, or otherwise altered, the following steps can be used to manually set the
interfaces back to DHCP enabled.

### Permanently Setting Mgmt1 to DHCP

Note that when set to DHCP enabled, the mgmt interface must be connected to a network with a running DHCP server in order for it
to aquire an IP address.

* Modify `/opt/dell/rm-tools/RMconfig/RMMgtNetwork/RackManagerDefault-forStark/network-scripts/ifcfg-enp1s0.4021`
  * Uncomment line 6 that sets BOOTPROTO to dhcp:
    * `BOOTPROTO=dhcp`
  * Comment out lines 13, 14, and 15 regarding the static settings:
    * `#BOOTPROTO=static`
    * `#IPADDR=<static_ip_address>`
    * `#NETMASK=<netmask_value>`
  * Comment put line 16 regarding the default gateway:
    * `#GATEWAY=<default_gateway_ip_address>`
  * Comment out lines 17 and 18 regarding dns nameservers:
    * `#DNS1=<nameserver_ip_address>`
    * `#DNS2=<nameserver_ip_address>`
  * Save changes to the file
* Run RMconfig of the RMMgtNetwork subsystem
  * `RMconfig -vvv -F -s RMMgtNetwork config`
* Restart the management network to apply the changes:
  * `systemctl restart RMMgtNetworkStart`
* Once the above finishes, you can verify Mgmt1’s IP address with the following:
  * `ip netns exec nsmgmt1 ip addr show dev enp1s0.4021`

### Permanently Setting Mgmt2 to DHCP

Note that when set to DHCP enabled, the mgmt interface must be connected to a network with a running DHCP server in order for it
to aquire an IP address.

* Modify `/opt/dell/rm-tools/RMconfig/RMMgtNetwork/RackManagerDefault-forStark/network-scripts/ifcfg-enp1s0.4022`
  * Uncomment line 6 that sets BOOTPROTO to dhcp:
    * `BOOTPROTO=dhcp`
  * Comment out lines 13, 14, and 15 regarding the static settings:
    * `#BOOTPROTO=static`
    * `#IPADDR=<static_ip_address>`
    * `#NETMASK=<netmask_value>`
  * Comment put line 16 regarding the default gateway:
    * `#GATEWAY=<default_gateway_ip_address>`
  * Comment out lines 17 and 18 regarding dns nameservers:
    * `#DNS1=<nameserver_ip_address>`
    * `#DNS2=<nameserver_ip_address>`
  * Save changes to the file
* Run RMconfig of the RMMgtNetwork subsystem
  * `RMconfig -vvv -F -s RMMgtNetwork config`
* Restart the management network to apply the changes:
  * `systemctl restart RMMgtNetworkStart`
* Once the above finishes, you can verify Mgmt2’s IP address with the following:
  * `ip netns exec nsmgmt2 ip addr show dev enp1s0.4022`

## Setting Mgmt Interfaces to Static

By default, the mgmt1 and mgmt2 interfaces are automatically set to DHCP enabled, meaning they will request an IP address from an available
DHCP server on the connected network.

If you want to statically set the IP address and supporting network information (netmask, gateway, dns), please use the following steps
which will guarantee the settings will persist across reboots.

### Permanently Setting Mgmt1 to Static

* Modify `/opt/dell/rm-tools/RMconfig/RMMgtNetwork/RackManagerDefault-forStark/network-scripts/ifcfg-enp1s0.4021`
  * Comment out line 6 that sets BOOTPROTO to dhcp:
    * `#BOOTPROTO=dhcp`
  * Uncomment lines 13, 14, and 15 regarding the static settings, and modify IPADDR and NETMASK values:
    * `BOOTPROTO=static`
    * `IPADDR=<static_ip_address>`
    * `NETMASK=<netmask_value>`
  * Uncomment line 16 and modify GATEWAY value:
    * `GATEWAY=<default_gateway_ip_address>`
  * Uncomment lines 17 and 18 and set values for DNS1 and DNS2 if info available:
    * `DNS1=<nameserver_ip_address>`
    * `DNS2=<nameserver_ip_address>`
  * Save changes to the file
* Run RMconfig of the RMMgtNetwork subsystem
  * `RMconfig -vvv -F -s RMMgtNetwork config`
* Restart the management network to apply the changes:
  * `systemctl restart RMMgtNetworkStart`
* Once the above finishes, you can verify Mgmt1’s IP address with the following:
  * `ip netns exec nsmgmt1 ip addr show dev enp1s0.4021`

### Permanently Setting Mgmt2 to Static

* Modify `/opt/dell/rm-tools/RMconfig/RMMgtNetwork/RackManagerDefault-forStark/network-scripts/ifcfg-enp1s0.4022`
  * Comment out line 6 that sets BOOTPROTO to dhcp:
    * `#BOOTPROTO=dhcp`
  * Uncomment lines 13, 14, and 15 regarding the static settings, and modify IPADDR and NETMASK values:
    * `BOOTPROTO=static`
    * `IPADDR=<static_ip_address>`
    * `NETMASK=<netmask_value>`
  * Uncomment line 16 and modify GATEWAY value:
    * `GATEWAY=<default_gateway_ip_address>`
  * Uncomment lines 17 and 18 and set values for DNS1 and DNS2 if info available:
    * `DNS1=<nameserver_ip_address>`
    * `DNS2=<nameserver_ip_address>`
  * Save changes to the file
* Run RMconfig of the RMMgtNetwork subsystem
  * `RMconfig -vvv -F -s RMMgtNetwork config`
* Restart the management network to apply the changes:
  * `systemctl restart RMMgtNetworkStart`
* Once the above finishes, you can verify Mgmt2’s IP address with the following:
  * `ip netns exec nsmgmt2 ip addr show dev enp1s0.4022`

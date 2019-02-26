Copyright &copy; 2018-2019 Dell Inc. or its subsidiaries. All rights reserved.
 
# RMNamespaceMonitor -- RackManager's Network Stability Service

---

## About
The ***RMNamespaceMonitor*** is a systemd integrated service that monitors and repairs the network namespaces and the DHCP clients that serve the management ports.

If a namespace or DHCP client crashes or malfunctions, then the RMNamespaceMonitor will shut down the defective service and then restart it. The service takes 15 minutes to detect and repair both management ports. The DHCP clients will not be started if the system is configured for a static IP.

***RMNamespaceMonitor*** specific logs can be found on the RackManager: `/var/log/rackmanager/RMNamespaceMonitor.log`

### Dependencies
* RMMgtNetworkStart
  * ***RMNamespaceMonitor*** uses RMMgtNetworkStart's functionality to apply specific namespace networking changes.
* Communication with the Interconnect Module (IM)


### Description
* ***RMNamespaceMonitor*** waits for other network scripts to build the network before starting.
* Once started, the namespace and dhcp client for each management port will be polled every 5 seconds.
* At each poll, ***RMNamespaceMonitor*** will check that a namespace exists if the port is ENABLED.
  * If the port is ENABLED, but the namespace for it is DOWN, then the namespace will be set to an UP state.
  * If a namespace is UP and the port is ENABLED, then the monitor will check that a valid IP is present.
  * If no valid IP is present and the system is configured for DHCP, then the DHCP client for that namespace will be restarted.
* Once a repair has been made, the service will give the new namespace or DHCP client 5 minutes to gain an IP before rechecking.
* If errors are encountered; they are logged to RMNamespaceMonitor.log.

### Current Limitation:
* None

## Usage
* ` systemctl {restart, start, stop} RMNamespaceMonitor`

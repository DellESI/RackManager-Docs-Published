Copyright 2017 Dell, Inc. All rights reserved.

# DSS9000 RackManager and RackManager Toolkit Docs

This is a ***Public Dell ESI Github repo***  containing published documentation for the DSS9000 RackManager.


* This doc is the top-level MAN page for the DSS9000 RackManager and RackManager Toolkit.
* The ***Redfish-API-Users-Guide-for-DSS9000-Rackmanager*** contains a general description of Redfish plus details regarding the Redfish service running on DSS9000 RackManager. 
* Two sub-folders contain detail man pages referenced herein
  * ./man_pages -- contains all of the user utility command MAN pages
  * ./service_man_pages -- contains MAN pages for RMTK services managed by systemd--being updated and not published yet
* All documents are natively in Github Markdown format (except concept deck) 
  * These are easy to browse on Github natively --- it rendors Markdown fast and good
  * Or after opening the Markdown doc in Chrome, you can Print to a pdf to get a PDF version of a doc
  * you can download the markdown files and get a free markdown reader from:   `www.markdownpad.com`,   
  * you can also download the entire repo to zip or with a git clone

---

## About RackManager

The DSS9000 ***RackManager*** is an embedded CentOS server in the DSS9000 rack that provides enhanced rack-level management functions.

* The ***RackManager*** hardware platform can be implemented by: 
  * For the DSS9000, the *default* platform is an Atom Server that is embedded in the IM (Infrastructure Module) of the DSS9000 rack
    * this may also be referred to as the "Stark RackManager (SRM)" or the "SilverShadow" card
 
* ***RackManager*** interfaces with other controllers in the DSS9000 rack only via the internal rack "Management Network"
  * The DSS9000 with RackManager has an enhanced GbE internal Management Network that interconnects RackManager directly with all of the sled BMCs, as well as the other infrastructure controllers (e.g. the MCs, IMs, BCs)
  * RackManager uses the management network to:
    * communicate with internal rack infrastructure management controllers -- primarily the managed MC
    * communicate directly with node BMCs
      * This allows RackManager software to use any network-based API supported by the BMC (e.g., ipmitool, WSManagement, racadm, redfish, etc.) over the high-speed GbE internal management network
      
* ***RackManager*** has the following external interfaces 
  * Two external RJ45 Ethernet Interfaces to connect to customer networks:
    * for the DSS9000 integrated Stark RM, these ports are labeled Mgmt1 and Mgmt2 on the IM module in the first PowerBay
    * for external 1U RMs, these two ports are most likely LOM2 and LOM3 of the external server
      * LOM1 will be used to connect to the DSS9000's IM module
  * A serial console to the RM is also supported:
    * for the DSS9000 integrated Stark RM, a USB-Serial interface on the IM module can be used to connect to either the Stark RM's serial port.
    * for external 1U RMs, the serial console connects to the BMC via the serial MUX -- see the server hardware guide.

* ***RackManager*** runs the off-the-shelf CentOS 7 Minimal Operating System:
  * will pre-installed at the factory on the Stark RackManager along with the RackManager Toolkit
  * can be re-installed or updated using normal CentOS yum update facilities
  
* ***RackManager*** does not replace the DSS9000's MC, but rather provides a more flexible and higher function management infrastructure for rack-level management
  * The MC is still present in each PowerBay in order to manage the PowerBay.
  * The Managed-MC in PowerBay-1 still consolidates rack-level power and the cooling status, and provides an internal network API that RackManager uses to get the rack-level infrastructure status, or to power-on/off/reset sleds, etc.
  
## About RackManager Boot Firmware

The embedded RackManager in the IM has boot firmware that runs at power-on or reset and boots the CentOS OS.
* This is not a full ACPI-capable BIOS
* Generally no configuration is required
* Details for configuring and updating the boot firmware is contained in the `RMbiosupdate_MAN.md` man page:
  * Link: `man_pages/RMbiosupdate_MAN.md`

## About RackManager Toolkit

The ***RackManager Toolkit*** (RMTK) is a set of utilities and services written by Dell ESI specifically to run on RackManager providing enhanced management.
 
* It is installed on top of a CentOS 7 image as a yum groupinstall named "Dell RackManager Toolkit Local Repository":
  * Link: `man_pages/rackmanager-toolkit-install_MAN.md`
  
* RMTK includes several Linux Services used by the other utilities and Dell added services:
  * OpenSSH sshd -- so that customers can ssh to the RackManager
    * a customer can ssh to the RackManager through the mgmt ports, and then run other utilities from the RM's bash command-line shell
  * Apache httpd -- The RMTK's RMRedfishService is front-ended with this Apache httpd
    * Future releases will include additional web GUI APIs
  * dhcpd -- used to give IP address to the sled BMCs, utility nodes, and internal switches on the "Internal Management Network" 
    * This is only for the internal management network and therefore not visible outside the rack
    * These devices are on isolated VLANs and dhcp will not serve any device connected to the base untagged management network
      
* RMTK includes several other pre-installed open-source / Dell utilities that a customer can run from the RM command shell:
  * ipmitool -- the open-source utility for IPMI-based computer hardware management
  * wsmancli -- the open source CLI utility based on OpenWSMAN for communicating with computers that implement the WSMAN Web Service Interface 
  * racadm -- the Dell PowerEdge iDrac CLI utility
  * redfishtool -- the open-source DMTF python-based program that runs intelligent redfish commands from a CLI
    
* RMTK also provides several additional utilities developed for RackManager:
  * RMconfig -- a basic utility to setup and configure the RackManager Toolkit: 
    * includes creating default RM Users: rackmanager_adm, rackmanager_oper, rackmanager_readonly
    * includes creating default RM permission groups:  RM_ADMIN, RM_OPER, RM_READONLY
    * sets-up default RMTK config files for sshd, dhcpd, httpd, etc., and puts the path to the RMTK utilities in the standard shell path
    * creates the network stack that allows the RM to communicate with the internal management network isolated from the external Ethernet interfaces.
  * RMg5update -- a utility to update DSS9000 infrastructure firmware: MCs, IM, BC, and G5 Switches
  * RMg5cli -- a utility to connect to the DSS9000 MC
  * RMredfishtool -- a version of redfishtool CLI that is optimized for the RackManager toolkit on DSS9000
  * RMbiosupdate -- a utility to update the Stark RackManager's BIOS ROM boot firmware
  * RMadmin -- provides several helpful debug and admin subcommands
  * RMversion -- displays the RM Toolkit version
    
* RMTK includes several key "Services":
  * RMRedfishService -- a rack-level implementation of the industry standard Redfish RESTful hardware management API. The RMRedfishService runs behind the Apache httpd (as either a reverse proxy or using the Apache mod-wsgi). The service provides:
    * a rack-level Redfish service implementation from which one can manage the entire rack
    * caches that select data for speed
    * node-specific data from the sled BMCs directly over the internal management network
    * chassis, fan and power data from the managed MC over the internal management network
  * RMNodeDiscoveryService -- discovers BMC nodes and creates hosts entries with names that map to block.slot
  * RMMgmtPortMonitor -- an internal service that monitors the state of ports on the internal management network as required based on the network topology
    * The key feature is monitoring the status of the Mgmt1 and Mgmt2 external links that connect to RM via VLANS, so if the link ever drops, the RM will know to re-bringup the link
  * RMTimeService -- used to get localtime from the DSS9000 MC if RM has lost its localtime due to a poweroff.
    * When RM starts if it has lost localtime, it re-initializes its localtime from the MC
    * it routinely synces its RM localtim to the MC so that the MC localtime if valid
    * Note that timezine is not synced, but timezone on the MC is not generally visable to  a user
  * RMMgtNetworkStart -- is a script used to startup the namespaced network stacks on the RM.  It is not a full service.
    * this subsystem is called by systemd as part of normal network start so that the namespaced network stacks on RM used to implement the vlan tunnel from RM to the physical Mgmt1 and Mgmt2 ports on the DSS9000 IM switch is configured correctly.

# RackManager Toolkit MAN Pages
For additional details on the RackManager Toolkit (RMTK) utilities and services see the following detailed MAN Pages:

* Pre-installed Open-Source/Dell Standard Utilities:
  * ipmitool    `-- the ipmitool man page can be found by typing 'man ipmitool'`
  * wsmancli    `-- see wsmancli man page at <link>`
  * racadm      `-- the racadm man page can be found by typing 'racadm help'`
  * redfishtool `-- see redfishtool man page at https://github.com/DMTF/Redfishtool/blob/master/README.md`
  
* RackManager Toolkit specific Utilities:
  * RMversion     `-- see /man_pages/RMversion_MAN.md`
  * RMconfig      `-- see /man_pages/RMconfig_MAN.md`
  * RMg5cli       `-- see /man_pages/RMg5cli_MAN.md`
  * RMredfishtool `-- see /man_pages/RMredfishtool.md`
  * RMg5update    `-- see /man_pages/RMg5update_MAN.md`
  * RMbiosupdate  `-- see /man_pages/RMbiosupdate_MAN.md`
  * RMadmin       `-- see /man_pages/RMadmin_MAN.md`
  
* RackManager Toolkit **Services** MAN Pages:
  * RMRedfishService  `-- see /service_man_pages/RMRedfishService_MAN.md`
  * RMNodeDiscoveryService  `-- see /service_man_pages/RMNodeDiscoveryService_MAN.md`
  * RMMgmtPortMonitor `-- see /service_man_pages/RMMgmtPortMonitor_MAN.md`
  * RMMgtNetworkStart `-- see service_man_pages/RMMgtNetworkStart_MAN.md`
  * RMTimeService  `--see /service_man_pages/RMTimeService_MAN.md`
  
# RackManager Users Guides
The following additional User Guides and API definitions for RackManager interfaces are available:
* ***Redfish-Users-Guide*** `-- see ./Redfish-Users-Guide.md`
  * this describes the Redfish Service implementation on RackManager and lists supported APIs


# RackManager Development Docs
Additional detailed development documentation for each included RackManager Toolkit utility is located in the DEV_SPECS folder of the rackmanager-docs repo.

---



Copyright &copy; 2018 Dell Inc. or its subsidiaries. All rights reserved.

# rackmanager-toolkit -- a compressed tarball of the rackmanager toolkit repository
---

## About
***rackmanager-toolkit*** is a compressed tarball of the rackmanager toolkit repository used to install various Dell Systems Management Tools

***rackmanager-toolkit*** depends on the tar utility

***rackmanager-toolkit***  depends on the 'yum' tool to install packages

Note that `yum clean all` should be executed prior to any yum commands in order to make sure all of yum's cached metadata, packages, mirrors, etc. get cleaned and updated.

## Usage

The following sections detail how to install or update the RackManager Toolkit, and also how to update the CentOS system using the provided RackManager Toolkit repository to ensure all dependencies are correct.

* [To INSTALL the RackManager Toolkit](#to-install-the-rackmanager-toolkit)
* [To UPDATE the RackManager Toolkit](#to-update-the-rackmanager-toolkit)
* [To install the RackManager Toolkit CentOS updates via a local directory](#to-install-the-rackmanager-toolkit-centos-updates-via-a-local-directory)
* [Run RMconfig and Reboot](#run-rmconfig-and-reboot)
* [To REMOVE the RackManager Toolkit](#to-remove-the-rackmanager-toolkit)


### To INSTALL the RackManager Toolkit
1. Login as a user with Administration privileges on a RackManager and uncompress the repository

   `tar -xvzf rackmanager-toolkit-repo-<version>.tar.gz -C /`

2. Clear yum's cache of metadata, packages, mirrors to make sure the new repo gets loaded

   `yum clean all`

3. Set the current date time if not already correctly set

   `date --set="Fri May 26 13:58:00 CDT 2017"`

4. ONLY for RMTK 1.1.0
   1. Mark-install the RMTK yum group
   
      `yum -y group mark-install --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository"`

   2. Synchronize the RMTK packages not owned by the RMTK group with the RMTK group so these will get updated ("kernel" must be defined directly)
   
      `yum -y group mark-packages-sync-force --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository" kernel`

6. Install the local RackManager Toolkit Repository 

   `yum -y groupinstall "Dell RackManager Toolkit Local Repository"`

> ***NOTE*** If you do not have an active internet connection, you will see "Could not resolve host: XXXXX" errors.  To avoid this, use the "disablerepo" directive to yum like so:

   `yum -y groupinstall --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository"`


### To UPDATE the RackManager Toolkit
1. Login as a user with Administration privileges on a RackManager and uncompress the repository

   `tar -xvzf rackmanager-toolkit-repo-<version>.tar.gz -C /`

2. Clear yum's cache of metadata, packages, mirrors to make sure the new repo gets loaded

   `yum clean all`

3. Set the current date time if not already correctly set
   (very important, RPMs must be earlier time than the system)

   `date --set="Fri May 26 13:58:00 CDT 2017"`

4. ONLY for RMTK 1.1.0
   1. Mark-install the RMTK yum group
   
      `yum -y group mark-install --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository"`

   2. Synchronize the RMTK packages not owned by the RMTK group with the RMTK group so these will get updated ("kernel" must be defined directly)
   
      `yum -y group mark-packages-sync-force --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository" kernel`

4. Update the local RackManager Toolkit Repository to the newer version

   `yum -y groupupdate "Dell RackManager Toolkit Local Repository"`

> ***NOTE*** If you do not have an active internet connection, you will see "Could not resolve host: XXXXX" errors.  To avoid this, use the "disablerepo" directive to yum like so:

   `yum -y groupupdate --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository"`

> ***NOTE*** Warning messages of the form "remove failed: No such file or directory" can be safely ignored. They result from the package manager noticing that new files have been added in the most recent version of the package.

### To install the RackManager Toolkit CentOS updates via a local directory
The updates package allows you to update offline machines with specific security related packages. To update the OS or other CentOS packages, see the [RackManager OS Installation and Update Guide](https://github.com/DellESI/RackManager-Docs-Published/blob/master/man_pages/RM_LinuxInstall_MAN.md).

1. Copy updates package to the RackManager

   `scp centos-updates-repo-20180405.tar.gz`

2. Login as a user with Administration privileges on the RackManager and uncompress the repository

   `tar -xvzf centos7-updates-repo-20180405.tar.gz -C /`

3. Clear yum's cache of metadata, packages, mirrors to make sure the new repo gets loaded

   `yum clean all`

4. Update available (security-focused) CentOS packages via the local repository

   `yum -y update --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" --enablerepo="local-centos7-updates"`

### Run RMconfig and Reboot

> ***NOTE*** that when updating to RMTK versions 1.1.3.4 or older and you have previously setup the mgmt1 and/or mgmt2 interfaces with static settings. In order for these static network settings to remain you must run the following 2 commands prior to running RMconfig, or you may risk losing your remote connection:

`cp /etc/sysconfig/network-scripts/ifcfg-enp1s0.4021 \
/opt/dell/rm-tools/RMconfig/RMMgtNetwork/RackManagerDefault-forStark/network-scripts/`

`cp /etc/sysconfig/network-scripts/ifcfg-enp1s0.4022 \
/opt/dell/rm-tools/RMconfig/RMMgtNetwork/RackManagerDefault-forStark/network-scripts/`


1. After either an install or an update, you should run RMconfig.

   `RMconfig -F config`

> ***NOTE*** If RMconfig does not exist, please log out and log back in.

2. After an update and running RMconfig, it is possible that network services may not be successfully restarted. It is best to reboot the system.

   `reboot`

### To REMOVE the RackManager Toolkit
1. Login as a user with Administration privileges on a RackManager
2. Remove the local rackmanager toolkit yum group

   `yum -y groupremove --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository"`

---
## Installation, Path, and Dependencies
* ***rackmanager-toolkit*** is released as a tar file
* Contents:
  * rackmanager yum repository
    * rm-tools RPMs
    * 3rd-party RPMs

* ***rackmanager yum repository*** :
   * /opt/dell/rm-toolkit-repo is where repository files will be located (repository directory)
   * /etc/yum.repos.d/dellemc-rackmanger-toolkit.repo - Configuration file name for RM related yum repositories
     * &quot;\[dellemc-rckmgrtoolkit-local\]&quot; - the definition header to the local filesystem repository

* ***rm-tools*** is released as an RPM that contains RackManager utilities
   * /opt/dell/rm-tools is the location where rm-tools RPMS data will be stored (rm utilities directory)


---
## See Also
* Rackmanager_Quickstart.md (coming soon)
  
## Limitations
* Requires Administrator privileges for all steps
* Remote repository implementation is left for the Adminstrator and would require the following:
  * define a stanza in the /etc/yum.repos.d/dellemc-rackmanager-toolkit.repo that contains a definition to point to an internal/external network server (e.g. NFS, Apache, etc) which hosts the repository files contained in the toolkit tarfile.

## Known Issues

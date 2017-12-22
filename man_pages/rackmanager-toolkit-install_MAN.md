# rackmanager-toolkit -- a compressed tarball of the rackmanager toolkit repository
---

## About
***rackmanager-toolkit*** is a compressed tarball of the rackmanager toolkit repository used to install various Dell Systems Management Tools

***rackmanager-toolkit*** depends on the tar utility

***rackmanager-toolkit***  depends on the 'yum' tool to install packages

## Usage
```
tar -xvzf rackmanager-toolkit-repo-<version>.tar.gz -C /
yum groupinstall "Dell RackManager Toolkit Local Repository"
```
Note: If you do not have an active internet connection, you will see "Could not resolve host: XXXXX" errors.  To avoid this, use the "disablerepo" and "enablerepo" directives to yum like so:
```
yum groupinstall --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository"
```

### To install the RackManager Toolkit:
* Login as a user with Administration privileges on a RackManager and uncompress the repository
```
tar -xvzf rackmanager-toolkit-repo-<version>.tar.gz -C /
```
* Set the current date time if not already correctly set
```
date --set="Fri May 26 13:58:00 CDT 2017"
```
* Install the local rackmanager toolkit repository 
```
yum groupinstall "Dell RackManager Toolkit Local Repository"
NOTE: use the yum option -y to default YES to questions.
```
Note: If you do not have an active internet connection, you will see "Could not resolve host: XXXXX" errors.  To avoid this, use the "disablerepo" directive to yum like so:
```
yum groupinstall --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository"
NOTE: use the yum option -y to default YES to questions.
```

### To UPDATE the RackManager Toolkit:
* Login as a user with Administration privileges on a RackManager and uncompress the repository
```
tar -xvzf rackmanager-toolkit-repo-<version>.tar.gz -C /
```
* Set the current date time if not already correctly set
   (very important, RPMs must be earlier time than the system)
```
date --set="Fri May 26 13:58:00 CDT 2017"
```
* Install the local rackmanager toolkit repository 
```
yum groupupdate "Dell RackManager Toolkit Local Repository"
NOTE: use the yum option -y to default YES to questions.
```
Note: If you do not have an active internet connection, you will see "Could not resolve host: XXXXX" errors.  To avoid this, use the "disablerepo" directive to yum like so:
```
yum groupupdate --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository"
NOTE: use the yum option -y to default YES to questions.
```

### Run RMconfig
After either an install or an update, you should run RMconfig.
```
RMconfig -F config
NOTE: If RMconfig does not exist, please log out and log back in
```

### Reboot
After an update & RMconfig, it is possible that network services
may not be successfully restarted. It is best to reboot the system.

### [OPTIONS]:
* None

---
## Installation, Path, and Dependencies:
* ***rackmanager-toolkit*** is released as a tar file
* Contents:
  * rackmanager yum repository
    * rm-tools RPMs
    * 3rd-party RPMs

* ***rackmanager yum repository*** :
   * /opt/dell/rm-toolkit-repo is where repository files will be located (repository directory)
   * /etc/yum.repos.d/dellemc-rackmanger-toolkit.repo - Configuration file name for RM related yum repositories
     * “[dellemc-rckmgrtoolkit-local]” – the definition header to the local filesystem repository

* ***rm-tools*** is released as an RPM that contains RackManager utilities
   * /opt/dell/rm-tools is the location where rm-tools RPMS data will be stored (rm utilities directory)

---
## Examples:

* First extract the rackmanager toolkit into the root filesystem
```
tar -xvzf rackmanager-toolkit-repo-0.4.2.tar.gz -C /
```
* Set the current date time
```
date --set="Fri May 26 13:58:00 CDT 2017"
```
* Next list packages in the local rackmanager toolkit repository 
```
  yum --enablerepo="dellemc-rckmgrtoolkit-local" list available
```

* Finally install the rackmanager toolkit repository located on the local filesystem
```
yum groupinstall "Dell RackManager Toolkit Local Repository"
```
Note: If you do not have an active internet connection, you will see "Could not resolve host: XXXXX" errors.  To avoid this, use the "disablerepo" directive to yum like so:
```
yum groupinstall --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local" "Dell RackManager Toolkit Local Repository"
```

---
## See Also:
  
## Limitations:
* Requires Administrator privileges
* Remote repository implementation is left for the Adminstrator and would require the following:
  * define a stanza in the /etc/yum.repos.d/dellemc-rackmanager-toolkit.repo that contains a definition to point to an internal/external network server (e.g. NFS, Apache, etc) which hosts the repository files contained in the toolkit tarfile 

## Known Issues:


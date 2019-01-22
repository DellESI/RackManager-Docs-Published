# DSS9000 Best Practices: RackManager Security
Dell EMC RackManager ToolKit (RMTK) provides a secure infrastructure and Systems Management Services in the DSS9000.
You can enhance your systems management capabilities using RMTK as the foundation.

## Know The Shared Security Model 
To leverage Systems Management features in RMTK, you must first familiarize yourself with RMTK shared responsibility model which requires customers to work together towards security objectives.
DSS9000 RMTK provides the infrastructure, services, tools while the customer is responsible for securing the Operating System and Network Infrastructure.

RMTK configures infrastructure components and provides utilities to enhance security.
Shared responsibility models exist for various RMTK services:
* Operating System OS Access and Security
* RMTK Management Network Infrastructure

## Manage OS-Level Access
To secure the operating system, the customer must set their "root" or superuser administrator password.
### Manage OS-Level Security
It's also recommended to configure the Operating System based on security best practices for Linux and specifically RedHat and CentOS based distributions.

## Manage RMTK Management Network Infrastructure Security
The RMTK Management Network Infrastructure allows the RackManager services to connect to system BMCs on the installed compute sleds.  The credentials used for this connection can be regenerated as needed to maintain security.
To secure the management network connection:
### For RMTK v1.0.x.x or v1.1.x.x
The RMRedfishService will resync credentials with all of the connected BMCs at service startup. To reset the credentials, edit the credentials vault to change the password and restart the RMRedfishService.

You can paste the following bash script code into a file and execute it on the target RackManager.

```bash
#!/bin/bash

# get the new password and change the vault
bmcuserpath="/etc/opt/dell/rm-tools/Credentials/bmcuser/.passwd"
echo "Enter the new password:"
read -s newpass
echo "Updating Credential vault..."
mv $bmcuserpath "$bmcuserpath-old"
echo "rackmgr_admin:$newpass" > "$bmcuserpath"
echo "Restarting RMRedfishService..."
systemctl restart RMRedfishService
echo "Waiting 60 secs for idrac passwords to update..."
sleep 60
echo "The Management Network passwords have been changed."
```
After the RMRedfishService has completed restarting, the password will be synchronized with all the sled BMCs. It may take a short while for the password to be changed on every BMC.

#### Verifying that the password has been altered on the BMC
To confirm that the BMC user password have been changed, you must login to the BMC console using the IP address of the BMC and the username 'rackmgr_admin'.  The steps below explain how to obtain the IP addresses of the BMCs that are connected to the RackManager and how to connect with ssh.
To get the list of BMC IP addresses run the RMadmin command as in this example:
```bash
root@localhost# RMadmin getnodelist --list
```
Select an IP address from the list, such as 10.253.4.44, and login to the BMC with the following command.
```bash
ssh rackmgr_admin@10.253.4.44
```
Enter the new password that you typed into the credentials vault file previously when prompted for the password. If the login is successful then the password change is complete.


### For RMTK v1.2.x.x and later
For RMTK version 1.2 and later we have added a method to the RMadmin utility to recreate the credentials vault and restart the RMRedfishService. On restart, the RMRedfishService will automatically generate a new password in the credentials vault for the BMC user, and it will propagate that password change to all connected BMCs. To run the RMadmin utility and reset the credentials:
```bash
RMadmin resetMgtNetworkPasswd
```

## Manager RMRedfishService Access
The RackManager RedfishService (RMRedfishService) has its own user accounts for authentication via the REST API.  Although these have similar names to the Operating System accounts (explained below), they are not the same accounts, and their credentials are maintained separately. These user accounts are named: 
```
root
rackmanager_adm
rackmanager_oper
rackmanager_readonly
```
To change the password for each account run the following command.  You must use the Redfish root account's password which has the same default setting as the OS root password.  If you have changed the OS root password these passwords may be different.
```bash
RMredfishtool -u root -p calvin AccountService setpassword rackmanager_adm *newpasswd*
RMredfishtool -u root -p calvin AccountService setpassword rackmanager_oper *newpasswd*
RMredfishtool -u root -p calvin AccountService setpassword rackmanager_readonly *newpasswd*
RMredfishtool -u root -p calvin AccountService setpassword root *newpasswd*
```

## Manage OS-Level Access
### Ensure to change the default root user’s password
Since the root account has full access to the system, the best practice is to ensure that it is changed from the default. 
```
[user@localhost ~]#ssh root@<your rack manager's IP address>
[root@rackmanager ~]# passwd
Changing password for user root.
New password: <type your new password here>
Retype new password: <type your new password here>
passwd: all authentication tokens updated successfully.
```
### Ensure to change other built-in user passwords
The RackManager Operating System has a number of built-in user accounts besides root that may be used for RackManager administration. The passwords for these accounts may also be changed by using the standard Linux command `passwd`, as in the example for root above.
Their names are: 
```
rackmanager_adm
rackmanager_oper
rackmanager_readonly
```



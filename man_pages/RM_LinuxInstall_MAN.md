Copyright &copy; 2016-2018 Dell Inc. or its subsidiaries. All rights reserved.

# RM_LinuxInstall -- RackManager OS Installation and Update Guide
---

## About
This document explains the step-by-step process of installing **CentOS 7.1** or **CentOS 7.5** onto the DSS 9000 RackManager.

Because the DSS 9000 RackManager does not contain any interface to connect a monitor, a serial console connection
must be established beforehand in order to configure the installation.

With a serial console connection in place, the DSS 9000 RackManager supports OS installation through USB.

## Required Operating System -- CentOS 7.1/7.5
* The RackManager Toolkit version 1.2 and newer requires the DSS 9000 RackManager hardware to be running **CentOS 7.1** or **CentOS 7.5**.

  > Note that older versions of the toolkit require **CentOS 7.1**
* Only the **minimal** software installation package of CentOS 7.1 or CentOS 7.5 should be selected and installed during installation configuration.
* In order to install the OS, you will need to download the .iso image from a mirror site. An example mirror site is listed for each version of CentOS, although others may be available and may have the same versions.
  * You can download either the DVD .iso or the Minimal .iso:
    * CentOS 7.1
      * Mirror:  http://archive.kernel.org/centos-vault/7.1.1503/isos/x86_64/
      * DVD: http://archive.kernel.org/centos-vault/7.1.1503/isos/x86_64/CentOS-7-x86_64-DVD-1503-01.iso
      * Minimal: http://archive.kernel.org/centos-vault/7.1.1503/isos/x86_64/CentOS-7-x86_64-Minimal-1503-01.iso 
    * CentOS 7.5
      * Mirror:  http://mirrors.rit.edu/centos/7/isos/x86_64/
      * DVD: http://mirrors.rit.edu/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1804.iso
      * Minimal: http://mirrors.rit.edu/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1804.iso
 
## Serial Console Connection

In order to configure installation options, you must first establish a serial console connection between the DSS 9000 RackManager's host and a separate PC. If this is the first time connecting the PC in use to the DSS 9000 RackManager,
you may need to first download and install the appropriate drivers.

### Drivers
* Download and install USB-to-UART drivers for Silicon Labs CP210x to the host PC using the following process:

1. Go to: http://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx
2. Select and download the appropriate set of drivers based on your OS version.
3. Extract the contents of the zip file.
4. Run the included installer.

### Physical Connection
* Using a standard USB Mini-B cable, connect the DSS 9000 RackManager's console port to the host computer.

> **NOTE**:
> - The DSS 9000 RackManager must be powered (+5V LED lit) in order to connect and establish a serial console connection.
> - To connect the host's serial console to the mini USB connector, the Console Select Jumper (J13) on the DSS 9000 RackManager must be in position 1-2.

* Verify the host computer can see two additional serial ports in the Device Manager.

> **NOTE**:
> - The two additional serial ports should look similar to:
>   - Silicon Labs Dual CP210x USB to UART Bridge: Enhanced COM Port (COMxx)
>   - Silicon Labs Dual CP210x USB to UART Bridge: Standard COM Port (COMxx)
> - The port marked **Standard** above is the host port. The standard port provides access to the RackManager's Linux shell.
> - The port marked **Enhanced** provides access to a micro-BMC (uBMC) interface for remotely managing/power cycling the RackManager's motherboard.

### Terminal Emulator

* Connect to the **standard** host port of the DSS 9000 RackManager using a terminal emulator (e.g. TeraTerm, Hyperterminal, PuTTY) and the following settings:

Parameter    | Value
------------ | -----
Speed        | 115,200
Data Bits    | 8
Parity       | None
Stop Bits    | 1
Flow Control | None

> **NOTE**: The preferred emulation mode is **ANSI**

## CentOS 7.1/7.5 Installation
The DSS 9000 RackManager currently only supports OS installation via USB with a bootable USB drive.

### Installing via USB
The following detailed steps walk you through the installation process via USB:

1. Create a bootable CentOS 7.1 or CentOS 7.5 USB drive using your favorite method, and insert the drive into one of the two USB ports on the DSS 9000 RackManager hardware, or in the USB port on the IM module if inside a powerbay.

  > **NOTE**: This does not work if using a USB hub, the bootable USB drive must be directly inserted
2. Confirm the DSS 9000 RackManager has power, and then connect to the serial console of DSS 9000 RackManager via your favorite serial terminal emulator.

  > **NOTE**: Follow the steps in the `Establishing a Serial Console Connection` section to establish a serial console connection.
3. Reboot/Reset the DSS 9000 RackManager by pressing the physical reboot button next to the Mini USB-B serial port with a pen, or the physical reset switch on the board itself

  > **NOTE**: If your setup is correct and functioning, you should see activity on your serial console
4. Before the existing OS begins to boot, press `F12` to go to the boot menu.
5. Press the number corresponding to the inserted USB drive. You should then be prompted with install options.
6. Move the cursor next to the `Install CentOS 7` option, and press `tab` so that you can configure the install to use the serial console. You should then be presented with the following line of text:
  * For **CentOS 7.1**
    > `vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CENTOS\x207\x20X8 rd.live.check quiet`
  * For **CentOS 7.5**
    > `vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CENTOS\x207_5 quiet`
  
7. Edit the line by adding **`console=ttyS1,115200`** to the end of the line so that it looks like so (don't forget the spaces):
  * For **CentOS 7.1**
    > `vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CENTOS\x207\x20X8 rd.live.check quiet console=ttyS1,115200`
  * For **CentOS 7.5**
    > `vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CENTOS\x207_5 quiet console=ttyS1,115200`
8. After editing the line, press `enter` to start the installation process. After some activity, you should eventually be provided with a menu of selections for installation settings. This is the base installation settings menu.
9. Language settings are English (United States) by default. You can skip selection 1 if you are okay with this selection, or you can press `1` and hit `enter` to set a different language setting.
10. Press `2` and hit `enter` to configure Timezone settings, then press `1` and hit `enter` to set timezone. You should then be prompted to select your timezone region.
11. Select the options for your time zone region. For the US, press `11` and hit `enter`. You should then be prompted to select your specific timezone.
12. Select your specific timezone. For the US Central Standard Time, press `3` and hit `enter`. After this step, the base installation settings menu should then be displayed again.
13. The installation source has already been specified on local media, thus the "Installation source" option can be skipped.
14. As the minimal software installation is set by default, and meets our needs, you can also skip the "Software selection" option.

  > **NOTE** After the installation selection, the CentOS 7.1 and CentOS 7.5 install steps differ. Choose the appropriate section below.
  
#### CentOS 7.1 Only
15. Press `5` and hit `enter` to configure network settings.
16. Press `1` and hit `enter` to set the host name.
17. Type your desired host name (e.g. `RackManager`) and press `enter`.
18. Press `2` and hit `enter` to configure device enp1s0.
19. The default values for selections 1-6 are acceptable, thus those selections can be skipped. To change the default if desired, press the corresponding selection number and hit `enter`.
20. If selection 7 is not enabled by default, press `7` and press `enter` to connect automatically after reboot.
21. If DSS 9000 RackManager is connected to a network via a network cable, you can press `8` and hit `enter` to apply this configuration in the installer, otherwise you can skip this selection.
   > **NOTE**: If DSS 9000 RackManager is not connected to a network and you try to apply configuration in installer, you will see an error message that says **Can't apply configuration, device activation failed.**
22. If you are satisfied with the network settings, press `c` and hit `enter` to continue back to the first network settings menu.
23. Press `c` and hit `enter` again to return to the base installation settings menu.
24. Press `6` and hit `enter` to configure the install destination settings.
25. Select the disk that corresponds to sda by pressing the associated number and hitting `enter`.
26. Press `c` and hit `enter` to proceed to the partitioning options.
27. `Use all space` is selected by default, which meets our needs. Press `c` and hit `enter` to continue to the partitioning scheme options.
28. `LVM` is selected by default, which also meets our needs. Press `c` and hit `enter` to continue back to the base installation settings menu.
29. `Kdump` is enabled by default which meets our needs, so selection 7 can be skipped.
30. Users can be created after the installation, so selection 8 can be skipped.
31. Press `9` and hit `enter` to provide a password for root.
32. Type the password (e.g. `password`) and press `enter`. You will then have to retype it to confirm.
  > **NOTE**: Skip to section "Both Centos 7.1 and Centos 7.5".

#### CentOS 7.5 Only
15. Press `5` and hit `enter` to configure the install destination settings.
16. Select the disk that corresponds to the SSD or HDD by pressing the associated number and hitting `enter`.

  > **NOTE**: Be careful to select the option for the SSD or HDD, and not the USB being used to install the OS
17. Press `c` and hit `enter` to proceed to the partitioning options.
18. `Use all space` is selected by default, which meets our needs. Press `c` and hit `enter` to continue to the partitioning scheme options.
19. `LVM` is selected by default, which also meets our needs. Press `c` and hit `enter` to continue back to the base installation settings menu.
20. `Kdump` is enabled by default which meets our needs, so selection 6 can be skipped.
21. Press `7` and hit `enter` to configure network settings.
22. Press `1` and hit `enter` to set the host name.
23. Type your desired host name (e.g. `RackManager`) and press `enter`.
24. If you are satisfied with the network settings, press `c` and hit `enter` to continue back to the first network settings menu.
25. Press `c` and hit `enter` again to return to the base installation settings menu.
26. Press `8` and hit `enter` to provide a password for root.
27. Type the password (e.g. `password`) and press `enter`. You will then have to retype it to confirm.

  > **NOTE**: If the password is weak, you will see a warning, and then it will ask you if you want to proceed with the password anyway. Type `yes` and hit `enter` to proceed, or type `no` and hit `enter` to provide a stronger password.
28. Users can be created after the installation, so selection 9 can be skipped.
  > **NOTE**: If all spokes are not complete, then check for notification symbols [!]. Occasionally, the source selection option will require you to look at a second time to verify that the configuration is correct.

#### Both Centos 7.1 and CentOS 7.5
29. All the installation settings have now been set, and installation is ready to begin. Press `b` and hit `enter` to begin the installation.
30. Wait for the installation to complete. This will take approximately 10 minutes, but could take longer.
31. Once the installation is complete, you can hit `enter` to quit, and the system will reboot.
32. You should then see the prompt to login.

  > **NOTE**: You may need to press `enter` for the login prompt to display.
  
## Updating the OS

### Updating a Machine With Internet
To update your OS to the latest minor version, simply type in the command:
```yum update```

### Updating a Machine Without Internet
The simplest way to do an offline update, is to make an offline repo, as shown below. For additional methods, see https://access.redhat.com/solutions/29269.

  1. To ensure you get the most updated OS packages, first ensure that you'vs upgraded to the latest version of RMTK.
  1. Download a newer CentOS ISO Image and make a bootable USB as you would for an installation.
  1. Plug the bootable USB you made into the internet challenged machine that you want to update.
  1. Mount the USB:

        ```bash
        mount /dev/sdb1 /media
        ```

  1. Convert the USB media into an offline yum repository:

        ```bash
        cd /etc/yum.repos.d/
        touch usb.repo
        ```

  1. Edit usb.repo and enter the following:

        ```bash
        [usb]
        gpgcheck=0
        name=usb
        baseurl=file:///media/
        ```

  1. Next, clear the cache and update using only usb media:
    
        ```bash
        yum clean all
        yum update --disablerepo=\* --enablerepo="dellemc-rckmgrtoolkit-local"  --enablerepo=usb 
        ```
    
        ```bash
        cat /etc/centos-release
        ```
  1. Lastly, remove the yum repo file, so that the yum updater doesn't look for the usb repository when running in the future.
          
          ```bash
          rm /etc/yum.repos.d/usb.repo
          ```
    
Note: If RMTK requires an older dependency, then the older dependencies will block this method. To update all CentOS packages except those required by RMTK; update to the newest RMTK before updating the OS or use the --skip-broken yum option.

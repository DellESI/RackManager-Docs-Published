Copyright &copy; 2018 Dell Inc. or its subsidiaries. All rights reserved.

# How to Enable PodManager Support
---

## About
This document explains the step-by-step process of enabling, as well as disabling, RackManager's support for successful communications with an implementation of the Rack Scale Design Pod Manager when using RackManager Toolkit version 1.1.x.

Because of some of the RackManager Toolkit 1.1.x default settings, communications from a Pod Manager implementation to the RackManager will fail. Thus, if the use of an integration of the RackManager with Pod Manager is desired, one must take extra steps to enable the support.

## Enabling

To make this process easier, we have included a bash script to perform the necessary configuration changes. This script is called `setupPodManagerSupport.sh` and is located in `/opt/dell/rm-tools/RMutils/scripts/`.

Executing the script with the `-h` option will show you the functionality of the script.

```
[root@RackManager ~]# /opt/dell/rm-tools/RMutils/scripts/setupPodManagerSupport.sh -h

    Usage: setupPodManagerSupport.sh [OPTIONS]
        -V                  -- display version and exit
        -h                  -- display usage help
        -R                  -- config RM to support requests from Pod Manager
        -D                  -- undo Pod Manager specific configs implemented with -R option

```

To apply the required configuration changes and enable support of Pod Manager requests, execute the script with the -R option. This execution may take several seconds, and will ultimately restart the network to apply configuration changes. Note that because of the network restart, it is possible thay you may see a `Write failed: Broken pipe` error along with your ssh session dropping. Simply wait several seconds, and then you should be able to reconnect.

```
[root@RackManager ~]# /opt/dell/rm-tools/RMutils/scripts/setupPodManagerSupport.sh -R
```

Once complete, the RackManager should be ready to communicate with Pod Manager.

## Disabling

The same script can be used to disable, or undo the configuration changes made during enabling. To disable these, execute the script with the -D option. This execution may take several seconds, and will ultimately restart the network to apply configuration changes. Note that because of the network restart, it is possible thay you may see a `Write failed: Broken pipe` error along with your ssh session dropping. Simply wait several seconds, and then you should be able to reconnect.

```
[root@RackManager ~]# /opt/dell/rm-tools/RMutils/scripts/setupPodManagerSupport.sh -D
```

Once complete, the RackManager should be back in its default configuration state, with regards to the specific configurations changed by the enabled option. Pod Manager will no longer be able to successfully communicate with the RackManager.

Copyright &copy; 2017-2018 Dell Inc. or its subsidiaries. All rights reserved.

# RMadmin -- a python3 utility to show and set variables on the DSS9000 RackManager

###To run:
```
* Login as a user with Admin role on RackManager
* enter RMadmin [subcommand] [options]
* RMadmin defaults to pyhon3.4
* RMadmin -h to show options
```

### [Synopis]:
```  
  RMadmin -V  [v]          --print version and exit
  RMadmin -h  [v]          --print usage and subcommand list.  and exit
  RMadmin -h  <subcommand> --print list of operations for specifid subcommand, and exit
  RMadmin [OPTIONS] <subcommand> [<subcommand-args>] -- run subcommand
```

### [LIST_OPTIONS]
     by default   --formats output as a json dict

### [OPTIONS] used by subcommands:
```
 -F<filter> --Filter=<filter>  --used by getSwitchType and getDhcpLeases subcommands to filter output
 -S<state> --State=<state>     --used by getSwitchType and getDhcpLeases subcommands to filter output
 -U<unique> --Unique=<unique>  --used by getDhcpLeases. if <unique>=False duplicate leases are included
 --IfDifferent=<secs>          --use by mctimedate. Update time only if MC and RM times differ > <sec> secs
 --IfRtcReset                  --used by mctimedate. Update RM time only if RTC lost time
 -l, --list                    --formats output with one entry per line.
 -i, --ip                      --formats output with only IPaddr one entry per line.
 -v, --verbose                 --add some more verbose output to some subcommands.
 -D, --Debug                   --adds significant debug messages to output
```



### [subcommands]:
```
hello                       -- prints hello and optionally argv[1] with verbose and -h flag
getnodelist                 -- report a json object of the compute nodes in the rack
getImMac                    -- report the Mac Address of the IM
getDhcpLeases               -- report a JSON object of DHCP Lease info of nodes and switches
                                  -l a list rather than JSON
                                  -F Node (nodes only)
                                  -F Switch (switches only
getswitchlist               -- report a JSON object of switches, their IPs and types
                                  -l a list rather than JSON
                                  -F Im (IM only) Block (blocks only)
getMgmtPortStatus           -- report a JSON object of link status and enabled state of Management Ports
                                Mgmt1 or Mgmt2 - report only one port
                                  -l report as a list rather than JSON
setMgmtPortState            -- Mgmt1 (or Mgmt2)  Enabled (or Disabled)
mctimedate                  -- get:     report the MC date/time
                               set:     set the MC date/time
                               mctosys: synch the MC date/time to the RM
                               systomc: synch the RM date/time to the MC
setNodeCredentials <nodeId> -- set node credentials to value in credential vault.
mgmtnet                     -- not implemented
updateRfResourceCache       -- not implemented
clearRfResourceCaches       -- not implemented
clearRfAccountsCaches       -- not implemented
syncRfAccountsCache         -- not implemented

```


### [Notes]:
    setNodeCredentials command gets credentials from `/etc/opt/dell/rm-tools/Credentials/bmcuser/.passwd` file.
          username:password are seperated by colon(:) character 



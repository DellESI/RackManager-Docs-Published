Copyright &copy; 2016-2018 Dell Inc. or its subsidiaries. All rights reserved.

# RMredfishtool  

## About

***RMredfishtool*** is a Dell ESI customized version of the open source ***redfishtool***
that implements the client side of the Redfish RESTful API for Data Center Hardware Management.

Customizations include optional support for:
* setting the remote host IP at localhost to point to the local Rackmanager Redfish service 
* TBD: providing G5 physical location ID aliases that to more easily point to well-known G5 resources
* TBD: performance optimizations by:
  * allowing RMredfishtool to make assumptions about how the local RM Redfish service constructs URIs within collections
  * caching some static output 
* TBD: a custom AuthLocal authentication mode that uses the user's role for authorization but allows the service to skip checking password since the user has already authenticated
* TBD: providing some Dell G5-specific OEM commands eg Chassis Reseat, etc


**Redfish** is the new RESTful API for hardware management defined by the DMTF Scalable Platform Management Forum (SPMF).  It provides a modern, secure, multi-node, extendable interface for doing hardware management.  The initial release included hardware inventory, server power-on/off/reset, reading power draw, setting power limits, reading sensors such as fans, read/write of ID LEDs, asset tags, and went beyond IPMI in functionality to include inventory of processors, storage, Ethernet controllers, and total memory.
(The current 0.9.1 version of redfishtool supports these initial features)
New Redfish extensions have now been added to the spec and include firmware update, BIOS config, memory inventory, direct attached storage control, and the list grows.

***redfishtool*** makes it simple to use the Redfish API from a BASH script or interactively from a client command shell.

While other generic http clients such as Linux curl can send and receive Redfish requests, 
***redfishtool*** goes well beyond these generic http clients by automatically handling many of the hypermedia and Redfish-specific protocol aspects of the Redfish API that require a client to often execute multiple queries to a redfish service to walk the hypermedia links from the redfish root down to the detailed URI of a specific resource (eg Processor-2 of Blade-4 in a computer blade system). Specifically, redfishtool provides the following functions over curl:

* implements Redfish Session Authentication as well as HTTP Basic Auth
* walks the Redfish schema following strict interoperability processors...] to find the targeted instance based on Id, UUID, URL or other attributes
* handles GETs for collections that are returned in multiple pieces--requiring client to read in a loop until the full collection is returned
* handles ETag and If-Match headers when PATCHing a resource to write properties
* implements many common set or action operations with simple commandline syntax (eg server reset, setting LEDs, assetTag, powerLimits, etc)
* negotiates the latest redfish protocol version between client and service (demonstrating the proper way to do this)
* can read specific properties of a resource, or expand collections to include all members of the collection expanded
* supports adding and deleting users, and common Redfish account service operations
* For debug, provides multiple levels of verbose output to add descriptive headers, and show what http requests are being executed
* For debug, includes multiple levels of status display showing http status codes and headers returned and sent 
* For easy parsing, outputs all responses in JSON format unless verbose or status debug options were specified 


## Why redfishtool?


1. ***redfishtool*** was originally written during the development of the Redfish specification to help find ambiguities in the spec.  
1. ***redfishtool*** is now also being used to test inter-operability between redfish service implementations.
1. In addition, ***redfishtool*** provides an example implementation for how a client can execute common server management functions like inventory; power-on/off/reset; setting power limits, indicator LEDs, and AssetTags, and searching a multi-node redfish service to find a specific node (with specific UUID, redfish Id, etc).  redfishtool follows strict rules of interoperability.  To support this goal, liberal comments are added throughout code to explain why each step is being executed.
1. As described above, it makes it easy to use the Redfish API from a BASH script, or as an easy-to-use interactive CLI -- but WITHOUIT creating a 'new API'.   All (rather most) of the responses from ***redfishtool*** are Redfish-defined responses.  The properties and resources are defined in the redfish spec.   ***redfishtool*** is just a tool to access the Redfish API-not a new interface itself.
     * The exception is that a 'list' operation was added for all collections to display the key properties for each of the members--rather than just the URIs to the members.
 


## Usage

***RMredfishtool***  [ ***Options*** ] [ ***SubCommands*** ] [ ***Operation*** ] [ ***OtherArgs*** ]

 * ***RMredfishtool*** is a python3.4+ program.  It uses the python3 "requests" lib for sending http requests, and a host of other standard libs in python3.4+
 * The ***RMredfishtool*** option/optarg parsing strictly follows the well established linux/GNU getopt syntax where arguments and options can be specified in any order, and both short (eg -r <host>) or long (--rhost=<host>) syntax is supported.
 * ***options*** are used to pass usernames, passwords, Host:port, authentication options, verbose/status flags, and also to specify how to search to find specific collection members (-I <Id>, -a (all), -M <prop>:<val> ). 
 * ***subCommands*** indicate the general area of the API (following ipmitool convention), and align with Redfish navigation property names like "Chassis", "Systems", "AccountService", etc.
 * ***Operations*** are specify an action or operation you want to perform like S`ystems setBootOverride` ..., or `Systems reset`.
 * ***OtherArgs*** are any other arguments after the Operation that are sometimes required--like:  `Systems <setBootOverride> <enableValue>` <targetValue>"


### Common OPTIONS:
       -V,  --version                   -- show RMredfishtool version, and exit
       -h,  --help                      -- show Usage, Options, and list of subCommands, and exit
       -u <user>,   --user=<usernm>     -- username used for remote redfish authentication
       -p <psswd>, --password=<psswd>   -- password used for remote redfish authentication
       -r <rhost>,  --rhost=<rhost>     -- remote redfish service hostname or IP:port
                                        -- by default <rhost> is localhost and thus routed to the RM Redfish service
       -t <token>,  --token=<token>     -- redfish auth session token-for sessions across multiple calls
       -q,  --quiet                     -- quiet mode--suppress error, warning, and diagnostic messages
       -c <cfgFile>,--config=<cfgFile>  -- read options (including credentials) from file <cfgFile>
       -T <timeout>,--Timeout=<timeout> -- timeout in seconds for each http request.  Default=10
       -P <property>, --Prop=<property> -- return only the specified property. Applies only to all "get" operations
       -v,  --verbose                   -- verbose level, can repeat up to 5 times for more verbose output
                                           -v(header), -vv(+addl info), -vvv(Request trace), -vvvv(+subCmd dbg), 
                                           -vvvvv(max dbg)
                                           *** use -vvv to see all of the requests being sent for a command
       -s,  --status                    -- status level, can repeat up to 5 times for more status output
                                          -s(http_status), 
                                          -ss(+r.url, +r.elapsed executionTime ),
                                          -sss(+requestHdrs,data,authType, +respStatus_code, +elapsed exec time, 
                                               AuthToken/sessId/sessUri)
                                          -ssss(+response headers for debug), -sssss(+response data for debug)
###### Options used by "raw" subcommand:
       -d <data>   --data=<data>        -- the http request "data" to send on PATCH,POST,or PUT requests

###### Options to specify top-level collection members: eg: `Systems - I <sysId>`
       -I <Id>, --Id=<Id>         -- Use <Id> to specify the collection member
       -M <prop>:<val> --Match=<prop>:<val> -- Use <prop>=<val> search to find the collection member
       -F,  --First               -- Use the 1st link returned in the collection or 1st "matching" link if used with -M
       -1,  --One                 -- Use the single link returned in the collection. Return error if more than one member
       -a,  --all                 -- Returns all members if the operation is a Get on a top-level collection like Systems
       -L <Link>,  --Link=<Link>  -- Use <Link> (eg /redfish/v1/Systems/1) to reference the collection member.
                                  --   If <Link> is not one of the links in the collection, and error is returned.
###### Options to specify 2nd-level collection members: eg: `Systems -I<sysId> Processors -i<procId>`
       -i <id>, --id=<id>         -- use <id> to specify the 2nd-level collection member
       -m <prop>:<val> --match=<prop>:val> -- use <prop>=<val> search of 2nd-level collection to specify member
       -l <link>  --link=<link>   -- Use <link> (eg /redfish/v1/Systems/1/Processors/1) to reference a 2nd level resource
                                  -- a -I|M|F|1|L option is still required to specify the link to the top-lvl collection
       -a,  --all                 -- Returns all members of the 2nd level collection if the operation is a Get on the
                                  --  2nd level collection (eg Processors). 
                                  --  -I|M|F|1|L still specifies the top-lvl collection.

###### Additional OPTIONS:
       -W <num>:<connTimeout>,           -- Send up to <num> {GET /redfish} requests with <connTimeout> TCP connection
           --Wait=<num>:<ConnTimeout>    --   timeouts before sending subcommand to rhost.  Default is -W 1:3
       -A <Authn>,   --Auth <Authn>      -- Authentication type to use:  Authn={None|Basic|Session}  Default is Basic
       -S <Secure>,  --Secure=<Secure>   -- When to use https: (Note: doesn't stop rhost from redirect http to https)
                                 <Secure>={Always | IfSendingCredentials | IfLoginOrAuthenticatedApi | Never(default) }
       -R <ver>,  --RedfishVersion=<ver> -- The Major Redfish Protocol version to use: ver={v1(default), v<n>, Latest }
       -C         --CheckRedfishVersion  -- tells RMredfishtool to execute GET /redfish to verify that the rhost supports
                                            the specified redfish protocol version before executing the sub-command.
                                            The -C flag is auto-set if the "-R Latest"  or "-W ..." options were selected.
                                            was specified by the user.
       -H <hdrs>, --Headers=<hdrs>       -- Specify the request header list--overrides defaults. Format "{ A:B, C:D...}
       -D <flag>,  --Debug=<flag>        -- Flag for dev debug. <flag> is a 32-bit uint: 0x<hex> or <dec> format
    
### Subcommands:
     about                    -- display version and other information about this version of RMredfishtool
     versions                 -- get redfishProtocol versions supported by rhost: GET ^/redfish
     root   |  serviceRoot    -- get serviceRoot resouce: GET ^/redfish/v1/
     Systems                  -- operations on Computer Systems in the /Systems collection
     Chassis                  -- operations on Chassis in the /Chassis collection
     Managers                 -- operations on Managers in the /Managers collection
     AccountService           -- operations on AccountService including user administration
     SessionService           -- operations on SessionService including Session login/logout
     odata                    -- get the Odata Service Document: GET ^/redfish/v1/odata
     metadata                 -- get the CSDL metadata Document: GET ^/redfish/v1/$metadata
     raw                      -- execute raw redfish http methods and URIs (-C option will be ignored)
     hello                    -- RMredfishtool hello world subcommand for dev testing
   For Subcommand usage, including subcommand Operations and OtherArgs, execute:
   
     RMredfishtool <SubCommand> -h  -- prints usage and options for the specific subCommand


### Subcommand Operations and Addl Args

###### Systems Operations
    RMredfishtool -r <rhost> Systems -h
    Usage:
     RMredfishtool [OPTNS]  Systems  <operation> [<args>]  -- perform <operation> on the system specified 
    <operations>:
      [collection]              -- get the main Systems collection. (Dflt operation if no member specified)
      [get]                     -- get the computerSystem object. (Default operation if collection member specified)
      list                      -- list information about the Systems collection members("Id", URI, and AssetTag)
      patch {A: B,C: D,...}     -- patch the json-formatted {prop: value...} data to the object
      reset <resetType>         -- reset a system.  <resetType>= On,  GracefulShutdown, GracefulRestart,
                                   ForceRestart, ForceOff, ForceOn, Nmi, PushPowerPutton
      setAssetTag <assetTag>    -- set the system's asset tag
      setIndicatorLed  <state>  -- set the indicator LED.  <state>=redfish defined values: Off, Lit, Blinking
      setBootOverride <enabledVal> <targetVal> -- set Boot Override properties. <enabledVal>=Disabled|Once|Continuous
                               -- <targetVal> =None|Pxe|Floppy|Cd|Usb|Hdd|BiosSetup|Utilities|Diags|UefiTarget|
      Processors [list]         -- get the "Processors" collection, or list "id" and URI of members.
       Processors [IDOPTN]        --  get the  member specified by IDOPTN: -i<id>, -m<prop>:<val>, -l<link>, -a #all

      EthernetInterfaces [list] -- get the "EthernetInterfaces" collection, or list "id" and URI of members.
       EthernetInterfaces [IDOPTN]--  get the member specified by IDOPTN: -i<id>, -m<prop>:<val>, -l<link>, -a #all

      SimpleStorage [list]      -- get the ComputerSystem "SimpleStorage" collection, or list "id" and URI of members.
       SimpleStorage [IDOPTN]     --  get the member specified by IDOPTN: -i<id>, -m<prop>:<val>, -l<link>, -a #all

      Logs [list]               -- get the ComputerSystem "LogServices" collection , or list "id" and URI of members.
       Logs [IDOPTN]              --  get the member specified by IDOPTN: -i<id>, -m<prop>:<val>, -l<link>, -a #all
      clearLog   <id>           -- clears the log defined by <id>
      examples                  -- example commands with syntax
      hello                     -- Systems hello -- debug command

###### Chassis Operations
     RMredfishtool -r <rhost> Chassis -h
     Usage:
        RMredfishtool [OPTNS]  Chassis  <operation> [<args>]  -- perform <operation> on the Chassis specified
     <operations>:
        [collection]                  -- get the main Chassis collection. (Dflt operation if no member specified)
        [get]                         -- get the Chassis object. (Defalut operation if collection member specified)
        list                          -- list information about the Chassis collection members("Id", URI, and AssetTag)
        patch {A: B,C: D,...}         -- patch the json-formatted {prop: value...} data to the object
        setAssetTag <assetTag>        -- set the Chassis's asset tag
        setIndicatorLed  <state>      -- set the indicator LED.  <state>=redfish defined values: Off, Lit, Blinking
        Power                         -- get the full Power resource under a specified Chassis instance.
        Thermal                       -- get the full Thermal resource under a specified Chassis instance.
    
        getPowerReading [-i<indx>] [consumed] -- get powerControl resource w/ power capacity, PowerConsumed, and power
                     power limits.  If "consumed" keyword is added, then only current usage of powerControl[indx] 
                     is returned. <indx> is the powerControl array index. default is 0.  normally, 0 is the only entry
        setPowerLimit [-i<indx>] <limit> [<exception> [<correctionTime>]]   -- set powerLimit control properties
                     <limit>=null disables power limiting. <indx> is the powerControl array indx (dflt=0)
    
        Logs [list]                   -- get the Chassis "LogServices" collection , or list "id" and URI of members.
          Logs [IDOPTN]                -- get the member specified by IDOPTN: -i<id>, -m<prop>:<val>, -l<link>, -a #all
        clearLog   <id>               -- clears the log defined by <id>
        examples                      -- example commands with syntax
        hello                         -- Chassis hello -- debug command
    
###### Managers Operations
    RMredfishtool -r <rhost> Managers -h
    Usage:
       RMredfishtool [OPTNS]  Managers  <operation> [<args>]  -- perform <operation> on the Managers specified
      <operations>:
     [collection]                 -- get the main Managers collection. (Dflt operation if no member specified)
     [get]                        -- get the specified Manager object. (Dflt operation if collection member specified)
     list                         -- list information about the Managers collection members("Id", URI, and UUID)
     patch {A: B,C: D,...}        -- patch the json-formatted {prop: value...} data to the object
     reset <resetType>            -- reset a Manager.  <resetType>= On,  GracefulShutdown, GracefulRestart,
                                      ForceRestart, ForceOff, ForceOn, Nmi, PushPowerPutton
     setDateTime <dateTimeString> --set the date and time
     setTimeOffset <offsetSTring> --set the time offset w/o changing time setting
     NetworkProtocol              -- get the "NetworkProtocol" resource under the specified manager.
     setIpAddress [-i<indx>]...   -- set the Manager IP address -NOT IMPLEMENTED YET

     EthernetInterfaces [list]    -- get the managers "EthernetInterfaces" collection, or list "id",URI, Name of members
      EthernetInterfaces [IDOPTN]   --  get the member specified by IDOPTN: -i<id>, -m<prop>:<val>, -a #all

     SerialInterfaces [list]      -- get the managers "SerialInterfaces" collection, or list "id",URI, Name of members.
      SerialInterfaces [IDOPTN]     --  get the member specified by IDOPTN: -i<id>, -m<prop>:<val>, -l<link>, -a #all

     Logs [list]                  -- get the Managers "LogServices" collection , or list "id",URI, Name of members.
      Logs [IDOPTN]                 --  get the member specified by IDOPTN: -i<id>, -m<prop>:<val>, -l<link>, -a #all
     clearLog   <id>              -- clears the log defined by <id>
     examples                     -- example commands with syntax
     hello                        -- Systems hello -- debug command

###### AccountService Operations
    RMredfishtool -r <rhost> AccountService -h
    Usage:
       RMredfishtool [OPTNS]  AccountService  <operation> [<args>]  -- perform <operation> on the AccountService
    <operations>:
       [get]                     -- get the AccountService object.
       patch {A: B,C: D,...}     -- patch the AccountService w/ json-formatted {prop: value...}
       Accounts [list]           -- get the "Accounts" collection, or list "Id", username, and Url
         Accounts [IDOPTN]       --   get the member specified by IDOPTN: -i<Id>, -m<prop>:<val>, -l<link>, -a #all
       Roles [list]              -- get the "Roles" collection, or list "Id", IsPredefined, and Url
         Roles [IDOPTN]          --   get the member specified by IDOPTN: -i<Id>, -m<prop>:<val>, -l<link>, -a #all
       adduser <usernm> <passwd> [<roleId>] -- add a new user to the Accounts collection
                                    -- <roleId>:{Admin | Operator | ReadOnlyUser | <a custom roleId}, dflt=Operator
       deleteuser <usernm>       -- delete an existing user from Accouts collection
       setpassword  <usernm> <passwd>  -- set (change) the password of an existing user account
       useradmin <userName> [enable|disable|unlock|[setRoleId <roleId>]] -- enable|disable|unlock.. a user account
       examples                  -- example commands with syntax
       hello                     -- AccountService hello -- debug command


###### SessionService Operations
    RMredfishtool -r <rhost> SessionService -h
    Usage:
       RMredfishtool [OPTNS]  SessionService  <operation> [<args>]  -- perform <operation> on the SessionService
    <operations>:
       [get]                       -- get the sessionService object.
       patch {A: B,C: D,...}       -- patch the sessionService w/ json-formatted {prop: value...}
       setSessionTimeout <timeout> -- patches the SessionTimeout property w/ etag support
       Sessions [list]             -- get the "Sessions" collection, or list "Id", username, and Url
         Sessions [IDOPTN]         --   get the member specified by IDOPTN: -i<Id>, -m<prop>:<val>, -l<link>, -a #all
       login <user> <passwd>       -- sessionLogin.  post to Sessions collection to create a session
       logout <sessionId>          -- logout or delete the session identified by <SessionId>
       examples                    -- example commands with syntax
       hello                       -- Systems hello -- debug command


###### raw Operations
    RMredfishtool -r <rhost> raw -h
    Usage:
       RMredfishtool [OPTNS] raw <method> <path>
    
       RMredfishtool raw -h# for help
       RMredfishtool raw examples  #for example commands
    
      <method> is one of:  GET, PATCH, POST, DELETE, HEAD, PUT
      <path> is full URI path to a redfish resource--the full path following <ipaddr:port>, starting with forward slash /
    
       Common OPTNS:
       -u <user>,   --user=<usernm>     -- username used for remote redfish authentication
       -p <passwd>, --password=<passwd> -- password used for remote redfish authentication
       -t <token>,  --token=<token>     -- redfish auth session token-for sessions across multiple calls
    
       -r <rhost>,  --rhost=<rhost>     -- remote redfish service hostname or IP:port
       -X <method>  --request=<method>  -- the http method to use. <method>={GET,PATCH,POST,DELETE,HEAD,PUT}. Dflt=GET
       -d <data>--data=<data>           -- the http request "data" to send on PATCH,POST,or PUT requests
       -H <hdrs>, --Headers=<hdrs>      -- Specify the request header list--overrides defaults. Format "{ A:B, C:D...}"
       -S <Secure>,  --Secure=<Secure>  -- When to use https: (Note: doesn't stop rhost from redirect http to https)
       
      <operations / methods>:
        GET           -- HTTP GET method
        PATCH         -- HTTP GET method
        POST          -- HTTP GET method
        DELETE        -- HTTP GET method
        HEAD          -- HTTP GET method
        PUT           -- HTTP GET method
        
      examples-- example raw commands with syntax
      hello   -- raw hello -- debug command
    
# Example Usage

### System subcommand Examples

    $ RMredfishtool -r 127.0.0.1:5000 Systems examples
     RMredfishtool -r<ip> Systems              # shows the systems collection
     RMredfishtool -r<ip> Systems list         # lists Id, Uri, AssetTag for all systems
     RMredfishtool -r<ip> Systems -I <id>      # gets the system with Id=<d>
     RMredfishtool -r<ip> Systems -M AssetTag:12345 # gets the system with AssetTag=12345
     RMredfishtool -r<ip> Systems -L <sysUrl>  # gets the system at URI=<systemUrl
     RMredfishtool -r<ip> Systems -F           # get the First system returned (for debug)
     RMredfishtool -r<ip> Systems -1           # get the first system and verify that there is only one system
     RMredfishtool -r<ip> Systems -I <id> patch {A: B,C: D,...}   # patch the json-formatted {prop: value...} 
                                                                  data to the object
     RMredfishtool -r<ip> Systems -I <id> reset <resetType>       # reset a system.  <resetType>=the redfish-defined 
                                                               _# values: On, Off, gracefulOff...
     RMredfishtool -r<ip> Systems -I <id> setAssetTag <assetTag>  # set the system's asset tag
     RMredfishtool -r<ip> Systems -I <id> setIndicatorLed  <state>  # set the indicator LED.  
                                                                   <state>=redfish defined values: Off, Lit, Blinking
     RMredfishtool -r<ip> Systems -I <id> setBootOverride <enabledVal> <targetVal> # set Boot Override properties. 
                                                                      <enabledVal>=Disabled|Once|Continuous
     RMredfishtool -r<ip> Systems -I<Id> Processors# get the processors Collection
     RMredfishtool -r<ip> Systems -I<Id> Processors list   # lists Id, Uri, & Socket for all processors in system w/ Id=<Id>
     RMredfishtool -r<ip> Systems -I<Id> Processors -i 1   # get the processor with id=1 in system with Id=<Id>
     RMredfishtool -r<ip> Systems -L <sysUrl> Processors -m Socket:CPU_1  # get processor with property Socket=CPU_1, 
                                                                          on system at url <sysUrl>
    
### Chassis subcommand Examples
    $ RMredfishtool -r 127.0.0.1:5000 Chassis examples
     RMredfishtool -r<ip> Chassis  # shows the Chassis collection
     RMredfishtool -r<ip> Chassis list # lists Id, Uri, AssetTag for all Chassis
     RMredfishtool -r<ip> Chassis -I <id>  # gets the Chassis with Id=<d>
     RMredfishtool -r<ip> Chassis -M AssetTag:12345# gets the Chassis with AssetTag=12345
     RMredfishtool -r<ip> Chassis -L <sysUrl>  # gets the Chassis at URI=<systemUrl
     RMredfishtool -r<ip> Chassis -F   # get the First Chassis returned (for debug)
     RMredfishtool -r<ip> Chassis -1   # get the first Chassis and verify that there is only one system
     RMredfishtool -r<ip> Chassis -I <id> patch {A: B,C: D,...} # patch the json-formatted {prop: value...} data 
                                                                to the object
     RMredfishtool -r<ip> Chassis -I <id> setAssetTag <assetTag># set the system's asset tag
     RMredfishtool -r<ip> Chassis -I <id> setIndicatorLed  <state>  # set the indicator LED.  
                                                                    <state>=redfish defined values: Off, Lit, Blinking
     RMredfishtool -r<ip> Chassis -I<Id> Power # get the full chassis Power resource
     RMredfishtool -r<ip> Chassis -I<Id> Thermal   # get the full chassis Thermal resource
     RMredfishtool -r<ip> Chassis -I<Id> getPowerReading[-i<indx> [consumed]   # get chassis/Power powerControl[<indx>] 
                                    resource if optional "consumed" arg, then return only the PowerConsumedWatts prop
     RMredfishtool -r<ip> Chassis -L<Url> setPowerLimit [-i<indx>] <limit> [<exception> [<correctionTime>]] 
                                    _# set power limit

### Managers subcommand Examples
    $ RMredfishtool -r 127.0.0.1:5000 Managers examples
     RMredfishtool -r<ip>   # shows the Managers collection
     RMredfishtool -r<ip> Managers list              # lists Id, Uri, AssetTag for all Managers
     RMredfishtool -r<ip> Managers -I <id>           # gets the Manager with Id=<d>
     RMredfishtool -r<ip> Managers -M AssetTag:12345 # gets the Manager with AssetTag=12345
     RMredfishtool -r<ip> Managers -L <mgrUrl>       # gets the Manager at URI=<mgrUrl
     RMredfishtool -r<ip> Managers -F                # get the First Manager returned (for debug)
     RMredfishtool -r<ip> Managers -1                # get the first Manager and verify that there is only one Manager
     RMredfishtool -r<ip> Managers -I <id> patch {A: B,C: D,...} # patch the json-formatted {prop: value...} data 
                                                                 to the object
     RMredfishtool -r<ip> Managers -I <id> reset <resetType>     # reset a Manager.  
                                                     <resetType>=the redfish-defined values: On, Off, gracefulOff...
     RMredfishtool -r<ip> Managers -I<Id> NetworkProtocol         # get the NetworkProtocol resource under the
                                                                  specified manager
     RMredfishtool -r<ip> Managers -I<Id> EthernetInterfaces list # lists Id, Uri, and Name for all of the NICs 
                                                                  for Manager w/ Id=<Id>
     RMredfishtool -r<ip> Managers -I<Id> EthernetInterfaces -i 1 # get the NIC with id=1 in manager with Id=<Id>
     RMredfishtool -r<ip> Managers -L <Url> EthernetInterfaces -m MACAddress:AA:BB:CC:DD:EE:FF # get NIC with MAC AA:BB... 
                                                                                               for manager at url <Url>

### AccountService subcommand Examples
    $ RMredfishtool -r 127.0.0.1:5000 AccountService examples
     RMredfishtool -r<ip> AccountService                # gets the AccountService
     RMredfishtool -r<ip> AccountService patch { "AccountLockoutThreshold": 5 } ]# set failed login lockout threshold
     RMredfishtool -r<ip> AccountService Accounts       # gets Accounts collection
     RMredfishtool -r<ip> AccountService Accounts list  # list Accounts to get Id, username, url for each account
     RMredfishtool -r<ip> AccountService Accounts -mUserName:john # gets the Accounts member with username: john
     RMredfishtool -r<ip> AccountService Roles  list    # list Roles collection to get RoleId, IsPredefined, & url
                                                        for each role
     RMredfishtool -r<ip> AccountService Roles   -iAdmin          # gets the Roles member with RoleId=Admin
     RMredfishtool -r<ip> AccountService adduser john 12345 Admin # add new user (john) w/ passwd "12345" and role: Admin
     RMredfishtool -r<ip> AccountService deleteuser john          # delete user "john"s account
     RMredfishtool -r<ip> AccountService useradmin john disable   # disable user "john"s account
     RMredfishtool -r<ip> AccountService useradmin john unlock    # unlock user "john"s account


### SessionService subcommand Examples
    $ RMredfishtool -r 127.0.0.1:5000 SessionService examples
     RMredfishtool -r<ip> SessionService                             # gets the sessionService
     RMredfishtool -r<ip> SessionService setSessionTimeout <timeout> # sets the session timeout property
     RMredfishtool -r<ip> SessionService Sessions                    # gets Sessions collection
     RMredfishtool -r<ip> SessionService Sessions -l<sessUrl>        # gets the session at URI=<sessUrl
     RMredfishtool -r<ip> SessionService Sessions -i<sessId>         # gets the session with session Id <sessId>
     RMredfishtool -r<ip> SessionService patch {A: B,C: D,...}       # patch the json-formatted {prop: value...} 
                                                                     data to the sessionService object
     RMredfishtool -r<ip> SessionService login <usernm> <passwd>     # login (create session)
     RMredfishtool -r<ip> SessionService logout <sessionId>          # logout (delete session <sessId>




# Known Issues, and ToDo Enhancements

1. modifications to make PATCH commands work better with Windows cmd shell quoting 
2. support clearlog
3. add additional APIs that have been added to Redfish after 1.0---this version supports only 1.0 APIs
4. add custom role create and delete




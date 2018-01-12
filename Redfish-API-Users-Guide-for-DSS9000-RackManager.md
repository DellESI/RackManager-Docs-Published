Copyright 2017 Dell, Inc. All rights reserved.

# ***Redfish*** API Users Guide for the DSS9000 ***RackManager*** 
Users Guide version: 1.1.0  -- for RMTK releases: v1.0.x, 1.1,x

---

## Introduction
The Redfish Service running on the DSS9000 ***RackManager*** provides a multi-node DMTF compliant Redfish API interface for managing 
all of the hardware in a DSS9000 rack.
This Redfish service  is included in the "RackManager Toolkit".

---

## About the ***RackManager*** "RedDrum" Redfish Service Implementation
The ***RackManager Redfish Service*** is based on the  ***RedDrum*** Redfish Service architecture and core code:

* The ***httpd front-end*** uses standard Centos7.1 httpd to process incoming http and https requests.
  * httpd handles all multi-threaded external http processing and SSL processing
  * Virtual servers are implemented for port 80 http and 443 https
  * Each of these virtual servers implements a reverse proxy to forward all "Redfish" APIs to the separate **RedfishService front-end**
  
* The **RedfishService front-end**  is implemented as a Python34 app based on the Flask REST framework.
  * The RedfishService runs as a separate app listening on localhost port 5001 to get requests from the Apache reverse proxies
  * This app provides authentication, request header checking and response header generation, URI and method parsing, and 
  constructs the Redfish Responses from an internal data cache, and calls the RedDrum implementation-specific **Backend** to 
  update the data cache.
  * The AccountService and SessionService is fully contained in the RedDrum Front-end code.
  * The RedfishService front-end code is single threaded but designed to execute single threaded responses dispatched from the httpd blazingly fast.
    * Most front-end requests execute in 1-4 milliseconds, since nearly all data is cached in the front-end.
  
* The RedDrum **Backend** provides the implementation-specific hardware discovery, monitoring, and controls.
The Backend for RackManager is optimized for DSS9000 RackManager and includes these features:   
  * A separate monitor thread is started for each server node and each MC infrastructure in the DSS9000
  * These monitors run in the background and store results in a backend **Redis** hash cache in RAM
  * For some APIs, the RedDrum front-end code calls backend methods that update sensor data in the front-end caches from this more dynamic backend Redis cache.   This is fast in order of 1ms to update a cache.

---

## About Redfish
The Redfish systems management API was defined by the Scalable Platform Management Forum (SPMF) 
which is part of the Distributed Management Task Force (DMTF) chartered with defining systems management standards.
Redfish provides a ***modern***, ***standard***, ***simple***, ***secure*** API for hardware management:
* ***Modern***---RESTful style API for Systems Management -- because customers/users asked for a REST API
* ***Standard***---across vendors and platform types.  Defined by DMTF standards body, SPMF forum
* ***Simple***---human readable JSON data representation, standard HTTP tool-chain understood by developers, and easy to parse
using modern languages like Python and JS
* ***Secure***---based on HTTPS authentication and encryption

### Redfish Standard Key Features
* **REST Based** -- incorporated the best practices for modern REST APIs.   
* **Uses standard HTTP method, header, and URI rules** -- HTTP methods (e.g. GET, POST, DELETE, PATCH), headers, and URIs are used 
in Redfish as defined by http RFCs and therefore supported by common http client libraries and tools (like curl, python requests, etc)
* **Authentication** -- uses standard HTTP ***BasicAuth*** Authentication as well as a ***Redfish-defined Session Authentication***
* **Encryption** -- uses standard HTTPS for encryption
* **User Roles** -- Redfish defines standard user ***Roles*** (e.g. Administrator, Operator, ReadOnlyUser), which contain sets of ***Privileges*** for doing actions
(eg ConfigureUsers, ConfigureResources, etc).   Custom roles can be defined by adding roles.
* **JSON Data** -- all request and response data is JSON formatted which is human readable and easy for modern languages to parse
* **Hypermedia API** -- Redfish is a true **Hypermedia** API which means that most of the URIs to resources are not defined by the
spec, but rather read from the interface.
  * There are four APIs defined by the spec:
    * /redfish/v1 or /redfish/v1/ -- returns the Redfish Root Service. This is the main entry point for the API
    * /redfish --returns a json array of the Redfish Protocol Versions supported by the Redfish Service (eg {"v1": "/redfish/v1"})
    * /redfish/odata --returns the OData Service Document. Not used by most json clients.
    * /redfish/$metadata --returns the OData metadata document. Not used by most json clients
  * For Redfish v1.0.x, all other specific URIs used to implement the API are read in a herchial manner starting with  with the Redfish Root Service API 
  at /redfish/v1.    This approach maximizes extensibility, but clients must be sure to not short-cut walking the hypermedia 
  API structure and hard-code 
  URIs for a specific implementation.
  
  The client must enter the Redfish service through the Redfish-defined URIs (e.g. /redfish/v1). 
  Then future requests that the client makes then use URIs that are returned as data from this root service request or subsequent 
  responses from those requests. 
  Responses to a Redfish request may contain additional URI links to get additional data (such as information about the Processors 
  in a system), or may contain URI links to other related resources (such as the fans that cool the system)
  
  Other than the “Redfish-defined URIs”, all of the other URIs are constructed by the Redfish service in whatever form the service 
  wants to implement. 
  Clients that want to be interoperable with v1.0.x of the Redfish Spec should not assume anything about how those URIs are 
  constructed—they should be considered opaque to the client.
  
  **The SPMF** is considering extensions that would allow Redfish to be  Swagger-compatible and therefore URI construction rules would 
  then be specified by the specification--but the full hypermedia paths will continue to be supported for backward compatibility,
  and any ***IDs*** used to identify instances of resources would still be generated by the service.
    * Note that the RedDrum implementation tries to follow the most common URI construction rules so that minimal or no change is expected at the point that this Swagger option in introduced.
    
 
 
### DMTF Redfish Links and Resources
In addition to the Redfish Standard and schemas, the DMTF standards body has published many artifacts about the Redfish API:
* Download Specification, Schema, and Tutorials/Education decks at: `http://www.dmtf.org/redfish`
  * Click the latest “DSP0266” link for the latest release of the Redfish Specification
  * Click the latest DSP8011 link to get a zip file of all of the latest Schema Files (
  * Near the bottom of the page, Click `<Introduction to Redfish>` under `Tutorials and Education` for the Redfish Overview and 
  Introduction slide deck (a good starting point)
  
* Published Redfish Schema files:
  * ALL Redfish V1 schema versions are at: `http://redfish.dmtf.org/schemas/v1/`
  * The latest versions (including latest erratas) are at: `http://redfish.dmtf.org/schemas/`
  
* Redfish Developer Information Site: `http://redfish.dmtf.org`
  * Has many links to mockups, YouTube videos, White Papers, Presentations, Webinars, etc.

* DMTF Open Source Tools Repos at: https://github.com/DMTF
  * `https://github.com/DMTF/Redfishtool`  -- a simple Redfish CLI tool
  * `https://github.com/DMTF/Redfish-Mockup-Server` 
  * `https://github.com/DMTF/Redfish-Mockup-Creator`
  * `https://github.com/DMTF/Redfish-Service-Validator`
  * `https://github.com/DMTF/Redfish-Interface-Emulator`
  * `https://github.com/DMTF/Redfish-Service-Conformance-Check`
  * `https://github.com/DMTF/Redfish-Tools`
    * has DMTF internally used tools: csdl-to-json-converter, odata-csdl-validator, and doc-generator
    
* BrightTalk webinars: `https://www.dmtf.org/education/webinars`
  * Introduction to Redfish (25min)
  * Redfish Data Model Deep Dive (55min)
  
* Public User Group / Forum: `http://www.redfishforum.com`

* If you have feedback contact your Dell account representative or you may provide feedback directly through the DMTF feedback 
   portal: http://www.dmtf.org/standards/feedback


### Redfish Features
The initial Redfish 1.0 specification and schemas targeted common Server Management functions that are implemented by most IPMI 2.0 Baseboard Management Controllers (BMCs), but in a RESTful API. In addition several inventory functions were added that are not defined in IPMI.
These Redfish 1.0 features included:
* Retrieve IPMI class data
  * Basic Server identification and asset info, health state, Temperature sensors and Fans, power supply, power consumption and thresholds.
* Retrieve BMC Infrastructure information
  * View configure BMC network settings, Manage local BMC user accounts.
* Retrieve basic I/O infrastructure inventory information (beyond standard IPMI)
  * Basic inventory information for Processors, Memory, storage controllers, and Network Controllers (eg MAC addresses) can be returned in Redfish defined APIs
  * ***Note:*** Level-2 Processor, Memory, Storage, and NIC inventory are not in v1.0.0 of RackManager.  They are coming in v1.1
* Reboot/Power cycle servers
  * Force power off, Force restart, Graceful restart, Graceful shutdown, Reset On.
* Change boot orders/device
  * Pxe, Hdd, Cd, Floppy, BiosSetup in Disabled, One time and Continuous mode.
* Manager User Accounts and Roles
  * New user accounts can be created or existing accounts can be changed using POST/PATCH methods.
  * Custom roles with different sets of privileges can be added, modified and deleted using POST, PATCH and DELETE methods.

---

## Current RackManager Feature Limitations:
  * RackManager v1.0.x Limitations:
    * Level-2 Systems APIs for Processor, Memory, Storage, and NIC inventory are supported. They are coming in v1.1.x
    * Level-2 Manager APIs for RackManager (NetworkProtocol inventory, and RM Ethernet settings) are not supported. Coming in v1.1.
  
---

# DSS9000 RedDrum-based Redfish Service Details
The DSS9000 RackManager implements a DMTF/SPMF-compliant Redfish Service that can be used to manage all of the resources in the rack.

In this section, we describe the specific Redfish APIs that are implemented on the RackManager Redfish Service including the properties that are writable with Patches and how the Redfish service handles HTTP headers.

## Redfish Defined URIs
As noted in the introduction, the Redfish standard defines 4 URIs that clients should use to “enter” the API. The RackManager Redfish service supports all of these:
* /redfish/v1 or /redfish/v1/ -- returns the Redfish Root Service. This is the main entry point for the API.
* /redfish --returns a json array of the Redfish Protocol Versions supported by the Redfish Service
* /redfish/odata --returns the OData Service Document. Not used by most json clients.
* /redfish/$metadata --returns the OData metadata document for the service. Not used by most json clients

All other URIs reference later in this doc are created by the RackManager Redfish service and can potentially change on later revisions of the RM software. 

For instance, navigation properties in the Root Service point to:
* The key Collections: Systems, Chassis, Managers
* resources: AccountService, SessionService, TaskService
* Schema file collections: JsonSchemas, Registries

ALL of the resources under the schemas get pointed two through the root service.

As stated elsewhere, programmatic construction of URI’s in undesirable.
URIs defined by the service are opaque to the client and do not follow any rules established by Redfish standard.
The RM Redfish Service implementation does have rules (see below) for how it constructs IDs and URIs. We include these rules here to aid in understanding. However, the goal is not for clients to construct URIs using these rules.

---

## Redfish Collections
Groups of similar resources are represented as collections.
A Redfish collection is a set of resources that typically can change if resources are added or deleted (e.g. system, chassis, managers, accounts, sessions…).
* **Systems** --
A System represents the logical view of a computer system. Any subsystem accessible from the host CPU is represented in a System resource. Each System instance will have CPUs, memory and other components. Computer systems are contained as members of the Systems collection.
* **Managers** --
The Managers collection contains BMCs, Enclosure Managers or any other component managing the infrastructure. Managers handle various management services and can also have their own components (such as NICs).
* **Chassis**  --
The Chassis collection contains resources that represent the physical aspects of the infrastructure. A single Chassis resource can house sensors, fans and the like. Racks, enclosures and blades are examples of Chassis resources included in the Chassis collection
Redfish “ResourceCollections” are always a set of URIs that correspond to members of the collection.
As with all URIs, they are opaque to clients—a client should not assume the construction of the URIs in the collection—such as the “Id” does not have to be the last segment in a URI/resource.

---

## Redfish Resources
A resource is an instance of a system, chassis, user, or service.
Resources contain a set of properties that describe the resource.

## Common Redfish resource properties
This section contains a set of common properties across all Redfish resources.
* **Id** -- The Id property of a resource identifies the resource within a collection. The value of Id shall be unique across a collection.
* **Name** --  -- 
The Name property is used to convey a human readable moniker for a resource. The type of the Name property shall be string. The value of
Name is NOT required to be unique across resource instances within a collection.
* **Description** -- 
The Description property is used to convey a human readable description of the resource. The type of the Description property shall be string.
* **Status** -- 
The Status property represents the status of a resource.
The value of the status property is a common status object type as defined by this specification. By having a common representation of status, clients can depend on consistent semantics. The Status object is capable of indicating the current intended state, the state the resource has been requested to change to, the current actual state and any problem affecting the current state of the resource.
* **Links** -- 
The Links property represents the links associated with the resource, as defined by that resources schema definition. All associated reference properties defined for a resource shall be nested under the links property.
All directly (subordinate) referenced properties defined for a resource shall be in the root of the resource.
* **RelatedItem** -- 
The RelatedItem property represents links to a resource (or part of a resource) as defined by that resources schema definition. This is not intended to be a strong linking methodology like other references. Instead it is used to show a relationship between elements or sub-elements in disparate parts of the service. For example, since Fans may be in one area of the implementation and processors in another, RelatedItem can be used to inform the client that one is related to the other (in this case, the Fan is cooling the processor).
* **Actions** -- 
The Actions property contains the actions supported by a resource.
* **OEM** -- 
The OEM property is used for OEM extensions as defined in Schema Extensibility.

---

## How RedDrum ***Constructs*** URIs and IDs
In this section, we use the following semantics: `^/<path> to indicate http[s]://<IPaddress>/<path>`

First, RM Redfish service starts all of its URIs with `/redfish/v1`

Next, the major Collections are identified by adding the collection name:
`E.g. the Chassis Collection is at URI: ^/redfish/v1/Chassis/`

Then, the URI for specific instances of a collection simply add the “Id” of the collection at the end:
`^/redfish/v1/Chassis/<chasId>`

Where <chasId> is assigned by the Redfish service.
 
The RackManager RedfishService constructs collection IDs (e.g. <chasId> ) by anchoring the member to the physical address of the resource in the chassis manager’s:
`“Rack<r> [-{Block|PowerBay|Rfu} <n> [-Sled<s>]]”`

### DSS9000 Chassis collection URI and ID Construction
The different chassis IDs for the rack, blocks, sleds, or powerBays would look like:
```
     Rack<r> -- for the entire rack: ex: “Rack1”
    Rack<r>-Block<b> – for DSS9000 Blocks ex: “Rack1-Block2”
    Rack<r>-PowerBay<p> --for powerBays ex: “Rack1-PowerBay-1
    Rack<r>-Block<b>-Sled<s>” -- for sleds , ex “Rack1-Block2-Sled3”
```

So the following are example URIs of chassis in RackManager
```
^/redfish/v1/ Chassis /Rack1 where <chasId> = “Rack1”
    ^/redfish/v1/ Chassis /Rack1-PowerBay1 where <chasId> = “Rack1-PowerBay1”
    ^/redfish/v1/ Chassis /Rack1-Block1 where <chasId> = “Rack1-Block1”
    ^/redfish/v1/ Chassis /Rack1-Block1-Sled2 where <chasId> = “Rack1-Block1-Sled2”
```

### DSS9000 Systems collection URI and ID Construction
The RM Redfish Service constructs URIs and Id for members of the “Systems” collection in a similar way.
```
   The Systems collection is at the URI: `^/redfish/v1/Systems`
   Specific members of the Systems collection are at `^/redfish/v1/Systems/<sysId>`:
       Where <sysId> is assigned by the Redfish service and constructed in the form:
           “Rack<r>-Block<b>-Sled<s>-Node-<n>”
   The following is an example URI for an entry in the Systems collection
       ^/redfish/v1/Systems/Rack1-Block2-Sled4-Node1
   Note: This is actually the computer system that is in Block2, Sled4
```
 
### DSS9000  Managers collection URI and ID Construction
The RM Redfish Service constructs URIs and Id for members of the “Managers” collection in a similar way.

```
     The Systems collection is at the URI: ^/redfish/v1/Managers
     Specific members of the Systems collection are at ^/redfish/v1/Managers/<mgrId>:
         Where <mgrId> is assigned by the Redfish service and constructed in the form:
             “Rack<r>-{Block|PowerBay|Rfu} <n>-{MC1|BC|IM}” 
             “RackManager"
     The following are example URIsfor an entries in the Managers collection:
         "RackManager"
         “Rack1-PowerBay1-MC1”
         “Rack1-PowerBay1-Im”
         “Rack1-Block2-Bc”
         
     So the following are example URIs of Managers in RM:
        ^/redfish/v1/Managers/RackManager
        ^/redfish/v1/Managers/Rack1-PowerBay1-MC1        --where <mgrId> = “Rack1-PowerBay1-MC1”
        ^/redfish/v1/Managers/ Rack1-PowerBay1-Im        -- where <mgrId> = “Rack1-PowerBay1-Im”
        ^/redfish/v1/Managers/ Rack1-Block2-Bc           --where <mgrId> = “Rack1-Block2-Bc”
       Note: Similar construction semantics are used for other resources.

```

## RackManager Redfish Configuration 
Several authentication options can be configured from the RedDrum config file at: `/etc/opt/dell/rm-tools/RedDrum.conf`

***Note***  -- dont use Redfish.conf.   The new RedDrum serivce uses RedDrum.conf

In the [Auth Section], there are several properties that affect authentication.  They are listed below with defaults:
```
     [Auth Section]
     RedfishAllowAuthNone=false
     #RedfishAuthNoneRole=Admin -- for RedDrum, the AuthNone Role is always Admin and not configurable
     RedfishAllowAuthenticatedAPIsOverHttp=true
     RedfishAllowBasicAuthOverHttp=true
     RedfishAllowSessionLoginOverHttp=true
     RedfishAllowUserCredUpdateOverHttp=true
```

* **RedfishAllowAuthNone** (default false) --- this should almost always be left false
  * This property applies when a redfish command for an API call is sent ***without authentication credentials*** over ***http***.
  * If set to false then the command request will be rejected with http status code 401 Unauthorized.
  * If set to true the request is executed with Admin privileges.
* **RedfishAllowAuthenticatedAPIsOverHttp** (default true) -- this is normal operation and secure if using Session AUth
  * If set to true then the request is checked for authentication. 
    * If the request does not have valid authentication then the request is rejected with http status code 401 Unauthorized. 
    * Requests with valid authentication are further checked for sufficient privilege for the API. 
      * Request with sufficient privileges then execute successfully.
      * Requests without sufficient privileges are rejected with http status code 403 Forbidden. 
  * If set to false then any request for Authenticated APIs over http is rejected with http error code 404 Not Found.
    * this is  not redirected as the intent of disabling this capability may be for security with the desire to not indicate if the API is even valid.

* **RedfishAllowBasicAuthOverHttp**  (default true ) --- NOTE: BasicAuth sends credentials in clear.  
  * If set to true then requests that come in on http using BasicAuth are executed.  
  * If set to false, basic authentication is ***not allowed*** over ***http*** and the authenticated APIs request is not allowed:
    * if HTTPS is enabled via the httpd.conf file (normal case), then a 302 redirect status_code is sent back with Location header.
    * if HTTPS is NOT enabled, a 401 Unaughorized status_code is returned
* **RedfishAllowSessionLoginOverHttp** (default true ) --- allows REdfish Session ***Login*** over http -- which sends credentials in clear
  * This applies when request is made using session login API: `POST /redfish/v1/SessionService/Sessions  with usernamd, passwd`
  * If set to true then request is executed.
  * If set to false then the request is not allowed:
    * if HTTPS is enabled via the httpd.conf file (normal case), then a 301 (Moved Permanently) redirect status_code is returned.
    * if HTTPS is NOT enabled, a 401 Unauthorized status_code is returned 
* **RedfishAllowUserCredUpdateOverHttp** (default true ) -- allows user password updates via http (sending passwd in clear)
  * This applies when request is made to user credential update API.
  * If set to true then request is executed
  * If set to false then the request is not allwoed:
    * if HTTPS is enabled via the httpd.conf file (normal case), then a 302 redirect status_code is sent back with Location header.
    * if HTTPS is NOT enabled, a 401 Unaughorized status_code is returned

## Authentication and Authorization
Most Redfish APIs require authentication and the only APIs that do not require authentication are the ‘Redfish Defined URIs” described earlier. Access to all other APIs below the root service do require authentication.

The Redfish Specification defines two types of authentication that must be supported:
* HTTP Basic Authentication
* Redfish Session Authentication

THe RackManager Redfish service support both authentication mechanisms – and no others currently.
***However***, since the implementation is on Centos7.1 Linux, it is very easy to extend authentication to other methods.  Contact Dell Support or sales for help if you need another authentication mechanism.

* **Basic authentication:** -- 
HTTP Basic Auth is the standard BasicAuth defined by the HTTP RFC and supported by nearly all http clients.
Since it passes credentials unencrypted, it is only allowed if HTTPS is uses.
* **Redfish Session authentication:** -- 
Redfish Session Auth is defined in the Redfish specification, and the RM Redfish service strictly complies with the definition. The client first sends a POST to the Sessions collections to “login” and in response receives an Auth Token that it can use put in the http header on subsequent requests. To logout, the client sends a DELETE to the session collection entry that was created during the login. Refer to the Redfish specification for details.
The DMTF Redfishtool CLI at `github.com/DMTF/Redfishtool` supports this authentication if an implementation reference is needed.

## Roles and Privileges
#### Privileges
In Redfish, a privilege is a permission to perform an operation (e.g. Read, Write) within a defined management domain.

The RM Redfish Service implements ONLY the five Privileges defined by the Redfish Specification:
* None – used to indicate that no privilege (no authentication) is required
* Login – indicates a user can login and Read resources but cannot update properties or perform operations
* ConfigureSelf – used to allow a user to update just their password
* ConfigureUsers – usually an admin level privilege. Allows the user to add or delete users, or change their role
* ConfigureComponents – an operator privilege. Allows the user to set properties on hardware resources like setting the AssetTag or Boot source on a server, or to perform operations on the resource (like reboot the server).
The RedDrum Redfish service does not have any OEM privileges

#### Roles
In Redfish, a Role is simply a defined set of Privileges.

There are three pre-defined “Built-in Roles” with assigned privileges:
* Administrator
Assigned Privileges: Login, ConfigureManager, ConfigureUsers, ConfigureComponents, ConfigureSelf
* Operator
Assigned Privileges: Login, ConfigureComponents, ConfigureSelf
* ReadOnly
Assigned Privileges: Login, ConfigureSelf
The RM Redfish service supports all three roles built-in.
Note:
* All users are assigned exactly one role.
* Two roles with the same privileges shall behave equivalently.
* Built in roles are not modifiable.

* **Custom Roles** -- The RM Redfish service allows a user to create new “custom” roles.
  * These custom roles can then be assigned to a userId instead of one of the predefined builtin roles.
  * It is also possible to create custom roles with different sets of privileges from the built-in ones.
  * Once created, custom roles work just like the built-in roles except that for custom roles, users can not only create the custom roles, but also modify them, or even delete them.

---

# DSS9000 RackManager Supported APIs - and writable properties
The following table contains the Redfish APIs supported by the RackManager Redfish Service.
The table is organized by:
* 1) HTTP Method (GET, PATCH, POSTs, DELETEs) and has the list o
* 2) The list of Resources or Actions that can be accessed or executed for that method.

The API details include:
* Description
* The RM Redfish URI construction related to the API
* Request and Response Data
* The HTTP Status code returned if success
* The Schema reference of the related resource in form
```
<Namespace>.<namespaceVersion>.<ResourceType> for resources, and
<Namespace>.<ResourceType> for collection resources
(collections do not have a versioned namespace in Redfish)
o The Namespace relates to the schema file that defines the resource
o The Namespace version is a string that defines the version of the Namespace that the RM Resource was built from. This is an appendix to the filename (for JsonSchema files. For CSDL XML Schema Files it is defined as a specific namespace inside the NameSpace schema file.
o The ResourceType is the Type of the specific resource defined in the schema file.
```
* All schema files can be found at the DMTF Redfish SchemaFile URL at http://redfish.dmtf.org/schemas/v1/
* Regarding the RackManager  URIs in the API table below:
  * As stated earlier in this spec, Redfish is a hypermedia API and all Redfish URIs except for the four “Redfish Defined URIs” are constructed by the service and not part of the standard. Therefore clients should obtain all specific URIs by walking the hypermedia navigation links and reading the URIs from the navigation properties.
  * Therefore, while the URIs listed in the following tables are indeed the URIs that are returned and implemented by versions v1.x.x DSS9000 RackManager Redfish Service, Redfish clients should not hard code these URIs based on the spec. They are included here for reference to debug and understand the API better. It is possible that future versions of the RM Redfish service will return different URIs in some cases
  
---

## GET APIs
* 1: Get supported Redfish Protocol versions
```
URI: ^/redfish
Request data: None
Response data: Redfish Version structure: { “v1”: “/redfish/v1” }
Privilege Required: None—no authentication required
Success Status Code: 200 OK
Schema Reference: NA
```
* 2:  Get root service
```
URI: ^/redfish/v1
Request data: None
Response data: The root service resource
Privilege required: None—no authentication required
Success Status Code: 200 OK
Schema Reference: ServiceRoot.v1_0_2.ServiceRoot
```
* 3: Get odata service doc
```
URI: ^/redfish/v1/odata
Request data: None
Response data: The Odata Service doc
Privilege required: None—no authentication required
Success Status Code: 200 OK
Schema Reference: NA
```
* 4: Get Odata Metatdata document
```
URI: ^/redfish/v1/$metadata
Request data: None
Response data: The CSDL Metadata document
Privilege required: None—no authentication required
Success Status Code: 200 OK
Schema Reference: NA
```
* 5: NA
* 6: Get JsonSchema Collection
  * v1.0.0 has only a couple of representative entries---full set coming in v1.1
```
RM URI: ^/redfish/v1/JsonSchemas
Request data: None
Response data: json schema collection resource
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: JsonSchemaFileCollection.JsonSchemaFileCollection
```
* 7: Get JsonSchema Entry
  * v1.0.0 has only a couple of representative entries---full set coming in v1.1
```
RM URI: ^/redfish/v1/JsonSchemas/<id>
Request data: None
Response data: the json schemafile
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: JsonSchemaFile.v1_0_2.JsonSchemaFile
```
* 8: Get Registries Collection
```
RM URI: ^/redfish/v1/Registries
Request data: None
Response data: the registries collection resource
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: MessageRegistryFileCollection.MessageRegistryFileCollection
``` 
* 9:Description: Get Registries Entry
  * v1.0 RackManager does not respond with any extended messages.   Base Registry is included here though
```
RM URI: ^/redfish/v1/Registries/<id>
Request data: None
Response data: the registries resource specified
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: MessageRegistryFile.v1_0_2.MessageRegistryFile
```

* 10A Get Task Service --not implemented 
* 10B Get Tasks Collection --not implemented

* 11:  Get Session Service resource
```
RM URI: ^/redfish/v1/SessionService
Request data: None
Response data: the session service
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: SessionService.v1_0_0.SessionService
```
* 12:  Get Sessions Collection
```
RM URI: ^/redfish/v1/SessionService/Sessions
Request data: None
Response data: the sessions collection
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: SessionCollection.SessionCollection
```
* 13:  Get a session entry
```
RM URI: ^/redfish/v1/SessionService/Sessions/<sessId>
Request data: None
Response data: the session resource
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: Session.v1_0_0.Session
``` 
*  14: Get account service
```
RM URI: ^/redfish/v1/AccountService
Request data: None
Response data: the account service resource
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: AccountService.v1_0_0.AccountService
```
* 15: Get accounts collection
```
RM URI: ^/redfish/v1/AccountService/Accounts
Request data: None
Response data: the accounts collection resource
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: ManagerAccountCollection.ManagerAccountCollection
```
* 16: Get a specific account entry
```
RM URI: ^/redfish/v1/AccountService/Accounts/<acctId>
Request data: None
Response data: the specified account entry
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: ManagerAccount.v1_0_0.ManagerAccount
```
* 17: Get roles collection
```
RM URI: ^/redfish/v1/AccountService/Roles
Request data: None
Response data: the roles collection
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: RoleCollection.RoleCollection
```
* 18: Get a specific role entry
```
RM URI: ^/redfish/v1/AccountService/Roles/<roleId>
Request data: None
Response data: the specified role resource
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: Role.v1_0_0.Role
```
* 19: Get Chassis Collection
```
RM URI: ^/redfish/v1/Chassis
Request data: None
Response data: the chassis collection
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: ChassisCollection.ChassisCollection
```
 
*  20: Get Chassis entry
```
RM URI: ^/redfish/v1/Chassis/<chasId>
Request data: None
Response data: the specified chassis resource
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: Chassis. v1_4_0.Chassis
```
* 21: Get Thermal info (fans, temp)
```
RM URI: ^/redfish/v1/Chassis/<chasIdT>/Thermal
Request data: None
Response data: the Thermal resource for the specified chassis
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: Thermal.v1_1_0.Thermal
```
* 22: Get Power info (PSUs, pwr)
```
RM URI: ^/redfish/v1/Chassis/<chasId>/Power
Request data: None
Response data: the Power resource for the specified chassis
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: Power.v1_2_0.Power
```
* 23: Get Systems collection
```
RM URI: ^/redfish/v1/Systems
Request data: None
Response data: The Systems Collection
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: ComputerSystemCollection. ComputerSystemCollection
```
* 24: Get a system entry
```
RM URI: ^/redfish/v1/Systems/<sysId>
Request data: None
Response data: the specified systems resource
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: ComputerSystem.v1_3_0.ComputerSystem
```
* 25: Get managers collection
```
RM URI: ^/redfish/v1/Managers
Request data: None
Response data: the managers collection
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: ManagerCollection.ManagerCollection
```
* 26: Get a manager entry
```
 RM URI: ^/redfish/v1/Managers/<mgrId>
Request data: None
Response data: the specified manager resource
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: Manager.v1_1_0.Manager
```
* 27: Get Managed MC network Protocols
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Managers/RackManager/NetworkProtocol
Request data: None
Response data: the NetworkProtocol resource for the managed MC
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: ManagerNetworkProtocol.v1_2_0.ManagerNetworkProtocol
```
* 28: Get Managed MC external enet collection
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Managers/RackManager/EthernetInterfaces
Request data: None
Response data: the EthernetIInterfaces collection for the Managed MC
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: EthernetInterfaceCollection.EthernetInterfaceCollection
```
* 29: Get Managed MC external enet eth0 resource
```
RM URI: ^/redfish/v1/Managers/RackManager/EthernetInterfaces/<interfaceId>
Request data: None
Response data: the EthernetInterface resource for the managed MC
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: EthernetInterface.v1_3_0.EthernetInterface
```
* 30: Get processors collection
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Systems/<sysId>/Processors
Request data: None
Response data: The Processors collection for the specified system
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: ProcessorCollection.ProcessorCollection
```
* 31: Get processor entry
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Systems/<sysId>/Processors/<processorId>
Request data: None
Response data: The specified Processor entry for the specified system
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: Processor.v1_1_0. Processor
```
* 32: Get simple storage collection
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Systems/<sysId>/SimpleStorage
Request data: None
Response data: The SimpleStorage collection for the specified system
Privilege required: Login
 Success Status Code: 200 OK
Schema Reference: SimpleStorageCollection.SimpleStorageCollection
```
* 33: Get simpleStorage entry
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Systems/<sysId>/SimpleStorage/<storageId>
Request data: None
Response data: The specified SimpleStorage entry for the specified system
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: SimpleStorage.v1_2_0.SimpleStorage
```
* 34: Get EthernetInterface collection
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Systems/<sysId>/EthernetInterfaces
Request data: None
Response data: The EthernetInterfaces collection for the specified system
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: EthernetInterfaceCollection.EthernetInterfaceCollection
```
* 35: Get systems enet interface entry
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Systems/<sysId>/EthernetInterfaces/<ethId>
Request data: None
Response data: The specified EthernetInterfaces entry for the specified system
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: EthernetInterface.v1_3_0.EthernetInterface
```
* 36: Get Memory collection
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Systems/<sysId>/Memory
Request data: None
Response data: The Memory collection for the specified system
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: MemoryCollection.MemoryCollection
```
* 37: Get Memory entry
  * not in RackManager v1.0.0.  coming in v1.1.0
```
RM URI: ^/redfish/v1/Systems/<sysId>/Memory/<memoryId>
Request data: None
Response data: The specified Memory entry for the specified system
Privilege required: Login
Success Status Code: 200 OK
Schema Reference: Memory.v1_3_0.Memory
```

 
---
 
 ## POST APIs
 
*  201 POST: Login using Redfish Session Authentication; Retrieve AuthToken
```
RM URI: ^/redfish/v1/SessionService/Sessions --or—
        ^/redfish/v1/SessionService/Sessions/Members
Request data:   { "UserName": "<myUserName>", "Password": "<myPassword>" }
Response data:  Same response as GET ^/redfish/v1/SessionService/Sessions/<sessionId>
                and the AuthToken is returned in header:
X-Auth-Token:  <token_string>
               and Location header points to new session URI:
Location:      /redfish/v1/SessionService/Sessions/<SessId>
Privilege required:    None (no authenticated required)
Success Status Code:   201 Created
Schema Reference:      SessionCollection.SessionCollection
```
* 202: POST: Create a new user
```
RM URI:    ^/redfish/v1/AccountService/Accounts --or--
            ^/redfish/v1/AccountService/Accounts/Members
Request data:
     { "Enabled": true, //defaults to true if not included 
       "Password": "<passwd>", 
       "UserName": "<userName>", 
       "RoleId": "<roleId_string>" } 
       (<roleId_string> must be an existing role Response data)
Response data:  Same response as GET ^/redfish/v1/AccountService/Accounts/<acctId>
                And Location header points to new session URI:
Location:        /redfish/v1/AccountService/Accounts/<acctId>
Privilege required:   ConfigureUsers
Success Status Code: 201 Created
Schema Reference: ManagerCollection.ManagerCollection
```
* 203: POST: Create a new `[custom]` Role
```
RM URI: ^/redfish/v1/AccountService/Roles --or--
        ^/redfish/v1/AccountService/Roles/Members
Request data:
        { "Id": "<newRoleId>", 
          "AssignedPrivileges": [ //***one or more of the defined privileges below: 
               "Login", "ConfigureManager", "ConfigureUsers", "ConfigureComponent", "ConfigureSelf"
          ] }
Response data: Same response as: GET ^/redfish/v1/AccountService/Roles/<roleId>
Location header points to new session URI:
Location:              /redfish/v1/AccountService/Roles/<roleId>
Privilege required:    Login
Success Status Code:   201 Created
Schema Reference: 
```
 
*  204: POST: Reset chassis (power on/off…)
  * For RM, this is supported only for ChassisType: "Sled" and "Rack"
```
RM URI:    ^/redfish/v1/Chassis/<chasId>/Actions/Chassis.Reset
Request data:
Request Body data: 
     { "ResetType": "<resetTypeEnum>" } 
     where <resetTypeEnum> supported values can be read from: Chassis property: 
         "Actions": { "#Manager.Reset": { ResetType@Redfish.AllowableValues" }} 
     For RM ChassisType: "Sled", the supported <resetTypeEnums> are: 
          "On", "ForceOff", "GracefulShutdown", "ForceRestart", "GracefulRestart" 
     For RM ChassisType: "Rack", the supporte <resetTypeEnums> are: "On", "ForceOff", "GracefulSHutdown"
Response data: None
Privilege required: Login
Success Status Code: 204 No Content
Schema Reference: Chassis.v1_2_0.Chassis
```
* 205: POST: OEM Reseat of Sled Action.
For RM, this is supported only for ChassisType: "Sled"
```
RM URI: ^/redfish/v1/Chassis/<chasId>/Actions/Chassis.Reseat
Request data: None
Response data: None
Privilege required: Login
Success Status Code: 204 No Content
Schema Reference: RM OEM Action –not in Chassis schema
```

*  206: POST: reset system (on/off…)
```
RM URI: ^/redfish/v1/Systems/<sysId>/Action/ComputerSystem.Reset
Request data:
   { "ResetType": "<resetTypeEnum>" } 
   where <resetTypeEnum> supported values can be read from: ComputerSystem property: 
      "Actions": { "#Manager.Reset": { ResetType@Redfish.AllowableValues" }} 
   For RM systems, The supported <resetTypeEnums> are: 
         "On", "ForceOff", "GracefulShutdown", "ForceRestart", "GracefulRestart"
Response data: None
Privilege required: Login
Success Status Code: 204 No Content
Schema Reference: ComputerSystem.v1_3_0.ComputerSystem
```
* 207: POST: Reset the Manager; For RM, this is supported for all managers
```
RM URI: ^/redfish/v1/Managers/<mgrId>/Actions/Manager.Reset
Request data:
     { "ResetType": "<resetTypeEnum>" } 
      where <resetTypeEnum> supported values can be read from: Manager property: 
          "Actions": { "#Manager.Reset": { ResetType@Redfish.AllowableValues" }} 
      The RM supported <resetTypeEnums> are: "GracefulShutdown", "ForceRestart", "GracefulRestart"
Response data: None
Privilege required: Login
Success Status Code: 204 No Content
Schema Reference: Manager.v1_1_0.Manager
```
 
---
 
## PATCH APIs
* 101: PATCH: Set Session Timeout Property in Session Service
```
RM URI: ^/redfish/v1/SessionService
Request data:
    { "SessionTimeout": "<newTimeoutVal>" } 
    where: <newTimeoutVal> is an int64 seconds, range: 30 to 86400 86400 second max is 24 hours
Response data: None
Privilege required: ConfigureManager
Success Status Code: 204 No Content
Schema Reference: SessionService.v1_0_0. SessionService
```
* 102: PATCH: Update Account Service Properties
```
RM URI: ^/redfish/v1/AccountService
Request data:
      { "AuthFailureLoggingThreshold": <thresh_Int64>, 
      "AccountLockoutThreshold": <lockThresh_Int16>, 
      "AccountLockoutDuration": <duration_Int32>, 
      "AccountLockoutCounterResetAfter>: <resetThresh_Int32> } 
      where: all of these integers are >=0, and
       "AccountLockoutDuration" is > AccountLockoutCounterResetAfter “MinPasswordLength” its read Only.
Response data: None
Privilege required: ConfigureUsers
Success Status Code: 204 No Content
Schema Reference: AccountService.v1_0_0.AccountService
```
* 103: PATCH: Update Specific User Account Properties
```
RM URI: ^/redfish/v1/AccountService/Accounts/<acctId>
Request data:
     { "Password": <passwd>, 
     "Enabled": true/false, 
     "Locked": false, 
     "RoleId": <roleId_string> }
    where: "Locked" can only be set to false via a PATCH, and "RoleId" must be an existing RoleId in Roles Collection
Response data: None
Privilege required: ConfigureUsers, or ConfigureSelf to update self’s password
Success Status Code: 204 No Content
Schema Reference: ManagerAccount.v1_0_0.ManagerAccount
```
 
*  104: PATCH: Update a custom role definition
  * update the members of the "AssignedProvileges" array:  (one or more of the privileges shown below)
  * update a custom role definition
```
RM URI: ^/redfish/v1/AccountService/Roles/<roleId>
Request data:
  { "AssignedPrivileges": [ "Login", "ConfigureManager", "ConfigureUsers", "ConfigureSelf", "ConfigureComponents" ] }
Response data: None
Privilege required: ConfigureUsers
Success Status Code: 204 No Content
Schema Reference: Role.v1_0_0.Role
```
* 105: PATCH: Update writable chassis Properties
```
RM URI: ^/redfish/v1/Chassis/<chasId>
NOTE: 
     IndicatorLED, AssetTag are the only writable properties. 
     But they are writable only for SOME chassis types--not all: 
        AssetTag is writable only for chassis with ChassisType: "Rack" eg URI: ^ /redfish/v1/Chassis/Rack1 
        IndicatorLED is writable only for chassis w/ ChassisType: "Sled" eg URI: ^/redfish/v1/Chassis/Rack1-Block<b>-Sled<s> 
Patch Request Data: 
    Either of: { "AssetTag": "<assetTagString>", "IndicatorLED": "<indicatorLedEnum>" } 
       where <indicatorLedEnum is one of: "Lit", "Off", "Blinking"
Response data: None
Privilege required: ConfigureComponents
Success Status Code: 204 No Content
Schema Reference: Chassis.v1_4_0.Chassis
```

* 106: PATCH: Update writable System Properties
Note: for RM, AssetTag is NOT writable for ComputerSystems. IndicatorLED, and BootSourceOverride properties are writable:
```
RM URI: ^/redfish/v1/Systems/<sysId>
Request data:
To set IndicatorLED, patch request data is: { "IndicatorLED": "<indicatorLedEnum>", } 
   where <indicatorLedEnum> is OneOf: "Lit", "Blinking", or "Off" 
To set BootSourceOverride properties, request data is one/all of: 
      { "Boot": { "BootSourceOverrideEnabled": "<enableEnum>", "BootSourceOverrideTarget": "<targEnum>" } } 
      where: <enableEnum> is oneOf: "Disabled", "Once", "Continuous" 
      and where <targEnum> supported values for the system can be read from property: 
      "Boot": { "BootSourceOverrideTarget@Redfish.AllowableValues" } 
      Forcurrent DSS9000 sleds, the allowable values WILL be: "None", "Pxe", "Hdd", "BiosSetup", "Cd", "Floppy"
Response data: None
Privilege required: ConfigureComponents
Success Status Code: 204 No Content
Schema Reference: ComputerSystem.v1_3_0.ComputerSystem
```
* 107: PATCH: Update writable Manager Properties
  * For DSS9000, this applies only to the MC1 in PowerBay1 or the RackManager rack manager
  * v1.0.0 of RackManager does not support patching RackManager.   Coming in v1.1
```
RM URI: ^/redfish/v1/Managers/<mgrId>
Request data: 
   Writable properties are: "DateTime", and "DateTimeLocalOffset"
   to set the DateTime, patch request data is: (example data) { "DateTime": "2016-01-17T01:10:30+06:00" } 
   to just change the timezone offset, patch data is: (example data) { "DateTimeLocalOffset": "05:00" }
Response data: None
Privilege required: ConfigureManager
Success Status Code: 204 No Content
Schema Reference: Manager.v1_1_0.Manager
```
 
*  108: NA
  * not supported in v1.0.0 of RackManager.  Coming in v1.1

* 109: PATCH: configure the Managed MC Ethernet Interface
  * not supported in v1.0.0 of RackManager.  Coming in v1.1
```
RM URI: ^/redfish/v1/Managers/RackManager/EthernetInterfaces/eth0
Request data:
     Writable Properties are: 
     { "Ipv4Addresses":  [ { "Address": "<192.168.0.10>", 
         "SubnetMask": "<255.255.252.0>", "AddressOrigin": "<origin>", "Gateway": <129.168.0.1>" }] } 
         where:< origin> is one of: "DHCP", "Static"
Response data: None
Privilege required: ConfigureManager
Success Status Code: 204 No Content
Schema Reference: EthernetInterface.v1_2_1.EthernetInterface
```
---

 
## DELETE APIs
* 301: DELETE:  Close the session specified by sessId.
  * This is essentially how you logout of a session
```
RM URI: ^/redfish/v1/SessionService/Sessions/<sessId>
Request data: None
Response data: None
Privilege required: 
     "ConfigureUsers "to delete ANY session
      -- or --
      "Login" for user to logout/delete that user 'sessions'
Success Status Code: 204 No Content
Schema Reference: Session.v1_0_0.Session
```
* 302: DELETE: Delete an existing User
```
RM URI: ^/redfish/v1/AccountService/Accounts/<acctId>
Request data: None
Response data: None
Privilege required: ConfigureUsers
Success Status Code: 204 No Content
Schema Reference: ManagerAccount.v1_0_0.ManagerAccount
```
* 303: DELETE: Delete an existing “Custom” role
```
RM URI: ^/redfish/v1/AccountService/Roles/<roleId>
Request data: None
Response data: None
Privilege required: ConfigureUsers
Success Status Code: 204 No Content
Schema Reference: Role.v1_0_0.Role
 ```

 
# Tools
Tools to use on systems supporting the Redfish standard fall into 3 categories; generic http(s) protocol data transfer, RestAPI browser-based plugins for development and Redfish protocol enabled RESTful API clients. Some examples are below.
* Redfish Protocol REST clients
  * ***Redfishtool*** is an Open Source Python-based command-line tool that strives to go beyond generic http clients by automatically handling many of the hypermedia and Redfish-specific protocol aspects of the Redfish API. An API may require a client to execute multiple queries to a redfish service to walk the hypermedia links from the redfish root down to the detailed URI of a specific resource. And Redfishtool wraps these queries into simple command-line driven program.
    * get Redfishtool at: `github.com/DMTF/Redfishtool`
* Generic HTTP(S) protocol data transfer clients
  *  cURL is a popular command line tool for transferring data to and from a server and supports many protocols including HTTP and HTTPS. Given the simplicity of cURL, protocol support, user authentication, SSL connections (among other features) it’s a useful rudimentary tool for manipulating RESTful-based API services including those implementing the Redfish specification. Specifically, it can be used to demonstrate RM Redfish API calls for GET, POST, PATCH and DELETE.
  * PycURL is a Python interface to libcurl can be used to fetch objects identified by a URL; Similar to cURL.
* HTTP(s) capture and modify clients
  * Fiddler is a Freeware HTTP debugging proxy server application developed by Telerik. It is also quite useful in Redfish API calls for GET, POST, PATCH and DELETE. It has a very simple and easy to use GUI.
  * Wireshark is a free and open source packet analyzer. It is used for network troubleshooting, analysis, software and communications protocol development, and education and also has a powerful GUI interface.
* Browser-based client plugins for RESTful API debugging and development
  * Postman is a Google Chrome extension which you can use to easily test your Web API methods. It can also be used against 3rd party APIs. RM Redfish APIs can be tested with ease using postman.
  * RESTClient, a debugger for RESTful web services which is very similar to Postman.

# References:
* Redfish Standards
  * Schemas, Specs, Mockups, White Papers, FAQ, Educational Material & more
  * http://www.dmtf.org/standards/redfish
* Redfish Tools
  * redfishtool, simulator, mockup creator, etc
  * https://www.dmtf.org/content/new-redfish-tools-released-github
* Redfish White Paper
  * dmtf.org/sites/default/files/standards/documents/DSP2044_1.0.0.pdf
* Redfish FAQ
  * dmtf.org/sites/default/files/standards/documents/DSP2045_1.0.0.pdf
* Redfish Home Page
  * www.dmtf.org/standards/redfish
* Redfish Specification
  * https://www.dmtf.org/sites/default/files/standards/documents/DSP0266_1.0.4.pdf
* Redfish Schema
  * http://redfish.dmtf.org/schemas/
 
 
 
 
 
 

# SNMP Adapter

The __snmpAdapter__ adapter provides the ability for the ClearBlade platform or ClearBlade Edge to function as a SNMP manager. 

The adapter utilizes MQTT topics to provide the mechanism whereby the ClearBlade Platform or ClearBlade Edge can interact with a SNMP network.

# MQTT Topic Structure
The __snmpAdapter__ adapter utilizes MQTT messaging to communicate with the ClearBlade Platform. The __snmpAdapter__ adapter will subscribe to a specific topic in order to handle requests from the ClearBlade Platform/Edge to interact with SNMP agents. In addition, the adapter has the capability to start a SNMP trap server to receive SNMP traps from SNMP agents and send the SNMP trap data to the ClearBlade Platform or ClearBlade Edge. The topic structures utilized by the __snmpAdapter__ are as follows:

  * Send SNMP Request to __snmpAdapter__: {__TOPIC ROOT__}/request
  * Send SNMP Response to Clearblade: {__TOPIC ROOT__}/response
  * Send SNMP errors to Clearblade: {__TOPIC ROOT__}/error
  * Send SNMP trap to platform/edge: {__TOPIC ROOT__}/trap


## ClearBlade Platform Dependencies
The SNMP adapter was constructed to provide the ability to communicate with a _System_ defined in a ClearBlade Platform instance. Therefore, the adapter requires a _System_ to have been created within a ClearBlade Platform instance.

Once a System has been created, artifacts must be defined within the ClearBlade Platform system to allow the adapters to function properly. At a minimum: 

  * A device needs to be created in the Auth --> Devices collection. The device will represent the adapter, for authentication purposes. The _name_ and _active key_ values specified in the Auth --> Devices collection will be used by the adapter to authenticate to the ClearBlade Platform or ClearBlade Edge. 
  * An adapter configuration data collection needs to be created in the ClearBlade Platform _system_ and populated with the data appropriate to the SNMP adapter. The schema of the data collection should be as follows:


| Column Name      | Column Datatype |
| ---------------- | --------------- |
| adapter_name     | string          |
| topic_root       | string          |
| adapter_settings | string (json)   |

### adapter_settings
The adapter_settings column will need to contain a JSON object containing the following attributes:

##### shouldHandleTraps
* A boolean value indicating whether or not the adapter should start a SNMP trap server in order to process SNMP traps from devices

##### trapServerPort
* An integer denoting the port number on which the SNMP trap server should listen
* Will default to 162 if not provided

##### snmpTransport
* Transport protocol to use ("udp" or "tcp")
* Will default to _udp_ if not provided

##### snmpVersion
* The SNMP version 
* 1, 2 or 3
* Will default to 3

##### snmpCommunity
*	SNMP Community string

##### snmpAppOpts
* netsnmp has '-C APPOPTS - set various application specific behaviours'
* Not sure if _snmpAppOpts_ applies to SNMP traps 

##### snmpMsgFlags
*	SNMPV3 MsgFlags
  * describe Authentication, Privacy, and whether a report PDU must be sent
* Not sure if _snmpMsgFlags_ applies to SNMP traps

##### snmpSecurityModel
* SecurityModel is an SNMPV3 Security Model
* UserSecurityModel (=3) is the only one implemented
* Not sure if _snmpSecurityModel_ applies to SNMP traps

##### snmpSecurityParameters
* SNMPV3 Security Model parameters struct
* Not sure if _snmpSecurityParameters_ applies to SNMP traps

##### snmpContextEngineID
* SNMPV3 ContextEngineID in ScopedPDU

##### snmpContextName
* SNMPV3 ContextName in ScopedPDU

#### adapter_settings_example
{
  "shouldHandleTraps": true,
  "trapServerPort": 164,
  "snmpTransport": "udp",
  "snmpVersion": 2,
	"snmpCommunity": "public",
  "snmpAppOpts": {"c": true, "p": true},
  "snmpMsgFlags": 2,
  "snmpSecurityModel": 3,
  "snmpSecurityParameters": {},
  "snmpContextEngineID": "",
  "snmpContextName": ""
}

## Usage

### Executing the adapter

`snmpAdapter -systemKey=<SYSTEM_KEY> -systemSecret=<SYSTEM_SECRET> -platformURL=<PLATFORM_URL> -messagingURL=<MESSAGING_URL> -deviceName=<DEVICE_NAME> -password=<DEVICE_ACTIVE_KEY> -adapterConfigCollectionID=<COLLECTION_ID> -logLevel=<LOG_LEVEL>`

   __*Where*__ 

   __systemKey__
  * REQUIRED
  * The system key of the ClearBLade Platform __System__ the adapter will connect to

   __systemSecret__
  * REQUIRED
  * The system secret of the ClearBLade Platform __System__ the adapter will connect to
   
   __deviceName__
  * The device name the adapter will use to authenticate to the ClearBlade Platform
  * Requires the device to have been defined in the _Auth - Devices_ collection within the ClearBlade Platform __System__
  * OPTIONAL
  * Defaults to __snmpAdapter__
   
   __password__
  * REQUIRED
  * The active key the adapter will use to authenticate to the platform
  * Requires the device to have been defined in the _Auth - Devices_ collection within the ClearBlade Platform __System__
   
   __platformUrl__
  * The url of the ClearBlade Platform instance the adapter will connect to
  * OPTIONAL
  * Defaults to __http://localhost:9000__

   __messagingUrl__
  * The MQTT url of the ClearBlade Platform instance the adapter will connect to
  * OPTIONAL
  * Defaults to __localhost:1883__

   __adapterConfigCollectionID__
  * REQUIRED 
  * The collection ID of the data collection used to house adapter configuration data

   __logLevel__
  * The level of runtime logging the adapter should provide.
  * Available log levels:
    * fatal
    * error
    * warn
    * info
    * debug
  * OPTIONAL
  * Defaults to __info__


## Setup
---
The __snmpAdapter__ adapter is dependent upon the ClearBlade Go SDK and its dependent libraries being installed as well as the Go SNMP library (github.com/soniah/gosnmp). The __snmpAdapter__ adapter was written in Go and therefore requires Go to be installed (https://golang.org/doc/install).


### Adapter compilation
In order to compile the adapter for execution, the following steps need to be performed:

 1. Retrieve the adapter source code  
    * ```git clone git@github.com:ClearBlade/GooglePubSubAdapter.git```
 2. Navigate to the __SNMP-ADAPTER__ directory  
    * ```cd GooglePubSubAdapter```
 3. ```go get -u github.com/ClearBlade/Go-SDK```
    * This command should be executed from within your Go workspace
 4. ```go get -u github.com/soniah/gosnmp```
 5. Compile the adapter
    * ```go build -o snmpAdapter```

### Payloads

#### Supported SNMP Operations
 * SNMP GET - snmpOperation="get"
 * SNMP GETNEXT - snmpOperation="getnext"
 * SNMP GETBULK (SNMP v2 and v3) - snmpOperation="getbulk"
 * SNMP SET - snmpOperation="set"

#### SNMP Request

The attributes inclueded with a SNMP request will be dependent upon the SNMP operation being performed. At the very minimum, the following attributes must be inclued in a SNMP request:

###### snmpAddress
* The IP address of the SNMP agent the SNMP request will be sent to

###### snmpPort
* The port the SNMP agent is listening on
* Defaults to 161

###### snmpOIDs
* An array of OIDs the request will be executed against

###### snmpOperation
* The SNMP operation to execute
* One of get, getnext, getbulk, set, walk, walkall, bulkwalk, bulkwalkall

###### snmpTransport
* Transport protocol to use ("udp" or "tcp")
* Will default to _udp_ if not provided

###### snmpVersion
* The SNMP version 
* 1, 2 or 3
* Will default to 3

##### Operation specific fields

###### snmpCommunity
*	SNMP Community string
* SNMP v1 and v2

###### snmpTimeout
*	Timeout for the SNMP Query

###### snmpRetries
*	Number of retries to attempt within timeout

###### snmpExponentialTimeout
*	Boolean value indicating whether or not to double the timeout in each retry

###### snmpMaxOids
*	Maximum number of oids allowed in a _Get_

###### snmpMaxRepetitions
*	sets the GETBULK max-repetitions used by _BulkWalk_
* Will default to 50
  * This may cause issues with some devices, if so set MaxRepetitions lower

###### snmpNonRepeaters
*	sets the GETBULK max-repeaters used by _BulkWalk_
* Will default to 0 (per RFC 1905)

##### snmpAppOpts
* netsnmp has '-C APPOPTS - set various application specific behaviours'
* c: do not check returned OIDs are increasing' - use AppOpts = {"c":true} with _walk_ or _bulkwalk_

##### snmpMsgFlags
*	SNMPV3 MsgFlags
  * describe Authentication, Privacy, and whether a report PDU must be sent

##### snmpSecurityModel
* SecurityModel is an SNMPV3 Security Model
* UserSecurityModel (=3) is the only one implemented

##### snmpSecurityParameters
* SNMPV3 Security Model parameters struct

##### snmpContextEngineID
* SNMPV3 ContextEngineID in ScopedPDU

##### snmpContextName
* SNMPV3 ContextName in ScopedPDU

```
{
  snmpAddress: "192.168.1.1",
  snmpPort: 161,
  snmpOIDs: ["myoid"],
  snmpVersion: 2,
  snmpCommunity: "mycommunity"
  snmpOperation: "get"
}
```

#### SNMP Response
The response of a SNMP operation will contain the original request as well as the data returned by the agent for each OID requested. The value and ASN.1 data types of each OID will be returned in an object indexed by the OID.
```
{
  "request": {
    snmpAddress: ,
    snmpPort: 161,
    snmpOIDs: [],
    snmpVersion: 2,
    snmpCommunity: 
    snmpOperation: "get", //get, getnext, getbulk, set
    snmpGetBulkNonRepeaters:    //The first N objects can be retrieved with a simple getnext command
    snmpGetBulkMaxRepetitions:  //Attempt up to M getnext operations to retrieve the remaining objects
  },
  ".1.3.6.1.6.3.1.1.6.1.0":{
    "value": 663205133,
    "asn1berType":
  }
}
```


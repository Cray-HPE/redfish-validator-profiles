# BMC Functional and Quality Requirements


### Version
| Date  |  Author | Content  |
|---|---|---|
|  2022-02-08 | Andrew Nieuwsma  | Initial Draft  |


## Overview

This document describes the functional and quality requirements that CSM expects any Redfish based BMC (baseboard management controller) controlling nodes to support.  This document is intended to give objective measureable requirements that correspond with the [profiles](../profiles/) listed.

Some of these requirements transcend BMC requirements and relate to the interconnection between BMC and nodes. 

## Definitions:

1. Redfish - Protocol developed by [DMTF](https://www.dmtf.org/standards/redfish)
2. BMC - Baseboard Management Controller, hereafter referred to as the 'BMC', this entity provides the Redfish interfact and controls the node.
3. Node - the hardware component that is controlled by the BMC, hereafter referred to as the 'node'.
4. Customer - The Cray System Managment (CSM) services is the consumer of the redfish API hosted on BMCs.  The customer refers to CSM and its usage pertaining to the Redfish APIs on the BMC.


## Functional Requirements

|  ID  |  Requirement | Notes  |
|---|---|---|
|  FR1 | The BMC shall support the paths, parameters, and cabilities specified in the [profiles](../profiles/) listing.   Here is a restatement of some of the core support requirements: The BMC shall at a minimum support: <ol><li>Power actions of the node and the BMC</li><li>Firmware inventory/update of the node and the BMC</li><li>Redfish Alerts and Events</li><li>telemetry (either streammed (ideal) or polled) of the node</li><li>power capping of the node.</li></ol> | The Customer has specified a minimum set of Redfish API requirments needed to support its operation. |
| FR2 | The BMC shall send power alerts regardless of the underlying cause of the power event.  This means that a power alert shall be sent if a human actor, IPMI instruction, or Redfish instruction initiates the power event. | It is crucial to the Customer operations that all power events, regardless of origination are sent so the Customer can manage the system. |
| FR3 | When the BMC reports a power status in response to a request to change status (e.g. a power-on is issued), the BMC shall follow through with the request.  This means that if you respond that a power-on event is occuring, that event shall occur. In no case shall the BMC respond that it is completing a power event if it is unable to do so. | We have seen Redfish implementations that exhibit the following incorrect behavior: <ol><li>The power off is issued to all nodes under the BMC.</li><li> Once all nodes have completed the power off (determined via a poll of power status), a power on is issued to all nodes with no wait time.</li><li>If not enough wait time is given, the nodes will complete the posted power on command but not power on.</li></oi>  |
| FR4 | The BMC when managing a node shall have a means of synchronizing the BMC time to the node time via Redfish. | Many on-node operations require clock time to be set accurately, being able to synchronize via the BMC is very beneficial to the customer. |
| FR5 | Every ethernet port shall have exactly one mac-address.  In no case shall the BMC and node share the same ethernet port. Each BMC shall have a dedicated ethernet port. | Customer discovery process depends upon MAC address based discovery. Multiplexing MACs on ethernet ports hinders customer discovery. |
| FR6 | The BMC shall support a means of setting the boot order for the node via Redfish.  The default boot order shall persist across boot attempts. | |
| FR7 | The BMC shall support communication via HTTP and HTTPS.  | |
| FR8 | The BMC shall support both sessions and basic-auth as the primary authentication method. | The customer relies almost exclusively on basic-auth. All expectations for performance are predicated on all communication being via basic-auth and not sessions. |
| FR9 | The node(s) connected to the BMC shall rerun boot options if the boot fails.   It shall not be acceptable for the node to fall into UEFI shell prompt and not retry the other boot options. |  |

## Performance and Quality of Service Requirements

Assumptions:
1. all communication from the customer to the BMC shall be over HTTP/HTTPS using basic-auth
2. the customer shall take responsibility to prevent DoS attacks on the system at large. 

|  ID  |  Requirement | Notes  |
|---|---|---|
| PR1 | The BMC shall support continious sustained Redfish connections (via basic-auth) of 30 requests per minute. | |
| PR2 | The BMC shall support a peak load of 100 requests in a single minute (via basic-auth) for a duration of 1 minute, before returning to the continuous sutained load previously defined. | |
| PR3 | The BMC shall support simultaneous connections via Redfish. | |
| PR4 | The response latency to access the redfish service root: `GET http://{BMC_name}/redfish/v1` shall be no more than XXXs | |
| PR5 | The BMC shall not categorize repeated connections from the same IP as a DoS attempt. | The customer depends heavily on Redfish connectivity, it is not correct for the BMC to treat this behavior as a DoS attempt. |
| PR6 | The redfish interface of the BMC shall have an availability of at least 99.9%. | |
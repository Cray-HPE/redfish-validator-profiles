# BMC & Node Functional and Quality Requirements


### Version 

Current version of file: `v1.0.1`

| Date  |  Author | Content  |
|---|---|---|
|  2022-02-08 | Andrew Nieuwsma  | Initial Draft  |
| 2022-02-10 | Andrew Nieuwsma | Published version 1.0.1 |


## Overview

This document describes the functional and quality requirements that CSM expects any Redfish based BMC (baseboard management controller) controlling nodes to support.  This document is intended to give objective measurable requirements that correspond with the [profiles](../profiles/) listed.

Some of these requirements transcend BMC requirements and relate to the interconnection between BMC and nodes. 

## Definitions:

1. Redfish - Protocol developed by [DMTF](https://www.dmtf.org/standards/redfish)
2. BMC - Baseboard Management Controller, hereafter referred to as the 'BMC', this entity provides the Redfish interface and controls the node.
3. Node - the hardware component that is controlled by the BMC, hereafter referred to as the 'node'.
4. Customer - The Cray System Management (CSM) services is the consumer of the Redfish API hosted on BMCs.  The customer refers to CSM and its usage pertaining to the Redfish APIs on the BMC.


## Functional Requirements

| Date |  ID  |  Requirement | Notes  |
|---|---|---|---|
| 2022-02-08 |  FR1 | The BMC shall support the paths, parameters, and capabilities specified in the [profiles](../profiles/) listing.   Here is a restatement of some of the core support requirements: The BMC shall at a minimum support: <ol><li>Power actions of the node and the BMC</li><li>Firmware inventory/update of the node and the BMC</li><li>Redfish Alerts and Events</li><li>streamed telemetry </li><li>power capping of the node.</li></ol> | The Customer has specified a minimum set of Redfish API requirements needed to support its operation. |
| 2022-02-08 | FR2 | The BMC shall send power alerts regardless of the underlying cause of the power event.  This means that a power alert shall be sent if a human actor, IPMI instruction, or Redfish instruction initiates the power event. | It is crucial to the Customer operations that all power events, regardless of origination are sent so the Customer can manage the system. |
| 2022-02-08 | FR3 | When the BMC reports a power status in response to a request to change status (e.g. a power-on is issued), the BMC shall follow through with the request.  This means that if you respond that a power-on event is occurring, that event shall occur. In no case shall the BMC respond that it is completing a power action if it is unable to do so. | We have seen Redfish implementations that exhibit the following incorrect behavior: <ol><li>The power off is issued to all nodes under the BMC.</li><li> Once all nodes have completed the power off (determined via a poll of power status), a power on is issued to all nodes with no wait time.</li><li>If not enough wait time is given, the nodes will complete the posted power on command but not power on.</li></oi>  |
| 2022-02-08 | FR4 | The BMC shall expose an endpoint to set the NTP server to sync time on the BMC. The BMC shall synchronize this time to the nodes it controls.  |  |
| 2022-02-08 | FR5 | Every ethernet port shall have exactly one mac-address.  In no case shall the BMC and node share the same ethernet port. Each BMC shall have a dedicated ethernet port. | Customer discovery process depends upon MAC address based discovery. Multiplexing MACs on ethernet ports hinders customer discovery. |
| 2022-02-08 | FR6 | The BMC shall support a means of setting the boot order for the node via Redfish.  The default boot order shall persist across boot attempts. | |
| 2022-02-08 | FR7 | The BMC shall support communication via HTTP and HTTPS.  | |
| 2022-02-08 | FR8 | The BMC shall support both sessions and basic-auth as the primary authentication method. | The customer relies almost exclusively on basic-auth. All expectations for performance are predicated on all communication being via basic-auth and not sessions. |
| 2022-02-08 | FR9 | The node(s) connected to the BMC shall rerun boot options if the boot fails.   It shall not be acceptable for the node to fall into UEFI shell prompt and not retry the other boot options. |  |
| 2022-02-10 | FR10 | The BMC shall support x509 certificate (server, TLS) enrollment via a customer-supplied certificate authority, and shall not require a Certificate Signing Request to do so. | Context: we have experienced BMCs that require a CSR in order to support an x509 certificate.  In those cases the only way to get a TLS cert onto the BMC is to tell it, via Redfish, to issue a CSR, pointing it to an authority.   The BMC does this on its own, and then returns the certificate to the caller, all in Redfish calls.   Then we can install THAT cert onto the BMC. If we try to generate our own and install it, it won't work -- the BMC will say the cert is invalid.  THEREFORE: supporting x509 certificate without requiring a CSR is a requirement.| 

## Performance and Quality of Service Requirements

Assumptions:
1. all communication from the customer to the BMC shall be over HTTP/HTTPS using basic-auth
2. the customer shall take responsibility to prevent DoS attacks on the system at large. 

|  ID  |  Requirement | Notes  |
|---|---|---|
| PR1 | The BMC shall service continuous sustained Redfish connections (via basic-auth) of at minimum 30 requests per minute. If this rate is exceeded, except by the peak load clause, the server shall be able to delay response times. The server shall not refuse connections. | |
| PR2 | The BMC shall service a peak load of burst connections as defined as: 100 requests in a single minute (via basic-auth) for a duration of 1 minute, before returning to the continuous sustained load previously defined. If this rate is exceeded the server shall be able to delay response times. The server shall not refuse connections.| |
| PR3 | The BMC shall support atleast 10 simultaneous connections via Redfish. | |
| PR4 | The `response time` - total time for server to respond to client after connection, to any endpoint in the Redfish tree on the BMC shall take no longer than `30 seconds`.| |
| PR5 | The BMC shall not categorize repeated connections from the same IP as a DoS attempt. | The customer depends heavily on Redfish connectivity, it is not correct for the BMC to treat this behavior as a DoS attempt. |
| PR6 | The Redfish interface of the BMC shall have an availability of at least 99.9%. | |
| PR7 | The freshness of streamed telemetry shall be no less than `0.1 hertz` for any datum. | The customer depends on fresh environmental telemetry to make operational decisions.  Having a minimum resolution of `0.1 hertz` is crucial to providing timely insights. |

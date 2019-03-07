# DIDL: Device Interface Description Language
  
*Version: 1.0.1*


*Go straight to the [DIDL Generator](https://didl.me/generator.html) or [DIDL Validator](https://didl.me/validator.html)*

## Table of Contents

   
   * [Abstract](#abstract)
   * [Introduction](#introduction)
   * [Specification](#specification)
       * [Overview](#overview)
       * [Scope and Application](#scope-and-application)
       * [Maximum Length](#maximum-length)
       * [DIDL Objects](#didl-objects)
       * [DIDL Set Schema and Definition](#didl-set-schema-and-definition)
       * [Examples](#examples)
            
## Abstract

Network device interface descriptions are free-form text fields used for the purpose of providing additional information about an interface, such as interface purpose, type of connection, upstream device(s), and/or provider/circuit identifiers.

This document proposes a standardized description language specification for interfaces which interconnect network devices with other network devices, referred to in this document as the Device Interface Description Language (DIDL).

## Introduction

Network device interface descriptions can provide network operators with critical information about an interface very quickly in the CLI, depending on the scope and detail of the information provided. In addition, along with being valuable to human operators manually interacting with a device, network interface descriptions can be parsed and used by automation and orchestration systems. The key for effective use of the interface description field in both of these potential use cases is consistency and standardization. In the case of the human operator, being able to quickly read and understand the description can be a tremendous time saver during outages, and can also help prevent outages caused by human error where an interface, or its up/down-stream neighbors, are mis-identified. In the case of automation/orchestration systems, a standard format is critical to ensuring that actions are applied to the intended interfaces.

This document defines standard, codified interface description field conventions (DIDL) for network devices. In this standards proposal, interface descriptions describe the location of the interface (External or Internal), the function of the interface (e.g. transit, peering, backbone, internal interconnect), the circuit provider with circuit ID (if applicable), intermediate patch panel port (if applicable), neighbor device and interface (if applicable), and the administrative state of the interface (reserved/active/inactive/decommission).

By ensuring that every link has a codified device interface description based on a standard format, clear documentation of the network, as well as automation/orchestration systems, can be built directly from the specification without having to survey every interface on every device. In addition, an attempt has been made to make the specification machine-friendly without losing all human readability.


## Specification

### Overview

***DIDL*** is a codified language for naming network device interfaces which is comprised of an ordered sequence of codified DIDL Objects (DIDLOs), separated by a standard delimiter. One full sequence of DIDLOs (a complete network interface description) is a ***DIDL Set.***

### Scope and Application

DIDL is intended to be used for interfaces which interconnect network devices with other network devices; individual hosts have intentionally been left out of this specification.

### Maximum Length

The total length of any DIDL Set should never exceed 255 characters, which is the maximum length allowed by the SNMP ifAlias OID. Care should also be taken to ensure that the maximum length allowed by the target platform is not exceeded.

### DIDL Objects

There are three types of DIDLOs: ***Uniform Values***, ***Formatted Values***, and ***Freeform Values***.

Uniform Values are constrained to a specific set of values defined in this specification, Formatted Values are dynamic values constrained to a set format, and Freeform values are unconstrained.

#### Case Sensitivity

Uniform Values should always be upper-case; clients which consume network device descriptions should always process Uniform Value DIDLOs as case-sensitive. Freeform Values (unconstrained) are assumed to be variable case; it should always be assumed that Formatted and Freeform Values are not case sensitive.

#### Unknown or Not-Applicable Values

Unknown or N/A values should be left blank, represented as two sequential delimiters (empty DIDLO).

#### Replacing/Encoding the Reserved Separator Character
The ":" (colon) character must always be replaced or encoded, as it is reserved for use as a separator between DIDLOs in a DIDL Set.

#### DIDL Object Definitions

The DIDLOs are defined as follows:

##### Location

*Uniform Value*

Valid values are ***EXT*** (external) and ***INT*** (internal).

-   ***EXT*** - for interfaces which are connected to devices belonging to another organization; typically these are transit, peering, or private network interconnect links

-   ***INT*** - for interfaces which are connected to devices within the same organization

##### Function

*Uniform Value*

Valid values are ***ITR*** (Internet transit), ***IXP*** (Internet exchange point), ***PNI*** (private network interconnect), ***PBB*** (production backbone), and ***IIC*** (internal interconnect):

-   ***ITR*** - for interfaces which are directly connected to provider edge equipment for Internet transit service

-   ***IXP*** - for interfaces which are directly connected to an Internet Exchange Point (provider-neutral peering exchange)

-   ***PNI*** - for interfaces which are directly connected to partner or provider devices 

-   ***PBB*** - for interfaces which are connected to devices in other data centers via dedicated backbone circuits

-   ***IIC*** - for interfaces which are connected to other network devices in the same organization via patch cables or simple cross connects


##### Upstream Service Provider

*Freeform Value*

If applicable, specify the upstream service provider (the end service an interface is providing connectivity to). For example, "Equinix Ashburn IX" would be a useful Upstream Service Provider DIDLO for an interface which provides connectivity to Equinix's public peering exchange in Ashburn. This should be left empty if the interface is not providing connectivity to an upstream service.

##### Circuit Provider

*Freeform Value*

If applicable, specify the name of the circuit provider. This should be left empty for any interface which is not connected to a carrier-provided transport service.

##### Circuit ID

*Freeform Value*

If applicable, specify the circuit ID (prepended with **CID#**) in the provider's format. This should be left empty for any interface which is not connected to a carrier-provided transport service, including logical (LAG) interfaces who's individual members may have circuit IDs.

*Example: CID#BCZN7813*

##### Patch Panel Info

*Formatted Value*

If applicable, specify the patch panel location/name and port which the interface is connected to, using the format [rack]-U[RU pos 1]/[RU pos 2]-P[port 1]/[port 2]...

This value should be empty for device-to-device or device-to-host direct connections.

*Examples: E12-U44/45-P3/4 (LC ports 3/4 in 2U panel in rack E12), G08-U45-P12 (MPO port 12 in 1U panel in rack G08)*

##### Neighboring Device

*Freeform Value*

Specify the hostname (or IP address if the hostname is not known) of the neighbor connected to an interface. This should be  empty if the device information is not available, e.g. when an interface is being reserved for future use but details of its upstream connection are not known.

*Example: ASH-DC03-SP-01*

##### Neighboring Device Port

*Formatted Value*

Specify the device interface name on the neighbor device that the local interface is connected to. This should be left empty if the port information is not available.

*Example: et-0/0/12*

##### Link Aggregation Group

*Formatted Value.*

Specify the name of the LAG/AE interface which this interface is a member of. This should
be left empty if the interface is not a member of a LAG group.

*Example: ae143*

##### State

*Uniform Value*

Valid values are RESERVED, INACTIVE, ACTIVE, and DECOMMISSION:

-   **RESERVED** - interface has been flagged for future use but has not been configured

-   **INACTIVE** - interface has been configured and is ready for use but is not currently active

-   **ACTIVE** - interface is active and in production

-   **DECOMMISSION** - interface is not active and its config is slated for removal.

### DIDL Set Schema and Definition

A DIDL Set is a group of ordered DIDLOs, joined with a standard separator.

#### Separator Character

A single colon (**:**) was chosen for the separator as it is not typically used within  values which make up individual DIDLOs. When programmatically generating DIDL Sets, special care should be taken to sanitize Freeform and Formatted DIDLOs to escape or encode the colon character with another value (e.g. a *pipe*).

#### Set Schema

A DIDL Set is comprised of the following ordered list of DIDLOs:

-   Position 00 - Location

-   Position 01 - Function

-   Position 02 - Upstream Service Provider

-   Position 03 - Circuit Provider

-   Position 04 - Circuit ID

-   Position 05 - Patch Panel Info

-   Position 06 - Neighbor Device

-   Position 07 - Neighbor Device Port

-   Position 08 - LAG Group

-   Position 09 - State

When joined using the standard **:** separator, the combined DIDLOs make up a DIDL Set:

***[Location]:[Function]:[Upstream Service Provider]:[Circuit Provider]:[CID#Circuit ID]:[Patch Panel Info]:[Neighbor Device]:[Neighbor Device Port]:[LAG Group]:[State]***

### Examples

Below are several examples of interface descriptions which were modeled using DIDL.

#### Transit Provider Circuit

This is a set of two physical interfaces in a LAG which connect to an Internet transit provider:

-   **Physical port #1 -** EXT:ITR:Zayo:CID#IPYX/063711/912/ZYO:B16-U45_44-p12:208.185.7.244::ae132:ACTIVE

-   **Physical port #2 -** EXT:ITR:Zayo:CID#IPYX/063711/912/ZYO:B16-U45_44-p14:208.185.7.244::ae132:ACTIVE

-   **Logical LAG interface -** EXT:ITR:Zayo:::208.185.7.244::ae132:ACTIVE

#### Internet Exchange Point

This is a set of three physical interfaces in a LAG which connect to a public peering exchange:

-   **Physical port #1 -** EXT:IXP:Equinix Ashburn IX:Zayo:CID#OGYX/144317//ZYO:G08-U45-p11p12:206.126.236.0/22::ae141:ACTIVE

-   **Physical port #2 -** EXT:IXP:Equinix Ashburn IX:Zayo:CID#OGYX/144319//ZYO:G08-U45-p13p14:206.126.236.0/22::ae141:ACTIVE

-   **Physical port #3 -** EXT:IXP:Equinix Ashburn IX:Zayo:CID#OGYX/144320//ZYO:G08-U45-p15p16:206.126.236.0/22::ae141:ACTIVE

-   **Logical LAG interface -** EXT:IXP:Equinix Ashburn IX:Zayo:::206.126.236.0/22::ae141:ACTIVE

*Note that in this example the network address for the upstream neighbor is used, as it's a public exchange and we connect directly with a number of peers in the exchange via BGP*

#### Internal Interconnect (local devices)

This is a set of interfaces which connect one device to another within the same room in a data center:

-   INT:IIC:::::ASH-DC03-SP-01:et-0/0/4::ACTIVE

-   INT:IIC:::::ASH-DC03-SP-01:et-0/0/5::ACTIVE

-   INT:IIC:::::ASH-DC03-SP-01:et-0/0/6::ACTIVE

-   INT:IIC:::::ASH-DC03-SP-01:et-0/0/7::ACTIVE

*Note in this example that the links are not aggregated, but rather are layer-3 (p2p)*

#### Internal Interconnect (inter-cage devices)

This is a set of interfaces which connect one device to another within different rooms in a data center. In this example that the devices are actually in completely different buildings/facilities, but the cross-connects are still consider "simple" cross connects (unlit single-mode fiber).

-   INT:IIC::::C11-U45-p0:TY3-BBR-01:et-0/0/50:ae22:ACTIVE

-   INT:IIC::::C11-U45-p1:TY3-BBR-01:et-1/0/50:ae22:ACTIVE

-   INT:IIC:::::TY3-BBR-01:ae100:ae22:ACTIVE

#### Simple Validation of a DIDL Set

Because the DIDLOs in a DIDL Set are positional/ordered, it's trivial to perform validation on it. This quick example uses a simple split command in Python:

```python
didl_set = "EXT:ITR:Zayo Ashburn IX:Zayo:ZXY/234098/W209:XVAFA-U50-P12/14:ASH-DCR-01:et-0/0/8:ae123:RESERVED"
objects = didl_set.split(":")
format_issues = []
if len(objects) != 10:
    print "Not a DIDL Set (value passed contains {} objects)".format(len(objects))
else:
    if objects[0] not in ['INT','EXT']:
        format_issues.append("Location object is invalid".format(objects[9]))
    if objects[1] not in ['ITR','IXP', 'PNI' 'PBB', 'IIC']:
        format_issues.append("Function object is invalid ({})".format(objects[1]))
    if objects[9] not in ['RESERVED','ACTIVE','INACTIVE','DECOMMMISSION']:
        format_issues.append("State object is invalid ({})".format(objects[9]))
    if len(didl_set) > 255:
        format_issues.append("DIDL Set is too long (contains {} chars, 255 is defined max)".format(len(didl_set)))

    if len(format_issues) == 0:
        print """
        
            DIDL Set passed basic validation tests and contains the following objects:
            Location: {0}
            Function: {1}
            Upstream Service: {2}
            Circuit Provider: {3}
            Circuit ID: {4}
            Patch Panel Info: {5}
            Neighbor Device: {6}
            Neighbor Port: {7}
            LAG Group: {8}
            State: {9}
            
        """.format(objects[0], objects[1], objects[2], objects[3], objects[4], objects[5], objects[6], objects[7], objects[8], objects[9])

    else:
        print "DIDL Set did not pass basic validation tests, the following issues were found:"
        for issue in format_issues:
            print issue
```

Which should return:

DIDL Set passed basic validation tests and contains the following objects:

Location: EXT  
Function: IXP  
Upstream Service: Equinix Ashburn IX  
Circuit Provider: Zayo  
Circuit ID: CID#OGYX/144317//ZYO  
Patch Panel Info: D15-U22p18p19  
Neighbor Device: 206.126.236.0/22  
Neighbor Port:  
LAG Group: ae141  
State: ACTIVE
 

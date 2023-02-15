# NFV on Public Cloud

## Introduction

1. Middleboxes are typically deployed on premise which led to the practice of
deploying NFV applications on on-premise clusters as well
    * Growth of public cloud makes migrating these services appealing due to
    reducing management costs
    * Techniques for offloading NFV workload to a managed cloud as well as other
    developments in the telecom industry that makes offloading NFV applications
    viable

## Benefits of using Managed Cloud Services

1. Offloading NFs to the cloud

| ![offloading](images/nfv4_offloading.png) |
|:--:|
| Offloading Middlebox Processing to the Cloud |

2. Why offload NF processing to the cloud?
    * Leverage economy of scale to cut costs
    * Simplify management
        - No need for training personnel
        - Upgrades are handled by cloud provider
        - Low-level configuration of NFs is replaced by policy configurations
            + Avoid failures due to misconfiguration
    * Elastic scaling
        - Scale in/out works much better on cloud vs on premise
            + Avoid failures due to overload

## Techniques for Offloading NF to Managed Cloud

1. Important questions to answer
    * How is the redirection implemented?
        - Functional equivalence needs to be maintained
        - Latency should not be inflated
    * How to choose cloud provider to offload to?
        - Dependent on cloud provider's geographical resource footprint
2. Bounce redirection
    * Simplest form of redirection
    * Tunnel ingress and egress traffic to the cloud service
    * Benefit: Does not require any modification to the enterprise or the client
    applications
    * Drawback: Extra round trip to the cloud
        - Can be feasible if cloud point-of-presence is located close to the
        enterprise
3. IP Redirection
    * Save extra round-trip by sending client traffic directly to the cloud
    service
    * Cloud service announces IP prefix on behalf of the enterprise
    * Drawback: multiple point-of-presence (PoP)
        - Cannot ensure that same PoP receives both flows a -> b and b -> a
            + There is not a guarantee about which PoP ends up receiving client's
            packets since all PoPs advertise the same IP address range
        - Since traffic is directed using Border Gateway Protocol (BGP), no
        guarantee of selecting PoP that minimizes end-to-end latency
4. DNS-based Redirection
    * Cloud provider runs DNS resolution on behalf of enterprise
    * Enterprise can send reverse traffic through the same cloud PoP as forward
    traffic
        - Gateway looks up destination cloud PoP's IP address in DNS service
    * Drawback: Loss of backwards compatibility
        - Legacy enterprise applications expose IP addresses to external clients
        (not DNS names)
5. Smart Redirection
    * For each client c and enterprise site e,
        - Choose the cloud PoP P(c,e) such that
            + P(c,e) = argmin(latency(P,c) + latency(P,e))
    * Requires the enterprise gateway to maintain multiple tunnels to each
    participating PoP
        - Cloud service computed estimate latencies between PoPs and clients/
        enterprises using IP address information
6. Latency Inflation due to Redirection
    * Original latency = Host 1 -> Host 2
    * Inflated latency = Host 1 -> Cloud PoP -> Host 2
    * More than 30% of host pairs have inflated latency < original latency
        - Triangle-inequality is voilated in inter-domain routing
        - Cloud providers are well connected to tier-1/tier-2 ISPs

| ![latency](images/nfv4_latency.png) |
|:--:|
| Redirection Latency |

## Observed Performance of NF Offloading

1. What about bandwidth savings?
    * Middleboxes like Web Proxy, WAN accelerator are used to limit WAN 
    bandwidth used by enterprise
        - HTTP proxy limits WAN bandwidth usage by caching web pages
    * If we move them to the cloud, WAN bandwidth becomes high for the
    enterprise
    * Safest solution is to not migrate those types of middleboxes
2. What about bandwidth savings?
    * Solution: Use general-purpose traffic compression in cloud-NFV gateway
    * Protocol agnostic compression technique achieves similar bandwidth
    compression as the original middlebox
3. Which cloud provider to select?
    * Amazon-like footprint
        - Few large PoPs
    * Akamai-like footprint
        - Large number of small PoPs
    * Emerging "edge-computing providers"
4. Telecom providers are ideal for edge computing
    * Telecommunication providers like AT&T and Verizon posses a geographical
    footprint much denser than AWS or Akamai
    * Residential Broadband service providers use functions like virtual
    Broadband Network Gateway (vBNG)
        - To provide residential broadband users with services like subscriber
        management, policy/QoS management, DNS, routing
        - Service providers also offer services like Video-on-Demand CDN,
        virtual Set Top Box
    * Such services are deployed close to the subscribers
        - These compute resources are potential candidates for offloading
5. OpenCORD Initiative
    * Telecommunication providers own central offices
        - Contain switching equipment
    * OpenCORD: Central Office Re-architected as a datacenter
    * Setting up central offices with general purpose servers
    * Provides infrastructure services
        - Deploy their own network functions
        - For 3rd parties to deploy NFV functions
    * Allow enterprises to host network functions on virtualized hardware
        - Colocated with telecom providers' network functions
    * This becomes a candidate realization of mobile edge computing
        - Good incentive for infrastructure owners to virtualize their
        infrastructure
6. Remote sites require illusion of homogenous network
    * Organizations like Chick Fil A or Honeywell have geo-distributed sites
        - Each site needs multiple network services
        - Firewalld, IDS, Deep Packet Inspection, HTTP Proxy, WAN optimizer
    * Used to be implemented on custom hardware on-premise
        - Can be offloaded to a managed service
7. Virtualized customer premise equipment (CPE)
    * Serves as a gateway for multiple parts of an Enterprise Network to connect
    to each other
    * Placed in Edge PoP or centralized datacenter
    * An industry solution for migrating NFV to a cloud service

## Mobile Edge Computing

1. NFs in Cellular Networks
    * Another evolution that is happening that is moving NFv to managed
    infrastructure
        - Converting RAW cellular packets to IP ready packets
    * Different from earlier middleboxes (which were meant for IP packets)
2. Building blocks of a cellular network
    * Access network
        - Consists of base stations (evolved NodeB - eNodeB)
        - Acts as interface between end-users (user equipment/UE) and core network
        - MAC scheduling for uplink and downlink traffic
        - Header compression and user-data encryption
        - Inter-cell Radio Resource Management
    * Core network
        - Mobility control -> making cell-tower handoff decisions for each user
        (Mobility Management Entity - MME)
        - Internet access -> IP address assignment and QoS enforcement (Packet
        Data Network Gateway - P-GW)
        - Packet routing to PGW and anchor point during inter-base station
        handover (Serving Gateway - S-GW)

| ![cell](images/nfv4_cell_network.png) |
|:--:|
| Cell Network |

3. Traditional Radio-Access Networks
    * Packet processing in access network perform 2 types of tasks
        - Analog radio funciton processing (RF processing)
            + Digital-to-analog converter/Analog-to-digital converter
            + Filtering and amplification of signal
        - Digital signal pcessing (Baseband processing):
            + L1, L2, and L3 functionality

| ![station](images/nfv4_base_station.png) |
|:--:|
| Evolution of Base Stations |

4. Benefits/Limitations of 3G/4G Design
    * Benefits:
        - Lower power consumption since RF functionality can be placed on poles
        and rooftops -> efficient cooling
        - Multiple BBus can be placed together in a convenient location ->
        cheaper maintenance
        - One BBU can serve multiple RRHs
    * Limitations:
        - Static RRH-to-BBU assignment -> Resource underutilization
        - BBUs are implemented as specialized hardware -> poor scalability and
        failure handling

## Cloud Radio Access Network (RAN)

1. Cloud Radio Access Network
    * Virtualizes the BBUs in a BBU pool
    * Base-band unit now implemented as software running on general purpose
    servers
    * Allows elastic scaling of BBUs based on current workload
    * BBU-RRH assignment is dynamic, leading to higher resource utilization
2. Location of virtual BBU pool
    * Splitting radio function and base band processing poses stringent
    requirements on connecting links
        - Low latency
        - Low jitter
        - High throughput
    * Need compute capacity in physical proximity of deployed base stations
        - Geo-distributed computing infrastructure
        - Virtualization support required for scalable network processing
3. The complete picture
    * Cellular network providers setup geo-distributed MEC capacity
        - MEC: Mobile edge computing
    * C-RAN functions are deployed on MEC servers
    * MEC capacity is made available for enterprises to offload their NFs
    * Unifying of IP-level network functions and the RAN-level NFs
        - All of them are being deployed on the same managed infrastructure

| ![ran](images/nfv4_ran.png) |
|:--:|
| Cloud Radio Access Network |

## Conclusion

1. Discussed how virtual NFs can be moved to manage cloud infrastructures of
service providers

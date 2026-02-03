# Zero Trust Subinterface Configuration

## Project Overview
This project demonstrates a high-security network architecture typically found in Enterprise environments. Unlike standard enterprise networks that utilize Layer 3 switches for inter-VLAN routing, this topology utilizes a **Palo Alto Next-Generation Firewall (NGFW)** as the gateway for all internal subnets (Router-on-a-Stick).

The goal is to move beyond simple connectivity and establish a **Zero Trust** foundation. By forcing all traffic between internal departments to pass through the firewall, the network ensures comprehensive **Deep Packet Inspection (DPI)**, strict logging, and granular policy enforcement between security zones.

## Network Topology & Enclaves

The network is segmented into three distinct security zones, isolated by Layer 2 VLANs on a Cisco switch and routed via Layer 3 Subinterfaces on the Palo Alto Firewall.

| Zone | VLAN ID | Description |
| :--- | :--- | :--- |
| Corporate | 10 | Standard staff network |
| Classified | 20 | Sensitive Network |
| Guest | 30 | Non-employee guest traffic |

![diagram](https://github.com/Princeton45/zero-trust-sub-config/blob/main/Screenshots/diagram.png)

## Traffic Flow Architecture

1.  **Ingress (Layer 2):** Traffic originates from an endpoint (e.g., Corporate PC). The Cisco Switch tags the frame with `VLAN 10` and forwards it via a generic 802.1Q Trunk port to the Firewall.
2.  **Gateway Processing (Layer 3):** The Palo Alto Firewall receives the tagged frame on `ethernet1/1.10`. It acts as the Default Gateway for the subnet.
3.  **Inspection Engine:** Unlike a Layer 3 switch which only checks IP headers, the Palo Alto inspects the payload. It checks:
    *   **App-ID:** Is the traffic actually what it claims to be?
    *   **User-ID:** Who is sending the packet?
    *   **Security Policy:** Is the Corporate Zone allowed to talk to Classified Zone?
4.  **Egress:** When traffic matches an "Allow" policy, the firewall routes the packet to the destination sub-interface (e.g., `ethernet1/1.30`), re-tags it for `VLAN 30`, and forwards it back through the trunk to the switch for final delivery.

## Technology Stack
*   **Virtualization:** GNS3 Network Simulator
*   **Firewall:** Palo Alto VM-Series (PA-VM)
*   **Switching:** Cisco IOSvL2
*   **Endpoints:** Ubuntu Docker Containers

## Project Implementation 

### 1. Switch VLANs

![vlan](https://github.com/Princeton45/zero-trust-sub-config/blob/main/Screenshots/vlan.png)


### 2. Interface & Zone Configuration (Subinterfaces)
This shows the "Router-on-a-Stick" configuration which includes the Palo ALto eth1/1 physical interface divided into logical subinterfaces, each assigned to a specific VLAN ID, IP subnet, and Security Zone.

![interface](https://github.com/Princeton45/zero-trust-sub-config/blob/main/Screenshots/interface.png)


### 3. Security Policy Implementation
*I configured a firewall security policy that permits traffic from the Corporate Zone to reach the Guest Zone, while an implicit deny blocks communication to the Classified Zone.*

![sec-policy](https://github.com/Princeton45/zero-trust-sub-config/blob/main/Screenshots/sec-policy.png)


### 4. Traffic Inspection & Verification
*The logs demonstrate the initial default "Deny" action of blocking unauthorized pings from the Corporate PC (10.10.10.100) to the Classified PC (10.10.20.100) followed by a successful "Allow" action allowing pings from Corporate PC to Guest PC.*

![logs](https://github.com/Princeton45/zero-trust-sub-config/blob/main/Screenshots/logs.png)


